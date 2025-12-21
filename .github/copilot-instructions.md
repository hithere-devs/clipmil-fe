# Clipmil Frontend - AI Coding Instructions

## Project Overview

Clipmil is a Pinterest-to-YouTube automation platform. This repo (`clipmil-fe`)
is the **React frontend** built with Vite + TypeScript + Tailwind CSS v4. The
backend is a separate Express/PostgreSQL service.

## Architecture

```
Frontend (this repo)          Backend (separate)
┌─────────────────────┐       ┌──────────────────────────────────┐
│  Vite + React 19    │──────▶│  Express API (:4000)             │
│  Tailwind CSS v4    │       │  PostgreSQL + Python DL + GCS    │
│  Supabase (storage) │       │  YouTube API + Gemini AI         │
└─────────────────────┘       └──────────────────────────────────┘
```

**Key backend endpoints consumed:** `/queue`, `/videos/*`, `/frames/*`,
`/research/*`, `/auth/*`, `/trigger-download`

## Tech Stack & Conventions

- **React 19** with functional components and hooks only
- **Tailwind CSS v4** - uses `@theme` directive in
  [src/index.css](src/index.css) for custom properties
- **TypeScript** - strict mode enabled
- **Routing**: React Router v7 with nested routes in [src/App.tsx](src/App.tsx)
- **UI Components**: Radix UI primitives wrapped in
  [src/components/ui/](src/components/ui/) using `class-variance-authority`
  (CVA)

## Authentication Pattern

All auth flows through the backend's Google OAuth2:

- Token stored in `localStorage` as `auth_token`
- [src/api.ts](src/api.ts): `fetchWithAuth()` - **always use this** for
  authenticated API calls
- [src/context/AuthContext.tsx](src/context/AuthContext.tsx): `useAuth()` hook
  provides `user`, `loading`, `signOut`
- Protected routes use `<ProtectedRoute>` wrapper in App.tsx

## Styling System

**Brand colors** (defined in `src/index.css` via `@theme`):

- Primary: `emerald` (#005c45) - main actions
- Secondary: `lime` (#ccff00) - highlights
- Accent: `coral` - navigation active states
- Dark mode: `dark-bg`, `dark-card`, `dark-border`

**Typography**:

- Body: `font-sans` (Oxygen)
- Headings: `font-display` (Clash Display) - auto-applied to h1-h6

**Component patterns**:

```tsx
// Use cn() from src/lib/utils.ts for conditional classes
import { cn } from '@/lib/utils';
<div className={cn('base-classes', condition && 'conditional-class')} />

// Button variants via CVA - see src/components/ui/button.tsx
<Button variant="default" size="lg">Primary Action</Button>
<Button variant="outline">Secondary</Button>
```

## Key Page Patterns

| Page                      | Purpose                      | Key State                     |
| ------------------------- | ---------------------------- | ----------------------------- |
| `DashboardHome`           | Stats + recent activity      | Polls `/queue` every 5s       |
| `QueuePage`               | Pending videos               | CRUD on queue items           |
| `HistoryPage`             | Processed videos             | Filter by status              |
| `ViralVideoGeneratorPage` | Multi-step wizard            | 7-step project creation flow  |
| `VideoDetailPage`         | Single video + Deep Research | Frame extraction, AI metadata |

## API Integration

```tsx
// Always use fetchWithAuth for protected endpoints
import { fetchWithAuth } from '../api';

// Pattern: fetch in useEffect with polling
useEffect(() => {
	const fetchData = async () => {
		const data = await fetchWithAuth('/endpoint');
		setState(data);
	};
	fetchData();
	const interval = setInterval(fetchData, 5000);
	return () => clearInterval(interval);
}, []);
```

**Video statuses** (from backend): `QUEUED` → `PROCESSING` → `DOWNLOADED` →
`UPLOADED` | `FAILED`

## Component Guidelines

1. **New UI components**: Add to `src/components/ui/`, follow Radix + CVA
   pattern from existing components
2. **Page components**: Place in `src/pages/`, add route in `App.tsx`
3. **Shared logic**: Extract to `src/context/` for global state or custom hooks
4. **Icons**: Use `lucide-react` exclusively

## Development Commands

```bash
npm run dev      # Vite dev server on :5173
npm run build    # TypeScript compile + Vite build
npm run lint     # ESLint check
```

**Environment variables** (`.env`):

- `VITE_API_URL` - Backend API URL (default: `http://localhost:4000`)
- `VITE_SUPABASE_URL`, `VITE_SUPABASE_KEY` - Supabase client config

## Backend Context (for full-stack work)

The backend processes videos through this pipeline:

1. **Queue** → Python Pinterest downloader (`scripts/pinterest_downloader.py`)
2. **Download** → Gemini AI generates metadata
3. **Upload** → YouTube API with user's OAuth tokens
4. **Storage** → GCS/S3 for assets, PostgreSQL for state

Key backend modules: `processor.ts` (cron job), `db.ts` (CRUD),
`youtubeUploader.ts`, `aiContentGenerator.ts`
