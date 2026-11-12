# Production Auth — Supabase Implementation Guide

Rekomendasi implementasi **invite-only access** untuk Sidecar Setup Guide menggunakan **Supabase Auth** pada tahap production.

---

## Arsitektur Production

```
┌─────────────────────────────────────────────────────┐
│                   Frontend (Next.js)                │
│  ┌──────────┐  ┌──────────┐  ┌───────────────────┐ │
│  │ Login    │  │ Tutorial │  │ Admin Dashboard   │ │
│  │ Page     │  │ Pages    │  │ (Invite/Revoke)   │ │
│  └────┬─────┘  └────┬─────┘  └────────┬──────────┘ │
│       │              │                 │            │
│       └──────────────┼─────────────────┘            │
│                      │                              │
│              Supabase Auth SDK                      │
└──────────────────────┼──────────────────────────────┘
                       │
         ┌─────────────┼─────────────────┐
         │       Supabase Backend        │
         │  ┌────────┐  ┌────────────┐   │
         │  │  Auth   │  │  Database  │   │
         │  │ Service │  │ (Postgres) │   │
         │  └────────┘  └────────────┘   │
         │  ┌────────┐  ┌────────────┐   │
         │  │  RLS    │  │  Storage   │   │
         │  │ Policies│  │  (Assets)  │   │
         │  └────────┘  └────────────┘   │
         └───────────────────────────────┘
```

## 1. Setup Supabase Project

### Buat Project

1. Daftar di [supabase.com](https://supabase.com)
2. Buat project baru → catat `SUPABASE_URL` dan `SUPABASE_ANON_KEY`
3. **Jangan commit** key ke repository — gunakan environment variables

### Install SDK

```bash
npm install @supabase/supabase-js @supabase/auth-helpers-nextjs
```

### Konfigurasi Client

```javascript
// lib/supabase.js
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY
)
```

## 2. Database Schema

### Tabel Allowlist & Progress

```sql
-- Allowlist: siapa yang boleh akses
CREATE TABLE allowlist (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  invited_by UUID REFERENCES auth.users(id),
  invited_at TIMESTAMPTZ DEFAULT now(),
  is_active BOOLEAN DEFAULT true,
  role TEXT DEFAULT 'member' CHECK (role IN ('admin', 'member'))
);

-- Progress: tracking per user
CREATE TABLE user_progress (
  user_id UUID REFERENCES auth.users(id) PRIMARY KEY,
  completed_chapters INTEGER[] DEFAULT '{}',
  last_chapter INTEGER DEFAULT 0,
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Audit log
CREATE TABLE access_log (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  action TEXT NOT NULL, -- 'invite', 'revoke', 'restore', 'login', 'login_denied'
  actor_id UUID REFERENCES auth.users(id),
  target_email TEXT,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Index untuk performa
CREATE INDEX idx_allowlist_email ON allowlist(email);
CREATE INDEX idx_access_log_created ON access_log(created_at DESC);
```

### Row Level Security (RLS)

```sql
-- Aktifkan RLS
ALTER TABLE allowlist ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_progress ENABLE ROW LEVEL SECURITY;
ALTER TABLE access_log ENABLE ROW LEVEL SECURITY;

-- Allowlist: admin bisa CRUD, member bisa baca email sendiri
CREATE POLICY "Admin full access" ON allowlist
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM allowlist
      WHERE email = auth.jwt()->>'email' AND role = 'admin' AND is_active = true
    )
  );

-- Progress: user hanya bisa akses milik sendiri
CREATE POLICY "Own progress" ON user_progress
  FOR ALL USING (auth.uid() = user_id);

-- Audit log: admin bisa baca semua
CREATE POLICY "Admin read logs" ON access_log
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM allowlist
      WHERE email = auth.jwt()->>'email' AND role = 'admin' AND is_active = true
    )
  );
```

## 3. Invite-Only Authentication Flow

### A. Admin Invite User

```javascript
// Kirim magic link invitation
async function inviteUser(email) {
  // 1. Tambahkan ke allowlist
  const { error: dbError } = await supabase
    .from('allowlist')
    .upsert({ email, is_active: true, invited_by: currentUser.id })

  if (dbError) throw dbError

  // 2. Kirim magic link via Supabase Auth
  const { error: authError } = await supabase.auth.admin.inviteUserByEmail(email, {
    redirectTo: `${window.location.origin}/welcome`
  })

  // 3. Log aksi
  await supabase.from('access_log').insert({
    action: 'invite',
    actor_id: currentUser.id,
    target_email: email
  })

  return { success: !authError, error: authError }
}
```

### B. Magic Link Sign-In (Member)

```javascript
// pages/login.js
async function signIn(email) {
  // Cek allowlist dulu sebelum kirim magic link
  const { data: allowed } = await supabase
    .from('allowlist')
    .select('is_active')
    .eq('email', email)
    .single()

  if (!allowed?.is_active) {
    return { error: 'Email tidak memiliki akses. Hubungi admin.' }
  }

  // Kirim magic link
  const { error } = await supabase.auth.signInWithOtp({
    email,
    options: {
      emailRedirectTo: `${window.location.origin}/auth/callback`
    }
  })

  return { success: !error, error }
}
```

### C. Auth Callback Handler

```javascript
// pages/auth/callback.js (atau app/auth/callback/route.js)
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs'
import { NextResponse } from 'next/server'

export async function GET(request) {
  const { searchParams } = new URL(request.url)
  const code = searchParams.get('code')

  if (code) {
    const supabase = createRouteHandlerClient({ cookies })
    await supabase.auth.exchangeCodeForSession(code)
  }

  return NextResponse.redirect(new URL('/chapter/1', request.url))
}
```

## 4. Middleware Protection

### Server-Side Route Guard

```javascript
// middleware.js
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'
import { NextResponse } from 'next/server'

export async function middleware(req) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req, res })

  const { data: { session } } = await supabase.auth.getSession()

  // Halaman publik: login, landing
  const publicPaths = ['/login', '/', '/auth/callback']
  if (publicPaths.some(p => req.nextUrl.pathname.startsWith(p))) {
    return res
  }

  // Tidak ada session → redirect ke login
  if (!session) {
    return NextResponse.redirect(new URL('/login', req.url))
  }

  // Cek allowlist aktif
  const { data: allowed } = await supabase
    .from('allowlist')
    .select('is_active, role')
    .eq('email', session.user.email)
    .single()

  if (!allowed?.is_active) {
    await supabase.auth.signOut()
    return NextResponse.redirect(new URL('/login?error=revoked', req.url))
  }

  // Admin-only routes
  if (req.nextUrl.pathname.startsWith('/admin') && allowed.role !== 'admin') {
    return NextResponse.redirect(new URL('/chapter/1', req.url))
  }

  return res
}

export const config = {
  matcher: ['/chapter/:path*', '/admin/:path*', '/overview']
}
```

## 5. Progress Persistence

### Sync Progress ke Database

```javascript
// hooks/useProgress.js
import { useState, useEffect } from 'react'
import { supabase } from '../lib/supabase'

export function useProgress(userId) {
  const [completed, setCompleted] = useState(new Set())

  // Load dari database
  useEffect(() => {
    async function load() {
      const { data } = await supabase
        .from('user_progress')
        .select('completed_chapters, last_chapter')
        .eq('user_id', userId)
        .single()

      if (data) {
        setCompleted(new Set(data.completed_chapters))
      }
    }
    if (userId) load()
  }, [userId])

  // Toggle chapter completion
  async function toggleChapter(chapter) {
    const next = new Set(completed)
    next.has(chapter) ? next.delete(chapter) : next.add(chapter)
    setCompleted(next)

    await supabase
      .from('user_progress')
      .upsert({
        user_id: userId,
        completed_chapters: [...next],
        last_chapter: chapter,
        updated_at: new Date().toISOString()
      })
  }

  return { completed, toggleChapter, progress: Math.round(completed.size / 6 * 100) }
}
```

## 6. Admin Dashboard

### Manage Access UI

```javascript
// pages/admin/index.js
export default function AdminDashboard() {
  // Fetch semua users dari allowlist
  // Invite new user
  // Revoke/restore access
  // View audit log

  async function revokeAccess(email) {
    await supabase
      .from('allowlist')
      .update({ is_active: false })
      .eq('email', email)

    await supabase.from('access_log').insert({
      action: 'revoke',
      actor_id: currentUser.id,
      target_email: email
    })
  }

  // ... render admin UI (mirip dengan prototype modal saat ini)
}
```

## 7. Protected Assets (Screenshot/Video)

### Supabase Storage dengan Signed URLs

```sql
-- Storage bucket untuk assets guide
INSERT INTO storage.buckets (id, name, public) VALUES ('guide-assets', 'guide-assets', false);

-- Policy: hanya authenticated + allowlisted users bisa akses
CREATE POLICY "Authenticated access" ON storage.objects
  FOR SELECT USING (
    bucket_id = 'guide-assets'
    AND auth.role() = 'authenticated'
    AND EXISTS (
      SELECT 1 FROM allowlist
      WHERE email = auth.jwt()->>'email' AND is_active = true
    )
  );
```

```javascript
// Dapatkan signed URL untuk asset
async function getAssetUrl(path) {
  const { data } = await supabase.storage
    .from('guide-assets')
    .createSignedUrl(path, 3600) // 1 jam

  return data?.signedUrl
}
```

## 8. Deployment Checklist

### Environment Variables

```env
# .env.local (JANGAN commit ke repo)
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGci...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGci...  # Server-side only
```

### Deployment Steps

1. **Setup Supabase** — Buat project, jalankan SQL migrations
2. **Migrate frontend** — Konversi `index.html` ke Next.js pages/app
3. **Setup environment** — Tambahkan env vars di hosting (Vercel/Netlify)
4. **Seed admin** — Insert admin pertama ke allowlist secara manual
5. **Test flow** — Invite → magic link → login → tutorial → progress
6. **Enable RLS** — Pastikan semua tabel dilindungi RLS
7. **Monitoring** — Setup Supabase dashboard alerts

### Hosting Options

| Platform | Keunggulan |
|----------|-----------|
| **Vercel** | Native Next.js support, edge functions, free tier |
| **Netlify** | Serverless functions, deploy previews |
| **Railway** | Full-stack, database included |

## 9. Migrasi dari Prototype

### Yang Dipertahankan

- ✅ Desain Apple-inspired dan CSS
- ✅ 6 chapter content dan steps
- ✅ Sidebar navigation dan progress bar
- ✅ Copy command functionality
- ✅ Troubleshooting dan FAQ
- ✅ Responsive layout
- ✅ Visual placeholders

### Yang Berubah

- 🔄 `localStorage` progress → Supabase database per user
- 🔄 In-memory users array → Supabase allowlist table
- 🔄 Frontend-only modal → Real invite with magic link
- 🔄 Single `index.html` → Next.js multi-page app
- 🔄 GitHub Pages → Vercel/Netlify with server-side auth
- 🔄 Public access → Protected routes with middleware

---

> **Catatan:** Dokumen ini adalah rekomendasi arsitektur. Implementasi aktual memerlukan setup Supabase project, konfigurasi email templates, dan pengujian end-to-end.
