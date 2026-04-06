# Introduction to Next.js

**The Full-Stack React Framework**

App Router | RSC | SSR | SSG | API routes | Vercel

---

## Table of Contents

1. [Agenda](#slide-02--agenda)
2. [What Is Next.js?](#slide-03--what-is-nextjs)
3. [App Router vs Pages Router](#slide-04--app-router-vs-pages-router)
4. [File-Based Routing](#slide-05--file-based-routing)
5. [Server Components vs Client Components](#slide-06--server-components-vs-client-components)
6. [Data Fetching](#slide-07--data-fetching)
7. [Server Actions](#slide-08--server-actions)
8. [Rendering Strategies](#slide-09--rendering-strategies)
9. [Layouts & Templates](#slide-10--layouts--templates)
10. [Middleware](#slide-11--middleware)
11. [API Routes / Route Handlers](#slide-12--api-routes--route-handlers)
12. [Styling](#slide-13--styling)
13. [Image & Font Optimisation](#slide-14--image--font-optimisation)
14. [Metadata & SEO](#slide-15--metadata--seo)
15. [Authentication Patterns](#slide-16--authentication-patterns)
16. [Database & ORM Integration](#slide-17--database--orm-integration)
17. [Testing](#slide-18--testing)
18. [Deployment](#slide-19--deployment)
19. [Summary & Next Steps](#slide-20--summary--next-steps)

---

## Slide 02 — Agenda

### Foundations

- What is Next.js & the App Router
- App Router vs Pages Router
- File-based routing conventions
- Server Components vs Client Components

### Data & Rendering

- Data fetching & caching
- Server Actions & mutations
- SSR, SSG, ISR & streaming
- Layouts, templates & error boundaries

### Features

- Middleware & edge functions
- API Route Handlers
- Styling approaches
- Image & font optimisation

### Production

- Metadata & SEO
- Authentication patterns
- Database & ORM integration
- Testing & deployment

---

## Slide 03 — What Is Next.js?

Next.js is a **full-stack React framework** created by **Vercel** (formerly ZEIT) in 2016. It extends React with server-side rendering, static generation, file-based routing, and a powerful build system — letting teams ship production-grade web applications without stitching together dozens of tools.

### History

- 2016 — First release (Pages Router)
- 2020 — ISR, Image component
- 2022 — App Router (v13 beta)
- 2023 — App Router stable (v13.4)
- 2024 — Partial Prerendering (v14/15)

### Key Features

- React Server Components
- File-based routing
- Server Actions (mutations)
- Automatic code splitting
- Built-in image/font optimisation

### Who Uses It?

- Vercel, Netflix, TikTok
- Hulu, Notion, Twitch
- Washington Post, Target
- OpenAI, Stripe Dashboard
- Most popular React framework

### Next.js is NOT

A replacement for React (it *uses* React) · A backend-only framework (it's full-stack) · Locked to Vercel (self-hosting is fully supported)

---

## Slide 04 — App Router vs Pages Router

Next.js has two routing systems. The **App Router** (introduced in v13) is the recommended default. The **Pages Router** remains fully supported for existing projects.

| Feature | App Router (`app/`) | Pages Router (`pages/`) |
|---------|---------------------|------------------------|
| Server Components | Default (RSC) | Not supported |
| Layouts | Nested, persistent | Manual with `_app.tsx` |
| Data Fetching | `async` components, `fetch` | `getServerSideProps`, `getStaticProps` |
| Streaming | Built-in with Suspense | Limited |
| Server Actions | Native support | Not available |
| Route Handlers | `route.ts` files | `pages/api/*.ts` |
| Middleware | Supported | Supported |

### Directory Structure (App Router)

```
app/
  layout.tsx        # Root layout
  page.tsx          # Home route (/)
  about/
    page.tsx        # /about
  blog/
    [slug]/
      page.tsx      # /blog/:slug
```

### Migration Strategy

- Both routers can coexist in one project
- Migrate route-by-route incrementally
- Move shared layouts first
- Convert data fetching last

---

## Slide 05 — File-Based Routing

In the App Router, the **file system is the router**. Each folder inside `app/` maps to a URL segment. Special files define behaviour at each route level.

### Special Files

| File | Purpose |
|------|---------|
| `page.tsx` | Makes route publicly accessible |
| `layout.tsx` | Shared UI for a segment |
| `loading.tsx` | Suspense fallback UI |
| `error.tsx` | Error boundary for segment |
| `not-found.tsx` | 404 UI for segment |
| `route.ts` | API endpoint (no UI) |
| `template.tsx` | Re-rendered layout variant |

### Dynamic & Advanced Routes

```
# Dynamic segments
app/blog/[slug]/page.tsx     → /blog/hello-world

# Catch-all
app/docs/[...slug]/page.tsx  → /docs/a/b/c

# Optional catch-all
app/shop/[[...slug]]/page.tsx

# Route groups (no URL impact)
app/(marketing)/about/page.tsx
app/(shop)/products/page.tsx

# Parallel routes
app/@modal/login/page.tsx
app/@sidebar/page.tsx
```

### Colocation

Only `page.tsx` and `route.ts` are publicly addressable. You can safely colocate components, tests, styles, and utilities inside route folders without exposing them as routes.

---

## Slide 06 — Server Components vs Client Components

In the App Router, components are **Server Components by default**. Add `"use client"` at the top of a file to make it a Client Component. The boundary between them is the serialisation boundary.

### Server Components (default)

```tsx
// app/users/page.tsx  (Server Component)
import { db } from '@/lib/db';

export default async function UsersPage() {
  const users = await db.user.findMany();

  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}
```

- Direct database/API access
- Zero client-side JS bundle
- Access secrets & env vars safely
- Cannot use hooks or browser APIs

### Client Components

```tsx
'use client';
// components/counter.tsx

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

- Interactivity (onClick, onChange)
- React hooks (useState, useEffect)
- Browser APIs (localStorage, etc.)
- Hydrated on the client

### Rule of Thumb

Keep components as **Server Components** until you need interactivity. Push `"use client"` as far down the tree as possible — wrap only the interactive leaf, not the entire page.

---

## Slide 07 — Data Fetching

In the App Router, data fetching happens inside **async Server Components** using the standard `fetch` API. Next.js extends `fetch` with built-in caching and revalidation.

### Fetch with Caching

```tsx
// Cached by default (static)
const res = await fetch('https://api.example.com/data');

// Revalidate every 60 seconds (ISR)
const res = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 }
});

// No cache (dynamic / SSR)
const res = await fetch('https://api.example.com/data', {
  cache: 'no-store'
});
```

### generateStaticParams

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map(post => ({
    slug: post.slug,
  }));
}

export default async function BlogPost(
  { params }: { params: { slug: string } }
) {
  const post = await getPost(params.slug);
  return <article>{post.content}</article>;
}
```

### Caching Strategies

| Strategy | Option | Behaviour |
|----------|--------|-----------|
| Static (default) | No option needed | Cached at build time, served from CDN |
| ISR | `next: { revalidate: N }` | Stale-while-revalidate after N seconds |
| Dynamic | `cache: 'no-store'` | Fresh data on every request |
| Tag-based | `next: { tags: ['posts'] }` | Invalidated via `revalidateTag('posts')` |

---

## Slide 08 — Server Actions

Server Actions let you define **async functions that run on the server**, callable directly from Client or Server Components. Mark them with `"use server"`. They replace the need for separate API endpoints for mutations.

### Inline Server Action

```tsx
// app/posts/new/page.tsx (Server Component)
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { db } from '@/lib/db';

export default function NewPost() {
  async function createPost(formData: FormData) {
    'use server';
    const title = formData.get('title') as string;
    await db.post.create({ data: { title } });
    revalidatePath('/posts');
    redirect('/posts');
  }

  return (
    <form action={createPost}>
      <input name="title" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

### Separate Actions File

```tsx
// app/actions.ts
'use server';

import { revalidateTag } from 'next/cache';
import { db } from '@/lib/db';

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  revalidateTag('posts');
}

export async function toggleLike(postId: string) {
  await db.like.toggle({ postId });
  revalidateTag('posts');
}
```

- Can be imported by Client Components
- Type-safe with TypeScript
- Progressive enhancement (works without JS)
- Automatic request deduplication

### Key Benefits

**No manual API layer** for mutations · Built-in **CSRF protection** · Works with forms natively (progressive enhancement) · Compose with `revalidatePath` and `revalidateTag` for cache invalidation

---

## Slide 09 — Rendering Strategies

Next.js supports multiple rendering strategies. The framework **automatically chooses** between static and dynamic based on your data fetching, but you can control it explicitly.

### SSG — Static Site Generation

- Pages rendered at **build time**
- HTML cached on CDN
- Fastest TTFB
- Use for marketing pages, docs, blogs

```tsx
// Static by default
export default async function Page() {
  const data = await fetch('...');
  return <div>{data}</div>;
}
```

### SSR — Server-Side Rendering

- Rendered on **every request**
- Always fresh data
- Higher TTFB than SSG
- Use for personalised content

```tsx
export const dynamic = 'force-dynamic';

export default async function Page() {
  const data = await fetch('...', { cache: 'no-store' });
  return <div>{data}</div>;
}
```

### ISR — Incremental Static Regeneration

- Static + **time-based revalidation**
- Stale-while-revalidate pattern
- Best of SSG and SSR
- Use for content that changes periodically

```tsx
export const revalidate = 60;

export default async function Page() {
  const data = await fetch('...');
  return <div>{data}</div>;
}
```

### Streaming with Suspense

```tsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <main>
      <h1>Dashboard</h1>
      <Suspense fallback={<Skeleton />}>
        <SlowDataComponent />
      </Suspense>
    </main>
  );
}
```

### Partial Prerendering (PPR)

- Combines static shell + dynamic holes
- Static HTML served instantly from CDN
- Dynamic parts stream in via Suspense
- Experimental in Next.js 14/15
- Best of all strategies in one route

---

## Slide 10 — Layouts & Templates

Next.js uses a **nested layout system**. Layouts wrap child segments and persist across navigations — they don't re-render or lose state when the user moves between sibling routes.

### Root Layout (required)

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <nav>Site Navigation</nav>
        {children}
      </body>
    </html>
  );
}
```

### Nested Layout

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

### loading.tsx

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <Skeleton />;
}
// Auto-wrapped in Suspense
```

### error.tsx

```tsx
'use client';
// app/dashboard/error.tsx
export default function Error({
  error, reset,
}: {
  error: Error; reset: () => void;
}) {
  return <button onClick={reset}>Retry</button>;
}
```

### not-found.tsx

```tsx
// app/dashboard/not-found.tsx
export default function NotFound() {
  return (
    <div>
      <h2>Not Found</h2>
      <p>Could not find resource</p>
    </div>
  );
}
```

### Layout vs Template

**Layouts** persist state across navigations. **Templates** (`template.tsx`) create a new instance on every navigation — useful for animations, per-page analytics, or resetting form state.

---

## Slide 11 — Middleware

Next.js **Middleware** runs *before* a request is completed. It executes at the edge (close to the user) and can rewrite, redirect, modify headers, or block requests. Define it in `middleware.ts` at the project root.

### Middleware Example

```tsx
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const token = request.cookies.get('session');

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  const response = NextResponse.next();
  response.headers.set('x-pathname', request.nextUrl.pathname);
  return response;
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*'],
};
```

### Common Use Cases

- **Authentication** — redirect if no session
- **A/B testing** — rewrite to variant pages
- **Geolocation** — route by country/region
- **Rate limiting** — block abusive requests
- **Bot detection** — challenge suspicious traffic
- i18n — redirect to locale-specific routes

### Matcher Config

```tsx
export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico).*)',
    '/dashboard/:path*',
    '/api/:path*',
  ],
};
```

---

## Slide 12 — API Routes / Route Handlers

Route Handlers let you create **custom API endpoints** using the Web `Request` and `Response` APIs. Define them in `route.ts` files inside the `app/` directory.

### Basic CRUD

```tsx
// app/api/posts/route.ts
import { NextResponse } from 'next/server';
import { db } from '@/lib/db';

export async function GET() {
  const posts = await db.post.findMany();
  return NextResponse.json(posts);
}

export async function POST(request: Request) {
  const body = await request.json();
  const post = await db.post.create({ data: body });
  return NextResponse.json(post, { status: 201 });
}
```

### Dynamic Route Handler

```tsx
// app/api/posts/[id]/route.ts
import { NextResponse } from 'next/server';

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const post = await db.post.findUnique({
    where: { id: params.id }
  });
  if (!post) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }
  return NextResponse.json(post);
}
```

### Streaming Response

```tsx
// app/api/stream/route.ts
export async function GET() {
  const stream = new ReadableStream({
    async start(controller) {
      for (const chunk of data) {
        controller.enqueue(new TextEncoder().encode(chunk));
        await wait(100);
      }
      controller.close();
    },
  });
  return new Response(stream);
}
```

### Runtime Options

| Runtime | Use Case |
|---------|----------|
| `nodejs` (default) | Full Node.js APIs, DB drivers |
| `edge` | Low latency, limited APIs |

Supported HTTP methods: `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `HEAD`, `OPTIONS`

---

## Slide 13 — Styling

Next.js supports multiple styling approaches out of the box. The choice depends on your team's preferences and whether components are Server or Client components.

### CSS Modules (recommended)

```css
/* app/components/Button.module.css */
.primary {
  background: #0070f3;
  color: white;
  border-radius: 6px;
  padding: 0.5rem 1rem;
}
```

```tsx
import styles from './Button.module.css';

export function Button({ children }) {
  return (
    <button className={styles.primary}>
      {children}
    </button>
  );
}
```

### Tailwind CSS

```tsx
export function Card({ title, description }) {
  return (
    <div className="rounded-lg border bg-white p-6 shadow-sm">
      <h3 className="text-lg font-semibold text-gray-900">{title}</h3>
      <p className="mt-2 text-gray-600">{description}</p>
    </div>
  );
}
```

Tailwind is the most popular choice in the Next.js ecosystem and is included by default in `create-next-app`.

### Global Styles

```tsx
// app/layout.tsx
import './globals.css';

export default function RootLayout({ children }) {
  return <html><body>{children}</body></html>;
}
```

### CSS-in-JS Limitations

- Libraries like styled-components, Emotion require `"use client"`
- Cannot be used in Server Components
- Runtime CSS-in-JS adds JS bundle weight
- Consider CSS Modules or Tailwind for RSC-first apps

---

## Slide 14 — Image & Font Optimisation

Next.js provides built-in components for **automatic image optimisation** and **zero-layout-shift font loading** that significantly improve Core Web Vitals.

### next/image

```tsx
import Image from 'next/image';

export function Hero() {
  return (
    <Image
      src="/hero.jpg"
      alt="Hero image"
      width={1200}
      height={600}
      priority
      placeholder="blur"
    />
  );
}
```

- Automatic WebP/AVIF conversion
- Responsive `srcSet` generation
- Lazy loading by default
- Prevents Cumulative Layout Shift
- On-demand resizing at the edge

### next/font

```tsx
// app/layout.tsx
import { Inter, Playfair_Display } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  variable: '--font-inter',
});

const playfair = Playfair_Display({
  subsets: ['latin'],
  variable: '--font-playfair',
});

export default function RootLayout({ children }) {
  return (
    <html className={`${inter.variable} ${playfair.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

- Self-hosted (no external requests)
- Zero layout shift via `size-adjust`
- Automatic subsetting
- Works with Google Fonts & local files
- CSS variable support for Tailwind

---

## Slide 15 — Metadata & SEO

Next.js provides a powerful **Metadata API** for defining SEO tags, Open Graph images, and structured data. Metadata can be static or dynamically generated per page.

### Static Metadata

```tsx
// app/layout.tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'My App',
    template: '%s | My App',
  },
  description: 'Built with Next.js',
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://myapp.com',
    siteName: 'My App',
  },
};
```

### Dynamic Metadata

```tsx
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next';

export async function generateMetadata(
  { params }: { params: { slug: string } }
): Promise<Metadata> {
  const post = await getPost(params.slug);
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.coverImage],
    },
  };
}
```

### JSON-LD

```tsx
export default function Page() {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: 'My Post',
    author: { '@type': 'Person', name: 'Author' },
  };
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
    />
  );
}
```

### Sitemap

```tsx
// app/sitemap.ts
export default async function sitemap() {
  const posts = await getPosts();
  return [
    { url: 'https://myapp.com', lastModified: new Date() },
    ...posts.map(post => ({
      url: `https://myapp.com/blog/${post.slug}`,
      lastModified: post.updatedAt,
    })),
  ];
}
```

### robots.ts

```tsx
// app/robots.ts
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/admin/',
    },
    sitemap: 'https://myapp.com/sitemap.xml',
  };
}
```

---

## Slide 16 — Authentication Patterns

Authentication in Next.js typically combines **Middleware** for route protection, **Server Components** for session checks, and a library like **Auth.js** (NextAuth.js v5) for the auth logic.

### Auth.js Setup

```tsx
// auth.ts
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';
import { PrismaAdapter } from '@auth/prisma-adapter';
import { prisma } from '@/lib/prisma';

export const { handlers, signIn, signOut, auth } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [GitHub],
});

// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/auth';
export const { GET, POST } = handlers;
```

### Middleware Protection

```tsx
// middleware.ts
import { auth } from './auth';

export default auth((req) => {
  if (!req.auth && req.nextUrl.pathname !== '/login') {
    const loginUrl = new URL('/login', req.nextUrl.origin);
    return Response.redirect(loginUrl);
  }
});

export const config = {
  matcher: ['/dashboard/:path*'],
};
```

### Server Component Session

```tsx
// app/dashboard/page.tsx
import { auth } from '@/auth';
import { redirect } from 'next/navigation';

export default async function Dashboard() {
  const session = await auth();
  if (!session) redirect('/login');

  return <h1>Welcome, {session.user.name}</h1>;
}
```

### Auth Strategies

| Strategy | Best For |
|----------|----------|
| JWT sessions | Stateless, edge-compatible |
| Database sessions | Revocable, secure |
| OAuth providers | Social login (GitHub, Google) |
| Credentials | Email/password (use with care) |

---

## Slide 17 — Database & ORM Integration

Next.js Server Components and Server Actions provide **direct database access** without an intermediate API layer. The two most popular ORMs in the ecosystem are **Prisma** and **Drizzle**.

### Prisma

```prisma
// prisma/schema.prisma
model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
}
```

```tsx
// lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma || new PrismaClient();

if (process.env.NODE_ENV !== 'production')
  globalForPrisma.prisma = prisma;
```

### Drizzle

```tsx
// db/schema.ts
import { pgTable, text, boolean, timestamp } from 'drizzle-orm/pg-core';

export const posts = pgTable('posts', {
  id: text('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  published: boolean('published').default(false),
  createdAt: timestamp('created_at').defaultNow(),
});
```

### server-only Pattern

```tsx
// lib/db.ts — prevent client import
import 'server-only';
import { prisma } from './prisma';

export async function getPosts() {
  return prisma.post.findMany({
    where: { published: true },
    orderBy: { createdAt: 'desc' },
  });
}
```

### Connection Pooling

- Serverless functions create many connections
- Use **PgBouncer** or **Prisma Accelerate**
- Neon, Supabase, PlanetScale offer built-in pooling
- Cache the client on `globalThis` in dev
- Set `connection_limit=1` per serverless instance

---

## Slide 18 — Testing

A solid Next.js testing strategy covers **unit tests** (components/utilities), **integration tests** (API routes, server actions), and **end-to-end tests** (full user flows).

### Vitest + React Testing Library

```tsx
// __tests__/components/button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from '@/components/button';

describe('Button', () => {
  it('handles click events', async () => {
    const onClick = vi.fn();
    render(<Button onClick={onClick}>Click me</Button>);

    await userEvent.click(screen.getByRole('button'));
    expect(onClick).toHaveBeenCalledOnce();
  });
});
```

### Playwright (E2E)

```tsx
// e2e/blog.spec.ts
import { test, expect } from '@playwright/test';

test('can create a blog post', async ({ page }) => {
  await page.goto('/posts/new');
  await page.fill('[name="title"]', 'Test Post');
  await page.fill('[name="content"]', 'Hello world');
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL(/\/posts\//);
  await expect(page.getByText('Test Post')).toBeVisible();
});
```

### MSW (Mock Service Worker)

```tsx
// mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/posts', () => {
    return HttpResponse.json([
      { id: '1', title: 'Mock Post' },
    ]);
  }),
];
```

### Testing Strategy

| Layer | Tool | What to Test |
|-------|------|--------------|
| Unit | Vitest | Utils, hooks, components |
| Integration | Vitest + RTL | Pages, API routes |
| E2E | Playwright | Full user flows |
| API mocking | MSW | External services |

---

## Slide 19 — Deployment

Next.js can be deployed to **Vercel** (zero-config), **self-hosted** with Node.js, or **containerised** with Docker. Each option has different trade-offs for features like ISR, Middleware, and Edge Runtime.

### Vercel (recommended)

- Zero configuration deployment
- Automatic edge network
- Built-in ISR & image optimisation
- Preview deployments per PR
- Analytics & speed insights
- Serverless & edge functions

```bash
npx vercel
```

### Self-Hosted (Node.js)

- Full control over infrastructure
- All features supported
- Requires own CDN for caching
- Manual scaling & monitoring

```bash
npm run build
npm start
```

### Docker

```dockerfile
FROM node:20-alpine AS base

FROM base AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]
```

### Environment Variables

| Prefix | Available |
|--------|-----------|
| `NEXT_PUBLIC_*` | Client + Server (inlined at build) |
| No prefix | Server only (secrets safe) |

### Edge vs Node Runtime

| Aspect | Node.js | Edge |
|--------|---------|------|
| Cold start | ~250ms | ~0ms |
| APIs | Full Node.js | Web APIs only |
| Max duration | Minutes | Seconds |
| Best for | Heavy compute, DB | Auth, redirects |

---

## Slide 20 — Summary & Next Steps

### Key Takeaways

- **App Router** is the modern default — use it for new projects
- **Server Components** reduce bundle size and enable direct data access
- **Server Actions** eliminate boilerplate API routes for mutations
- Multiple **rendering strategies** (SSG, SSR, ISR, streaming) per route
- Built-in **image**, **font**, and **metadata** optimisation
- Middleware runs at the edge for auth, redirects, and A/B tests
- Deploy anywhere — Vercel, Docker, or bare Node.js

### Essential Resources

- [nextjs.org/docs](https://nextjs.org/docs) — Official documentation
- [nextjs.org/learn](https://nextjs.org/learn) — Interactive tutorial
- [github.com/vercel/next.js](https://github.com/vercel/next.js) — Source & examples
- [vercel.com/templates](https://vercel.com/templates) — Starter templates

### Recommended Next Steps

- Run `npx create-next-app@latest` and explore
- Build a blog with App Router & Server Actions
- Add Auth.js authentication
- Deploy to Vercel with a database (Neon/Supabase)
- Write E2E tests with Playwright
- Experiment with Partial Prerendering

---

*Thank you! — Built with Reveal.js · Single self-contained HTML file*
