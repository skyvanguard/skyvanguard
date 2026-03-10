# SkyVanguard Web - Design Document

**Date:** 2026-03-10
**Status:** Approved
**Type:** Personal portfolio + blog + research platform

## Overview

Personal website for skyvanguard deployed on Vercel. Showcases projects, publishes technical blog posts (CTF writeups, pentesting, AI/ML tutorials), and hosts research papers with interactive visualizations.

## Decisions

- **Stack:** Next.js 15 (App Router) + Tailwind CSS 4 + MDX + Framer Motion + React Three Fiber
- **Repo:** New separate repository (`skyvanguard-web`)
- **Language:** Bilingual EN/ES with next-intl
- **Style:** Creative/experimental - dark theme, 3D hero, scroll animations, terminal aesthetic
- **Content:** MDX files for blog posts, research papers, and project descriptions
- **Deploy:** Vercel with auto-deploy from GitHub

## Architecture

```
skyvanguard-web/
├── src/
│   ├── app/
│   │   ├── [locale]/           # i18n routes (en, es)
│   │   │   ├── page.tsx        # Landing/Hero
│   │   │   ├── projects/       # Project showcase
│   │   │   ├── blog/           # Technical blog
│   │   │   ├── research/       # Research papers
│   │   │   └── about/          # About me
│   │   └── layout.tsx
│   ├── components/
│   │   ├── 3d/                 # Three.js components (hero scene, visualizations)
│   │   ├── ui/                 # Reusable UI components
│   │   ├── blog/               # Blog-specific components
│   │   └── layout/             # Nav, Footer, LanguageSwitcher
│   ├── content/
│   │   ├── blog/               # MDX blog posts (en/es)
│   │   ├── research/           # MDX research papers (en/es)
│   │   └── projects/           # Project metadata (JSON/MDX)
│   ├── lib/
│   │   ├── mdx.ts              # MDX utilities
│   │   ├── i18n.ts             # i18n config
│   │   └── content.ts          # Content queries
│   └── messages/
│       ├── en.json             # English UI translations
│       └── es.json             # Spanish UI translations
├── public/
│   ├── models/                 # 3D models (GLTF)
│   └── images/
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

## Dependencies

- `next` 15 (App Router, SSG)
- `tailwindcss` 4 + `@tailwindcss/typography`
- `next-intl` (i18n)
- `next-mdx-remote` (MDX rendering)
- `framer-motion` (animations)
- `@react-three/fiber` + `@react-three/drei` (3D)
- `shiki` (syntax highlighting)

## Pages

### Landing Page (`/`)
- **Hero:** Interactive 3D particle network scene reacting to mouse, animated terminal-style title, CTA
- **Highlights:** Animated cards for main areas (Cybersecurity, AI, Research) with on-scroll animations
- **Featured Projects:** 3-4 featured projects grid (Kryon, mcp-vanguard, arandu, zeta-life) with hover effects
- **Latest Posts:** 3 recent blog posts
- **Contact:** Links to GitHub, email, TryHackMe

### Projects (`/projects`)
- Filterable grid by category (Security, AI, Infrastructure, Research)
- Each project: image/demo, description, tech stack badges, GitHub link
- Expandable detail or individual project page

### Blog (`/blog`)
- Post list with tags, date, reading time
- Categories: CTF Writeups, Pentesting, AI/ML, Tutorials
- MDX posts with syntax highlighting (shiki), interactive components, diagrams
- RSS feed

### Research (`/research`)
- Formal papers with abstract, sections, embedded visualizations
- Interactive charts/graphs inline (e.g., zeta-life visualizations)
- Citation format, optional PDF export

### About (`/about`)
- Personal story (the question at age 8, self-taught, Paraguay)
- Experience/milestone timeline
- Visual tech stack
- TryHackMe stats

## Visual Design

### Color Palette
- Background: `#0a0a0a` (deep black) with subtle gradients
- Primary: `#00FF41` (signature hacker green)
- Accents: `#00B4D8` (cyan), `#6E40C9` (violet) for categories
- Text: `#e5e5e5` (light) / `#a3a3a3` (secondary)

### Typography
- Headings: `JetBrains Mono` or `Fira Code` (monospace)
- Body: `Inter` (readability)
- Code blocks: `Fira Code` with ligatures

### Creative Elements
- 3D interactive particle network hero (React Three Fiber)
- Framer Motion page transitions
- Scroll-triggered fade/slide animations
- Custom cursor with glow/trail effect
- Terminal-style sections (prompt, typing effect)
- Glassmorphism cards with backdrop blur
- Responsive: mobile-first, 3D simplified/disabled on mobile

## Internationalization

- next-intl with middleware for locale routing
- URLs: `domain/en/blog/...` and `domain/es/blog/...`
- Default locale: English
- Visible language toggle in navigation
- UI strings in `messages/en.json` and `messages/es.json`
- Content: separate MDX files `post-name.en.mdx` / `post-name.es.mdx`

## Content Format (MDX)

```mdx
---
title: "Post Title"
date: "2026-03-10"
tags: ["pentesting", "xxe"]
locale: "en"
category: "blog"
readingTime: 8
---

Content with embedded React components, code blocks, and interactive demos.
```

## Deployment

- Vercel with auto-deploy from GitHub
- Domain: `skyvanguard.vercel.app` (free) or custom domain
- Vercel Analytics integrated
- next/image for automatic image optimization
