# ▲ Introduction to Next.js

An interactive Reveal.js presentation covering Next.js — from the App Router and React Server Components through data fetching, server actions, rendering strategies, middleware, and deployment.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_Next_js/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | Next.js overview |
| 02 | Agenda | Topics at a glance |
| 03 | What Is Next.js? | Origin, Vercel, key features, who uses it |
| 04 | App Router vs Pages Router | Directory structure, migration, when to use which |
| 05 | File-Based Routing | Folder conventions, dynamic routes, route groups, parallel routes |
| 06 | Server Components vs Client Components | "use client", serialisation boundary, when to use each |
| 07 | Data Fetching | Async server components, fetch caching, revalidation, generateStaticParams |
| 08 | Server Actions | "use server", form actions, mutations, revalidatePath/Tag |
| 09 | Rendering Strategies | SSR, SSG, ISR, streaming, partial prerendering |
| 10 | Layouts & Templates | Root layout, nested layouts, loading.tsx, error.tsx, not-found.tsx |
| 11 | Middleware | Matcher, redirects, rewrites, authentication, geolocation |
| 12 | API Routes / Route Handlers | GET, POST, streaming responses, edge runtime |
| 13 | Styling | CSS Modules, Tailwind, CSS-in-JS limitations in RSC, global styles |
| 14 | Image & Font Optimisation | next/image, next/font, automatic optimisation, LCP |
| 15 | Metadata & SEO | generateMetadata, Open Graph, JSON-LD, sitemap, robots |
| 16 | Authentication Patterns | NextAuth.js/Auth.js, middleware, session management |
| 17 | Database & ORM Integration | Prisma, Drizzle, server-only, connection pooling |
| 18 | Testing | Playwright, Vitest, React Testing Library, MSW |
| 19 | Deployment | Vercel, self-hosted, Docker, edge vs Node runtime, environment variables |
| 20 | Summary & Next Steps | Key takeaways and recommended reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

Vercel, *Next.js Documentation* — nextjs.org/docs · Vercel, *Next.js Learn Course* — nextjs.org/learn · React Team, *React Server Components RFC* — github.com/reactjs/rfcs · Auth.js, *Authentication for Next.js* — authjs.dev · Prisma, *Next.js Integration Guide* — prisma.io/nextjs

## License

Educational use. Code examples provided as-is.
