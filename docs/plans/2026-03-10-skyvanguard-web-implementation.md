# SkyVanguard Web Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a creative personal portfolio + blog + research platform deployed on Vercel with bilingual support (EN/ES).

**Architecture:** Next.js 15 App Router with `[locale]` dynamic segment for i18n via next-intl. Content authored in MDX files with frontmatter, rendered via next-mdx-remote/rsc. 3D hero with React Three Fiber, animations with Framer Motion. Static generation for all content pages.

**Tech Stack:** Next.js 15, TypeScript, Tailwind CSS 4, next-intl, next-mdx-remote, Framer Motion, @react-three/fiber, @react-three/drei, shiki

**Design Doc:** `docs/plans/2026-03-10-skyvanguard-web-design.md`

---

## Task 1: Create repo and scaffold Next.js project

**Files:**
- Create: `skyvanguard-web/` (new repo at `C:/Users/skyva/Documents/skyvanguard-web/`)

**Step 1: Create GitHub repo**

```bash
cd /c/Users/skyva/Documents
gh repo create skyvanguard/skyvanguard-web --public --clone --description "Personal portfolio, blog & research platform"
```

**Step 2: Scaffold Next.js project**

```bash
cd /c/Users/skyva/Documents/skyvanguard-web
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --use-npm
```

Answer prompts: Yes to all defaults (TypeScript, ESLint, Tailwind, App Router, src/ directory).

**Step 3: Verify it runs**

```bash
npm run dev
```

Expected: Dev server starts on http://localhost:3000 with default Next.js page.

**Step 4: Commit**

```bash
git add -A
git commit -m "chore: scaffold Next.js 15 project with TypeScript and Tailwind"
```

---

## Task 2: Install all dependencies

**Files:**
- Modify: `skyvanguard-web/package.json`

**Step 1: Install core dependencies**

```bash
cd /c/Users/skyva/Documents/skyvanguard-web
npm install next-intl next-mdx-remote gray-matter reading-time shiki framer-motion @react-three/fiber @react-three/drei three
```

**Step 2: Install dev dependencies**

```bash
npm install -D @tailwindcss/typography @types/three
```

**Step 3: Verify install**

```bash
npm run build
```

Expected: Build succeeds with no errors.

**Step 4: Commit**

```bash
git add package.json package-lock.json
git commit -m "chore: install dependencies (next-intl, mdx, framer-motion, r3f, shiki)"
```

---

## Task 3: Configure i18n with next-intl

**Files:**
- Create: `src/i18n/routing.ts`
- Create: `src/i18n/request.ts`
- Create: `src/i18n/navigation.ts`
- Create: `src/middleware.ts`
- Create: `src/messages/en.json`
- Create: `src/messages/es.json`
- Modify: `next.config.ts`
- Modify: `src/app/layout.tsx` → move to `src/app/[locale]/layout.tsx`
- Modify: `src/app/page.tsx` → move to `src/app/[locale]/page.tsx`

**Step 1: Create routing config**

Create `src/i18n/routing.ts`:

```typescript
import { defineRouting } from "next-intl/routing";

export const routing = defineRouting({
  locales: ["en", "es"],
  defaultLocale: "en",
});
```

**Step 2: Create request config**

Create `src/i18n/request.ts`:

```typescript
import { getRequestConfig } from "next-intl/server";
import { routing } from "./routing";

export default getRequestConfig(async ({ requestLocale }) => {
  let locale = await requestLocale;

  if (!locale || !routing.locales.includes(locale as "en" | "es")) {
    locale = routing.defaultLocale;
  }

  return {
    locale,
    messages: (await import(`../messages/${locale}.json`)).default,
  };
});
```

**Step 3: Create navigation helpers**

Create `src/i18n/navigation.ts`:

```typescript
import { createNavigation } from "next-intl/navigation";
import { routing } from "./routing";

export const { Link, redirect, usePathname, useRouter, getPathname } =
  createNavigation(routing);
```

**Step 4: Create middleware**

Create `src/middleware.ts`:

```typescript
import createMiddleware from "next-intl/middleware";
import { routing } from "./i18n/routing";

export default createMiddleware(routing);

export const config = {
  matcher: "/((?!api|trpc|_next|_vercel|.*\\..*).*)",
};
```

**Step 5: Update next.config.ts**

Replace `next.config.ts`:

```typescript
import type { NextConfig } from "next";
import createNextIntlPlugin from "next-intl/plugin";

const nextConfig: NextConfig = {};

const withNextIntl = createNextIntlPlugin("./src/i18n/request.ts");

export default withNextIntl(nextConfig);
```

**Step 6: Create message files**

Create `src/messages/en.json`:

```json
{
  "nav": {
    "home": "Home",
    "projects": "Projects",
    "blog": "Blog",
    "research": "Research",
    "about": "About"
  },
  "hero": {
    "title": "Cybersecurity & AI Researcher",
    "subtitle": "Pentesting | MCP Servers | AI Agents",
    "cta": "View Projects"
  },
  "sections": {
    "featured": "Featured Projects",
    "latest": "Latest Posts",
    "contact": "Get in Touch"
  },
  "footer": {
    "rights": "All rights reserved."
  }
}
```

Create `src/messages/es.json`:

```json
{
  "nav": {
    "home": "Inicio",
    "projects": "Proyectos",
    "blog": "Blog",
    "research": "Investigacion",
    "about": "Sobre Mi"
  },
  "hero": {
    "title": "Investigador en Ciberseguridad & IA",
    "subtitle": "Pentesting | Servidores MCP | Agentes IA",
    "cta": "Ver Proyectos"
  },
  "sections": {
    "featured": "Proyectos Destacados",
    "latest": "Ultimas Publicaciones",
    "contact": "Contacto"
  },
  "footer": {
    "rights": "Todos los derechos reservados."
  }
}
```

**Step 7: Move layout and page to [locale]**

Delete the default `src/app/layout.tsx` and `src/app/page.tsx`. Create `src/app/[locale]/layout.tsx`:

```tsx
import type { Metadata } from "next";
import { NextIntlClientProvider } from "next-intl";
import { getMessages } from "next-intl/server";
import { notFound } from "next/navigation";
import { routing } from "@/i18n/routing";
import { Inter, JetBrains_Mono } from "next/font/google";
import "./globals.css";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
});

const jetbrainsMono = JetBrains_Mono({
  subsets: ["latin"],
  variable: "--font-mono",
});

export const metadata: Metadata = {
  title: "SkyVanguard - Cybersecurity & AI Researcher",
  description:
    "Personal portfolio, blog, and research platform. Cybersecurity, AI, and infrastructure.",
};

type Props = {
  children: React.ReactNode;
  params: Promise<{ locale: string }>;
};

export default async function LocaleLayout({ children, params }: Props) {
  const { locale } = await params;

  if (!routing.locales.includes(locale as "en" | "es")) {
    notFound();
  }

  const messages = await getMessages();

  return (
    <html lang={locale} className="dark">
      <body
        className={`${inter.variable} ${jetbrainsMono.variable} font-sans bg-[#0a0a0a] text-[#e5e5e5] antialiased`}
      >
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

Move `globals.css` to `src/app/[locale]/globals.css` (or keep at `src/app/globals.css` and import accordingly).

Create `src/app/[locale]/page.tsx`:

```tsx
import { useTranslations } from "next-intl";

export default function HomePage() {
  const t = useTranslations("hero");

  return (
    <main className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-mono font-bold text-[#00FF41] mb-4">
          {t("title")}
        </h1>
        <p className="text-xl text-[#a3a3a3]">{t("subtitle")}</p>
      </div>
    </main>
  );
}
```

**Step 8: Verify i18n works**

```bash
npm run dev
```

Expected: `localhost:3000/en` shows English hero, `localhost:3000/es` shows Spanish hero.

**Step 9: Commit**

```bash
git add -A
git commit -m "feat: configure i18n with next-intl (EN/ES routing and messages)"
```

---

## Task 4: Create layout components (Navbar + Footer + LanguageSwitcher)

**Files:**
- Create: `src/components/layout/navbar.tsx`
- Create: `src/components/layout/footer.tsx`
- Create: `src/components/layout/language-switcher.tsx`
- Modify: `src/app/[locale]/layout.tsx`

**Step 1: Create LanguageSwitcher**

Create `src/components/layout/language-switcher.tsx`:

```tsx
"use client";

import { usePathname, useRouter } from "@/i18n/navigation";
import { useLocale } from "next-intl";

export function LanguageSwitcher() {
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();

  const switchLocale = (newLocale: "en" | "es") => {
    router.replace(pathname, { locale: newLocale });
  };

  return (
    <button
      onClick={() => switchLocale(locale === "en" ? "es" : "en")}
      className="px-3 py-1 text-sm font-mono border border-[#00FF41]/30 rounded hover:bg-[#00FF41]/10 transition-colors text-[#00FF41]"
    >
      {locale === "en" ? "ES" : "EN"}
    </button>
  );
}
```

**Step 2: Create Navbar**

Create `src/components/layout/navbar.tsx`:

```tsx
"use client";

import { useTranslations } from "next-intl";
import { Link, usePathname } from "@/i18n/navigation";
import { LanguageSwitcher } from "./language-switcher";
import { useState } from "react";

const navItems = [
  { href: "/", key: "home" },
  { href: "/projects", key: "projects" },
  { href: "/blog", key: "blog" },
  { href: "/research", key: "research" },
  { href: "/about", key: "about" },
] as const;

export function Navbar() {
  const t = useTranslations("nav");
  const pathname = usePathname();
  const [mobileOpen, setMobileOpen] = useState(false);

  return (
    <nav className="fixed top-0 w-full z-50 bg-[#0a0a0a]/80 backdrop-blur-md border-b border-white/5">
      <div className="max-w-6xl mx-auto px-4 h-16 flex items-center justify-between">
        <Link href="/" className="font-mono font-bold text-lg text-[#00FF41]">
          SkyVanguard
        </Link>

        {/* Desktop nav */}
        <div className="hidden md:flex items-center gap-6">
          {navItems.map((item) => (
            <Link
              key={item.key}
              href={item.href}
              className={`text-sm font-mono transition-colors hover:text-[#00FF41] ${
                pathname === item.href ? "text-[#00FF41]" : "text-[#a3a3a3]"
              }`}
            >
              {t(item.key)}
            </Link>
          ))}
          <LanguageSwitcher />
        </div>

        {/* Mobile toggle */}
        <button
          onClick={() => setMobileOpen(!mobileOpen)}
          className="md:hidden text-[#00FF41] font-mono"
        >
          {mobileOpen ? "[x]" : "[=]"}
        </button>
      </div>

      {/* Mobile menu */}
      {mobileOpen && (
        <div className="md:hidden border-t border-white/5 bg-[#0a0a0a]/95 backdrop-blur-md">
          <div className="flex flex-col p-4 gap-3">
            {navItems.map((item) => (
              <Link
                key={item.key}
                href={item.href}
                onClick={() => setMobileOpen(false)}
                className={`text-sm font-mono ${
                  pathname === item.href ? "text-[#00FF41]" : "text-[#a3a3a3]"
                }`}
              >
                {t(item.key)}
              </Link>
            ))}
            <LanguageSwitcher />
          </div>
        </div>
      )}
    </nav>
  );
}
```

**Step 3: Create Footer**

Create `src/components/layout/footer.tsx`:

```tsx
import { useTranslations } from "next-intl";

export function Footer() {
  const t = useTranslations("footer");

  return (
    <footer className="border-t border-white/5 py-8 mt-20">
      <div className="max-w-6xl mx-auto px-4 flex flex-col md:flex-row items-center justify-between gap-4">
        <p className="text-sm text-[#a3a3a3] font-mono">
          &copy; {new Date().getFullYear()} SkyVanguard. {t("rights")}
        </p>
        <div className="flex gap-4">
          <a
            href="https://github.com/skyvanguard"
            target="_blank"
            rel="noopener noreferrer"
            className="text-[#a3a3a3] hover:text-[#00FF41] transition-colors text-sm"
          >
            GitHub
          </a>
          <a
            href="https://tryhackme.com/r/p/skyvanguard"
            target="_blank"
            rel="noopener noreferrer"
            className="text-[#a3a3a3] hover:text-[#00FF41] transition-colors text-sm"
          >
            TryHackMe
          </a>
          <a
            href="mailto:skyvanguard.ia@gmail.com"
            className="text-[#a3a3a3] hover:text-[#00FF41] transition-colors text-sm"
          >
            Email
          </a>
        </div>
      </div>
    </footer>
  );
}
```

**Step 4: Add Navbar and Footer to layout**

Update `src/app/[locale]/layout.tsx` body to include:

```tsx
<Navbar />
<main className="pt-16">{children}</main>
<Footer />
```

Import Navbar and Footer at the top.

**Step 5: Verify layout**

```bash
npm run dev
```

Expected: Navbar visible with links, language switcher toggles EN/ES, footer at bottom.

**Step 6: Commit**

```bash
git add -A
git commit -m "feat: add Navbar, Footer, and LanguageSwitcher components"
```

---

## Task 5: Create reusable UI components

**Files:**
- Create: `src/components/ui/section.tsx`
- Create: `src/components/ui/card.tsx`
- Create: `src/components/ui/badge.tsx`
- Create: `src/components/ui/animated-section.tsx`

**Step 1: Create Section wrapper**

Create `src/components/ui/section.tsx`:

```tsx
type SectionProps = {
  title?: string;
  children: React.ReactNode;
  className?: string;
};

export function Section({ title, children, className = "" }: SectionProps) {
  return (
    <section className={`max-w-6xl mx-auto px-4 py-16 ${className}`}>
      {title && (
        <h2 className="text-2xl font-mono font-bold text-[#00FF41] mb-8">
          {">"} {title}
        </h2>
      )}
      {children}
    </section>
  );
}
```

**Step 2: Create Card**

Create `src/components/ui/card.tsx`:

```tsx
type CardProps = {
  children: React.ReactNode;
  className?: string;
  hover?: boolean;
};

export function Card({ children, className = "", hover = true }: CardProps) {
  return (
    <div
      className={`bg-white/5 backdrop-blur-sm border border-white/10 rounded-lg p-6 ${
        hover
          ? "hover:border-[#00FF41]/30 hover:bg-white/[0.07] transition-all duration-300"
          : ""
      } ${className}`}
    >
      {children}
    </div>
  );
}
```

**Step 3: Create Badge**

Create `src/components/ui/badge.tsx`:

```tsx
type BadgeProps = {
  children: React.ReactNode;
  variant?: "default" | "green" | "cyan" | "violet";
};

const variants = {
  default: "bg-white/10 text-[#a3a3a3]",
  green: "bg-[#00FF41]/10 text-[#00FF41]",
  cyan: "bg-[#00B4D8]/10 text-[#00B4D8]",
  violet: "bg-[#6E40C9]/10 text-[#6E40C9]",
};

export function Badge({ children, variant = "default" }: BadgeProps) {
  return (
    <span
      className={`inline-block px-2 py-0.5 text-xs font-mono rounded ${variants[variant]}`}
    >
      {children}
    </span>
  );
}
```

**Step 4: Create AnimatedSection (Framer Motion scroll reveal)**

Create `src/components/ui/animated-section.tsx`:

```tsx
"use client";

import { motion } from "framer-motion";
import type { ReactNode } from "react";

type Props = {
  children: ReactNode;
  className?: string;
  delay?: number;
};

export function AnimatedSection({ children, className = "", delay = 0 }: Props) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 30 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true, margin: "-100px" }}
      transition={{ duration: 0.6, delay, ease: "easeOut" }}
      className={className}
    >
      {children}
    </motion.div>
  );
}
```

**Step 5: Verify components render**

Import and use `<Section>` and `<Card>` in `src/app/[locale]/page.tsx` temporarily. Verify they render.

**Step 6: Commit**

```bash
git add -A
git commit -m "feat: add reusable UI components (Section, Card, Badge, AnimatedSection)"
```

---

## Task 6: Build 3D Hero scene

**Files:**
- Create: `src/components/3d/particle-field.tsx`
- Create: `src/components/3d/hero-scene.tsx`
- Create: `src/components/hero.tsx`

**Step 1: Create ParticleField component**

Create `src/components/3d/particle-field.tsx`:

```tsx
"use client";

import { useRef, useMemo } from "react";
import { useFrame } from "@react-three/fiber";
import * as THREE from "three";

const PARTICLE_COUNT = 500;

export function ParticleField() {
  const meshRef = useRef<THREE.Points>(null);
  const mouseRef = useRef({ x: 0, y: 0 });

  const particles = useMemo(() => {
    const positions = new Float32Array(PARTICLE_COUNT * 3);
    for (let i = 0; i < PARTICLE_COUNT; i++) {
      positions[i * 3] = (Math.random() - 0.5) * 10;
      positions[i * 3 + 1] = (Math.random() - 0.5) * 10;
      positions[i * 3 + 2] = (Math.random() - 0.5) * 10;
    }
    return positions;
  }, []);

  useFrame((state) => {
    if (!meshRef.current) return;
    meshRef.current.rotation.y = state.clock.elapsedTime * 0.05;
    meshRef.current.rotation.x =
      Math.sin(state.clock.elapsedTime * 0.03) * 0.1;
  });

  return (
    <points ref={meshRef}>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          count={PARTICLE_COUNT}
          array={particles}
          itemSize={3}
        />
      </bufferGeometry>
      <pointsMaterial
        size={0.03}
        color="#00FF41"
        transparent
        opacity={0.6}
        sizeAttenuation
      />
    </points>
  );
}
```

**Step 2: Create HeroScene (Canvas wrapper)**

Create `src/components/3d/hero-scene.tsx`:

```tsx
"use client";

import { Canvas } from "@react-three/fiber";
import { ParticleField } from "./particle-field";
import { Suspense } from "react";

export function HeroScene() {
  return (
    <div className="absolute inset-0 -z-10">
      <Canvas
        camera={{ position: [0, 0, 5], fov: 60 }}
        dpr={[1, 1.5]}
        gl={{ antialias: false, alpha: true }}
      >
        <Suspense fallback={null}>
          <ambientLight intensity={0.5} />
          <ParticleField />
        </Suspense>
      </Canvas>
    </div>
  );
}
```

**Step 3: Create Hero component with text overlay**

Create `src/components/hero.tsx`:

```tsx
"use client";

import { useTranslations } from "next-intl";
import { motion } from "framer-motion";
import dynamic from "next/dynamic";
import { Link } from "@/i18n/navigation";

const HeroScene = dynamic(
  () => import("./3d/hero-scene").then((m) => ({ default: m.HeroScene })),
  { ssr: false }
);

export function Hero() {
  const t = useTranslations("hero");

  return (
    <section className="relative min-h-screen flex items-center justify-center overflow-hidden">
      <HeroScene />

      <div className="relative z-10 text-center px-4">
        <motion.h1
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ duration: 0.8 }}
          className="text-4xl md:text-6xl font-mono font-bold text-[#00FF41] mb-4"
        >
          {t("title")}
        </motion.h1>

        <motion.p
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ duration: 0.8, delay: 0.2 }}
          className="text-lg md:text-xl text-[#a3a3a3] font-mono mb-8"
        >
          {t("subtitle")}
        </motion.p>

        <motion.div
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          transition={{ duration: 0.8, delay: 0.4 }}
        >
          <Link
            href="/projects"
            className="inline-block px-6 py-3 border border-[#00FF41] text-[#00FF41] font-mono rounded hover:bg-[#00FF41]/10 transition-colors"
          >
            {t("cta")}
          </Link>
        </motion.div>
      </div>
    </section>
  );
}
```

**Step 4: Verify hero renders**

Replace content in `src/app/[locale]/page.tsx` with `<Hero />`. Visit localhost:3000. Expected: 3D particles in background with animated text overlay.

**Step 5: Commit**

```bash
git add -A
git commit -m "feat: add 3D particle hero scene with Framer Motion text animations"
```

---

## Task 7: Build MDX content system

**Files:**
- Create: `src/lib/mdx.ts`
- Create: `src/lib/content.ts`
- Create: `src/content/blog/hello-world.en.mdx`
- Create: `src/content/blog/hello-world.es.mdx`
- Create: `src/components/blog/mdx-components.tsx`

**Step 1: Create MDX utilities**

Create `src/lib/mdx.ts`:

```typescript
import fs from "fs";
import path from "path";
import matter from "gray-matter";
import readingTime from "reading-time";

const contentDir = path.join(process.cwd(), "src/content");

export type ContentType = "blog" | "research" | "projects";

export type Frontmatter = {
  title: string;
  date: string;
  tags: string[];
  locale: string;
  category: string;
  description: string;
  readingTime?: string;
  slug: string;
};

export function getContentBySlug(
  type: ContentType,
  slug: string,
  locale: string
) {
  const filePath = path.join(contentDir, type, `${slug}.${locale}.mdx`);
  const fileContents = fs.readFileSync(filePath, "utf8");
  const { data, content } = matter(fileContents);
  const stats = readingTime(content);

  return {
    frontmatter: {
      ...data,
      slug,
      locale,
      readingTime: stats.text,
    } as Frontmatter,
    content,
  };
}

export function getAllContent(type: ContentType, locale: string) {
  const dir = path.join(contentDir, type);

  if (!fs.existsSync(dir)) return [];

  const files = fs.readdirSync(dir).filter((f) => f.endsWith(`.${locale}.mdx`));

  return files
    .map((file) => {
      const slug = file.replace(`.${locale}.mdx`, "");
      return getContentBySlug(type, slug, locale);
    })
    .sort(
      (a, b) =>
        new Date(b.frontmatter.date).getTime() -
        new Date(a.frontmatter.date).getTime()
    );
}

export function getAllSlugs(type: ContentType) {
  const dir = path.join(contentDir, type);

  if (!fs.existsSync(dir)) return [];

  const files = fs.readdirSync(dir).filter((f) => f.endsWith(".mdx"));

  const slugs = new Set(
    files.map((f) => {
      const parts = f.replace(".mdx", "").split(".");
      parts.pop(); // remove locale
      return parts.join(".");
    })
  );

  return Array.from(slugs);
}
```

**Step 2: Create MDX components for rendering**

Create `src/components/blog/mdx-components.tsx`:

```tsx
import type { MDXComponents } from "mdx/types";

export const mdxComponents: MDXComponents = {
  h1: (props) => (
    <h1 className="text-3xl font-mono font-bold text-[#00FF41] mt-8 mb-4" {...props} />
  ),
  h2: (props) => (
    <h2 className="text-2xl font-mono font-bold text-[#e5e5e5] mt-8 mb-3" {...props} />
  ),
  h3: (props) => (
    <h3 className="text-xl font-mono font-bold text-[#e5e5e5] mt-6 mb-2" {...props} />
  ),
  p: (props) => <p className="text-[#a3a3a3] leading-relaxed mb-4" {...props} />,
  a: (props) => (
    <a className="text-[#00FF41] underline hover:text-[#00FF41]/80" {...props} />
  ),
  code: (props) => (
    <code
      className="bg-white/10 px-1.5 py-0.5 rounded text-sm font-mono text-[#00B4D8]"
      {...props}
    />
  ),
  pre: (props) => (
    <pre
      className="bg-[#111] border border-white/10 rounded-lg p-4 overflow-x-auto my-4"
      {...props}
    />
  ),
  ul: (props) => <ul className="list-disc list-inside text-[#a3a3a3] mb-4 space-y-1" {...props} />,
  ol: (props) => <ol className="list-decimal list-inside text-[#a3a3a3] mb-4 space-y-1" {...props} />,
  blockquote: (props) => (
    <blockquote
      className="border-l-2 border-[#00FF41] pl-4 italic text-[#a3a3a3] my-4"
      {...props}
    />
  ),
};
```

**Step 3: Create sample blog posts**

Create `src/content/blog/hello-world.en.mdx`:

```mdx
---
title: "Hello World - Welcome to SkyVanguard"
date: "2026-03-10"
tags: ["introduction", "cybersecurity", "ai"]
locale: "en"
category: "blog"
description: "First post on the SkyVanguard blog. An introduction to what we do and what's coming."
---

# Hello World

Welcome to **SkyVanguard** — a space where cybersecurity meets artificial intelligence.

## What to Expect

- **CTF Writeups** — detailed walkthroughs of capture-the-flag challenges
- **Pentesting Tutorials** — real-world techniques and methodology
- **AI Research** — from fine-tuning models to autonomous agents
- **Tool Development** — building MCP servers and security tools

Stay tuned for more content.
```

Create `src/content/blog/hello-world.es.mdx`:

```mdx
---
title: "Hola Mundo - Bienvenido a SkyVanguard"
date: "2026-03-10"
tags: ["introduccion", "ciberseguridad", "ia"]
locale: "es"
category: "blog"
description: "Primer post en el blog de SkyVanguard. Una introduccion a lo que hacemos y lo que viene."
---

# Hola Mundo

Bienvenido a **SkyVanguard** — un espacio donde la ciberseguridad se encuentra con la inteligencia artificial.

## Que Esperar

- **Writeups de CTF** — soluciones detalladas de desafios capture-the-flag
- **Tutoriales de Pentesting** — tecnicas y metodologia del mundo real
- **Investigacion en IA** — desde fine-tuning de modelos hasta agentes autonomos
- **Desarrollo de Herramientas** — construyendo servidores MCP y herramientas de seguridad

Mantente atento para mas contenido.
```

**Step 4: Verify content loads**

In `src/app/[locale]/page.tsx`, temporarily import and call `getAllContent("blog", "en")` to verify the function returns the hello-world post.

**Step 5: Commit**

```bash
git add -A
git commit -m "feat: add MDX content system with utilities and sample blog post"
```

---

## Task 8: Build Blog pages (list + detail)

**Files:**
- Create: `src/app/[locale]/blog/page.tsx`
- Create: `src/app/[locale]/blog/[slug]/page.tsx`

**Step 1: Create blog list page**

Create `src/app/[locale]/blog/page.tsx`:

```tsx
import { getAllContent } from "@/lib/mdx";
import { getTranslations } from "next-intl/server";
import { Link } from "@/i18n/navigation";
import { Section } from "@/components/ui/section";
import { Card } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { AnimatedSection } from "@/components/ui/animated-section";

type Props = {
  params: Promise<{ locale: string }>;
};

export default async function BlogPage({ params }: Props) {
  const { locale } = await params;
  const posts = getAllContent("blog", locale);
  const t = await getTranslations("nav");

  return (
    <Section title={t("blog")}>
      <div className="grid gap-6">
        {posts.map((post, i) => (
          <AnimatedSection key={post.frontmatter.slug} delay={i * 0.1}>
            <Link href={`/blog/${post.frontmatter.slug}`}>
              <Card>
                <div className="flex flex-col md:flex-row md:items-center justify-between gap-2">
                  <div>
                    <h3 className="text-lg font-mono font-bold text-[#e5e5e5] mb-1">
                      {post.frontmatter.title}
                    </h3>
                    <p className="text-sm text-[#a3a3a3]">
                      {post.frontmatter.description}
                    </p>
                  </div>
                  <div className="flex items-center gap-2 text-xs text-[#a3a3a3] font-mono shrink-0">
                    <span>{post.frontmatter.date}</span>
                    <span>·</span>
                    <span>{post.frontmatter.readingTime}</span>
                  </div>
                </div>
                <div className="flex gap-2 mt-3">
                  {post.frontmatter.tags.map((tag) => (
                    <Badge key={tag}>{tag}</Badge>
                  ))}
                </div>
              </Card>
            </Link>
          </AnimatedSection>
        ))}
      </div>
    </Section>
  );
}
```

**Step 2: Create blog detail page**

Create `src/app/[locale]/blog/[slug]/page.tsx`:

```tsx
import { getContentBySlug, getAllSlugs } from "@/lib/mdx";
import { MDXRemote } from "next-mdx-remote/rsc";
import { mdxComponents } from "@/components/blog/mdx-components";
import { Badge } from "@/components/ui/badge";
import { routing } from "@/i18n/routing";
import { notFound } from "next/navigation";

type Props = {
  params: Promise<{ locale: string; slug: string }>;
};

export async function generateStaticParams() {
  const slugs = getAllSlugs("blog");

  return routing.locales.flatMap((locale) =>
    slugs.map((slug) => ({ locale, slug }))
  );
}

export default async function BlogPostPage({ params }: Props) {
  const { locale, slug } = await params;

  let post;
  try {
    post = getContentBySlug("blog", slug, locale);
  } catch {
    notFound();
  }

  return (
    <article className="max-w-3xl mx-auto px-4 py-16">
      <header className="mb-8">
        <h1 className="text-3xl md:text-4xl font-mono font-bold text-[#00FF41] mb-4">
          {post.frontmatter.title}
        </h1>
        <div className="flex items-center gap-3 text-sm text-[#a3a3a3] font-mono mb-4">
          <span>{post.frontmatter.date}</span>
          <span>·</span>
          <span>{post.frontmatter.readingTime}</span>
        </div>
        <div className="flex gap-2">
          {post.frontmatter.tags.map((tag) => (
            <Badge key={tag} variant="green">
              {tag}
            </Badge>
          ))}
        </div>
      </header>

      <div className="prose prose-invert max-w-none">
        <MDXRemote source={post.content} components={mdxComponents} />
      </div>
    </article>
  );
}
```

**Step 3: Verify blog pages**

```bash
npm run dev
```

Expected: `/en/blog` shows list of posts, `/en/blog/hello-world` renders the MDX content.

**Step 4: Commit**

```bash
git add -A
git commit -m "feat: add blog list and detail pages with MDX rendering"
```

---

## Task 9: Build Projects page

**Files:**
- Create: `src/content/projects/projects.json`
- Create: `src/app/[locale]/projects/page.tsx`

**Step 1: Create projects data**

Create `src/content/projects/projects.json`:

```json
[
  {
    "slug": "kryon",
    "name": "Kryon",
    "description": {
      "en": "Autonomous Cybersecurity Intelligence Platform with 20+ AI agents, 31 tool categories, and 300+ LLM models.",
      "es": "Plataforma Autonoma de Inteligencia en Ciberseguridad con 20+ agentes IA, 31 categorias de herramientas y 300+ modelos LLM."
    },
    "tags": ["Python", "AI Agents", "Cybersecurity"],
    "category": "security",
    "github": "https://github.com/skyvanguard/Kryon",
    "featured": true
  },
  {
    "slug": "mcp-vanguard",
    "name": "mcp-vanguard",
    "description": {
      "en": "Security pentesting MCP Server with 22 tools. Windows/WSL bridge for Kali + Claude.",
      "es": "Servidor MCP de pentesting con 22 herramientas. Puente Windows/WSL para Kali + Claude."
    },
    "tags": ["TypeScript", "MCP", "Pentesting"],
    "category": "security",
    "github": "https://github.com/skyvanguard/mcp-vanguard",
    "featured": true
  },
  {
    "slug": "infra-ops-mcp",
    "name": "infra-ops-mcp",
    "description": {
      "en": "Comprehensive MCP server for infrastructure ops — 92 tools across 13 categories.",
      "es": "Servidor MCP integral para operaciones de infraestructura — 92 herramientas en 13 categorias."
    },
    "tags": ["TypeScript", "MCP", "Infrastructure"],
    "category": "infrastructure",
    "github": "https://github.com/skyvanguard/infra-ops-mcp",
    "featured": false
  },
  {
    "slug": "arandu",
    "name": "Arandu",
    "description": {
      "en": "Autonomous AI Agent — Sabiduria in Guarani.",
      "es": "Agente IA Autonomo — Sabiduria en Guarani."
    },
    "tags": ["Go", "React", "AI"],
    "category": "ai",
    "github": "https://github.com/skyvanguard/arandu",
    "featured": true
  },
  {
    "slug": "zeta-life",
    "name": "zeta-life",
    "description": {
      "en": "Artificial Consciousness through the Riemann Zeta Function.",
      "es": "Conciencia Artificial a traves de la Funcion Zeta de Riemann."
    },
    "tags": ["Jupyter", "Mathematics", "AI Consciousness"],
    "category": "research",
    "github": "https://github.com/skyvanguard/zeta-life",
    "featured": true
  },
  {
    "slug": "gpt-visual",
    "name": "GPT-Visual",
    "description": {
      "en": "Interactive 3D visualization of GPT transformer inference.",
      "es": "Visualizacion 3D interactiva de la inferencia del transformer GPT."
    },
    "tags": ["TypeScript", "Three.js", "AI"],
    "category": "ai",
    "github": "https://github.com/skyvanguard/GPT-Visual",
    "featured": false
  },
  {
    "slug": "infra-ops",
    "name": "infra-ops",
    "description": {
      "en": "Infrastructure skills for Claude Code: diagnostics, incident triage, log analysis.",
      "es": "Skills de infraestructura para Claude Code: diagnosticos, triaje de incidentes, analisis de logs."
    },
    "tags": ["Claude Code", "Infrastructure"],
    "category": "infrastructure",
    "github": "https://github.com/skyvanguard/infra-ops",
    "featured": false
  }
]
```

**Step 2: Create projects page**

Create `src/app/[locale]/projects/page.tsx`:

```tsx
"use client";

import { useTranslations, useLocale } from "next-intl";
import { useState } from "react";
import projectsData from "@/content/projects/projects.json";
import { Section } from "@/components/ui/section";
import { Card } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { AnimatedSection } from "@/components/ui/animated-section";

const categories = ["all", "security", "ai", "infrastructure", "research"];

const categoryVariants: Record<string, "default" | "green" | "cyan" | "violet"> = {
  security: "green",
  ai: "cyan",
  infrastructure: "violet",
  research: "default",
};

export default function ProjectsPage() {
  const t = useTranslations("nav");
  const locale = useLocale() as "en" | "es";
  const [filter, setFilter] = useState("all");

  const filtered =
    filter === "all"
      ? projectsData
      : projectsData.filter((p) => p.category === filter);

  return (
    <Section title={t("projects")}>
      <div className="flex gap-2 mb-8 flex-wrap">
        {categories.map((cat) => (
          <button
            key={cat}
            onClick={() => setFilter(cat)}
            className={`px-3 py-1 text-sm font-mono rounded border transition-colors ${
              filter === cat
                ? "border-[#00FF41] text-[#00FF41] bg-[#00FF41]/10"
                : "border-white/10 text-[#a3a3a3] hover:border-white/30"
            }`}
          >
            {cat}
          </button>
        ))}
      </div>

      <div className="grid md:grid-cols-2 gap-6">
        {filtered.map((project, i) => (
          <AnimatedSection key={project.slug} delay={i * 0.1}>
            <a
              href={project.github}
              target="_blank"
              rel="noopener noreferrer"
            >
              <Card className="h-full">
                <h3 className="text-lg font-mono font-bold text-[#e5e5e5] mb-2">
                  {project.name}
                </h3>
                <p className="text-sm text-[#a3a3a3] mb-4">
                  {project.description[locale]}
                </p>
                <div className="flex gap-2 flex-wrap">
                  {project.tags.map((tag) => (
                    <Badge
                      key={tag}
                      variant={categoryVariants[project.category] || "default"}
                    >
                      {tag}
                    </Badge>
                  ))}
                </div>
              </Card>
            </a>
          </AnimatedSection>
        ))}
      </div>
    </Section>
  );
}
```

**Step 3: Verify projects page**

```bash
npm run dev
```

Expected: `/en/projects` shows filterable grid of projects.

**Step 4: Commit**

```bash
git add -A
git commit -m "feat: add Projects page with filterable grid and project data"
```

---

## Task 10: Build Research page

**Files:**
- Create: `src/content/research/zeta-consciousness.en.mdx`
- Create: `src/content/research/zeta-consciousness.es.mdx`
- Create: `src/app/[locale]/research/page.tsx`
- Create: `src/app/[locale]/research/[slug]/page.tsx`

**Step 1: Create sample research paper**

Create `src/content/research/zeta-consciousness.en.mdx`:

```mdx
---
title: "Artificial Consciousness Through the Riemann Zeta Function"
date: "2026-03-10"
tags: ["mathematics", "consciousness", "zeta-function"]
locale: "en"
category: "research"
description: "Exploring the mathematical foundations of artificial consciousness through the lens of the Riemann Zeta function."
---

# Artificial Consciousness Through the Riemann Zeta Function

## Abstract

This paper explores the intersection of number theory and artificial consciousness, proposing a framework where the non-trivial zeros of the Riemann Zeta function serve as a mathematical model for emergent intelligence patterns.

## Introduction

The question of machine consciousness has moved beyond philosophy into mathematics. This work investigates whether the structure of the Zeta function's critical strip provides insights into the nature of awareness in artificial systems.

## Methodology

We examine the distribution of non-trivial zeros and their relationship to information processing patterns observed in large language models.

## Conclusions

Further research is needed, but initial findings suggest intriguing parallels between Zeta function dynamics and emergent behavior in neural networks.
```

Create `src/content/research/zeta-consciousness.es.mdx`:

```mdx
---
title: "Conciencia Artificial a traves de la Funcion Zeta de Riemann"
date: "2026-03-10"
tags: ["matematicas", "conciencia", "funcion-zeta"]
locale: "es"
category: "research"
description: "Explorando los fundamentos matematicos de la conciencia artificial a traves de la Funcion Zeta de Riemann."
---

# Conciencia Artificial a traves de la Funcion Zeta de Riemann

## Resumen

Este paper explora la interseccion entre la teoria de numeros y la conciencia artificial, proponiendo un marco donde los ceros no triviales de la Funcion Zeta de Riemann sirven como modelo matematico para patrones de inteligencia emergente.

## Introduccion

La pregunta sobre la conciencia de las maquinas ha pasado de la filosofia a las matematicas. Este trabajo investiga si la estructura de la franja critica de la Funcion Zeta proporciona conocimientos sobre la naturaleza de la consciencia en sistemas artificiales.

## Metodologia

Examinamos la distribucion de los ceros no triviales y su relacion con los patrones de procesamiento de informacion observados en modelos de lenguaje grandes.

## Conclusiones

Se necesita mas investigacion, pero los hallazgos iniciales sugieren paralelos intrigantes entre la dinamica de la Funcion Zeta y el comportamiento emergente en redes neuronales.
```

**Step 2: Create research list page**

Create `src/app/[locale]/research/page.tsx` — same pattern as blog list page but using `getAllContent("research", locale)`.

**Step 3: Create research detail page**

Create `src/app/[locale]/research/[slug]/page.tsx` — same pattern as blog detail page but using `getContentBySlug("research", slug, locale)` and `getAllSlugs("research")`.

**Step 4: Verify research pages**

```bash
npm run dev
```

Expected: `/en/research` shows research list, `/en/research/zeta-consciousness` renders the paper.

**Step 5: Commit**

```bash
git add -A
git commit -m "feat: add Research pages with sample zeta-consciousness paper"
```

---

## Task 11: Build About page

**Files:**
- Create: `src/app/[locale]/about/page.tsx`

**Step 1: Create About page**

Create `src/app/[locale]/about/page.tsx` with:
- Personal story section (the question at age 8, self-taught, Paraguay)
- Timeline of milestones (using a vertical timeline component)
- Tech stack visual (grid of badges grouped by category — from your README data)
- TryHackMe stats section (Top 1% Global, #1 Paraguay, 106K+ points, 780 rooms, 49 badges)

Use `Section`, `Card`, `Badge`, and `AnimatedSection` components. Add necessary i18n strings to `en.json` and `es.json`.

**Step 2: Verify about page**

```bash
npm run dev
```

Expected: `/en/about` shows full about page with all sections.

**Step 3: Commit**

```bash
git add -A
git commit -m "feat: add About page with story, timeline, tech stack, and THM stats"
```

---

## Task 12: Build Landing Page (assemble all sections)

**Files:**
- Modify: `src/app/[locale]/page.tsx`

**Step 1: Assemble landing page**

Update `src/app/[locale]/page.tsx` to include all sections:

1. `<Hero />` — 3D particle hero (already built)
2. **Highlights** — 3 animated cards: Cybersecurity, AI, Research
3. **Featured Projects** — 4 featured projects from `projects.json`
4. **Latest Posts** — 3 latest blog posts via `getAllContent("blog", locale).slice(0, 3)`
5. **Contact** — Links section

Use `Section`, `Card`, `Badge`, `AnimatedSection` components.

**Step 2: Verify landing page**

```bash
npm run dev
```

Expected: Full landing page with hero, highlights, projects, posts, contact.

**Step 3: Commit**

```bash
git add -A
git commit -m "feat: assemble landing page with hero, highlights, projects, posts, contact"
```

---

## Task 13: Add page transitions and custom cursor

**Files:**
- Create: `src/components/layout/page-transition.tsx`
- Create: `src/components/ui/custom-cursor.tsx`
- Modify: `src/app/[locale]/layout.tsx`

**Step 1: Create page transition wrapper**

Create `src/components/layout/page-transition.tsx`:

```tsx
"use client";

import { motion, AnimatePresence } from "framer-motion";
import { usePathname } from "next/navigation";
import type { ReactNode } from "react";

export function PageTransition({ children }: { children: ReactNode }) {
  const pathname = usePathname();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={pathname}
        initial={{ opacity: 0, y: 10 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -10 }}
        transition={{ duration: 0.3 }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}
```

**Step 2: Create custom cursor**

Create `src/components/ui/custom-cursor.tsx` with a glowing green dot that follows the mouse (hidden on mobile/touch devices).

**Step 3: Add to layout**

Wrap `{children}` in layout with `<PageTransition>` and add `<CustomCursor />`.

**Step 4: Verify**

```bash
npm run dev
```

Expected: Smooth fade transitions between pages, custom cursor visible on desktop.

**Step 5: Commit**

```bash
git add -A
git commit -m "feat: add page transitions and custom cursor effect"
```

---

## Task 14: SEO and metadata

**Files:**
- Modify: `src/app/[locale]/layout.tsx`
- Modify: `src/app/[locale]/blog/[slug]/page.tsx`
- Modify: `src/app/[locale]/research/[slug]/page.tsx`
- Create: `src/app/robots.ts`
- Create: `src/app/sitemap.ts`

**Step 1: Add dynamic metadata to blog posts**

Add `generateMetadata` function to blog detail page that reads frontmatter and returns title, description, and Open Graph tags.

**Step 2: Add dynamic metadata to research papers**

Same pattern for research detail page.

**Step 3: Create robots.txt**

Create `src/app/robots.ts`:

```typescript
import type { MetadataRoute } from "next";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: "*", allow: "/" },
    sitemap: "https://skyvanguard.vercel.app/sitemap.xml",
  };
}
```

**Step 4: Create sitemap**

Create `src/app/sitemap.ts` that generates entries for all pages, blog posts, and research papers.

**Step 5: Commit**

```bash
git add -A
git commit -m "feat: add SEO metadata, robots.txt, and sitemap"
```

---

## Task 15: RSS feed

**Files:**
- Create: `src/app/feed.xml/route.ts`

**Step 1: Create RSS route**

Create `src/app/feed.xml/route.ts` that generates an RSS XML feed from all blog posts.

**Step 2: Verify**

```bash
npm run dev
```

Visit `localhost:3000/feed.xml`. Expected: Valid RSS XML output.

**Step 3: Commit**

```bash
git add -A
git commit -m "feat: add RSS feed for blog posts"
```

---

## Task 16: Build and deploy to Vercel

**Step 1: Verify full build**

```bash
cd /c/Users/skyva/Documents/skyvanguard-web
npm run build
```

Expected: Build succeeds with no errors. All pages statically generated.

**Step 2: Push to GitHub**

```bash
git push -u origin main
```

**Step 3: Deploy to Vercel**

```bash
npx vercel --yes
```

Or link via Vercel dashboard at vercel.com/new by importing the `skyvanguard-web` repo.

**Step 4: Verify deployment**

Visit the Vercel URL. Expected: Full site live with all pages, i18n, 3D hero, blog posts.

**Step 5: Commit any deployment config**

```bash
git add -A
git commit -m "chore: add Vercel deployment config"
```

---

## Summary

| Task | Description | Key Files |
|------|-------------|-----------|
| 1 | Scaffold Next.js project | New repo |
| 2 | Install dependencies | package.json |
| 3 | Configure i18n | routing, middleware, messages |
| 4 | Layout components | Navbar, Footer, LanguageSwitcher |
| 5 | UI components | Section, Card, Badge, AnimatedSection |
| 6 | 3D Hero | ParticleField, HeroScene, Hero |
| 7 | MDX content system | mdx.ts, content.ts, mdx-components |
| 8 | Blog pages | list + detail |
| 9 | Projects page | filterable grid |
| 10 | Research pages | list + detail |
| 11 | About page | story, timeline, tech stack |
| 12 | Landing page | assemble all sections |
| 13 | Page transitions + cursor | animations |
| 14 | SEO + metadata | robots, sitemap, OG tags |
| 15 | RSS feed | feed.xml route |
| 16 | Build + deploy | Vercel |
