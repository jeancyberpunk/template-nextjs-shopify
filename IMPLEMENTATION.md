# Implementation: PostHog, Framer Motion & Font Awesome

## Overview

This document covers the integration of three libraries into the Next.js 15 + Supabase starter:

| Library | Purpose | Status |
|---------|---------|--------|
| [PostHog](https://posthog.com) | Product analytics & event tracking | Newly added |
| [Framer Motion](https://www.framer.com/motion/) | Declarative animations | Already installed |
| [Font Awesome](https://fontawesome.com) | Icon library | Newly added |

---

## PostHog

### Packages

```
posthog-js
```

### Environment Variables

Add the following to your `.env.local`:

```env
NEXT_PUBLIC_POSTHOG_KEY=your-posthog-project-api-key
NEXT_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
```

You can find your project API key in the PostHog dashboard under **Project Settings**.

> If `NEXT_PUBLIC_POSTHOG_KEY` is not set, PostHog is completely disabled — no scripts load and the provider renders children directly.

### Files

#### `app/components/posthog-provider.tsx`

Client component that initializes the PostHog SDK and wraps the app with `PostHogProvider` from `posthog-js/react`. Configuration:

- `person_profiles: "identified_only"` — only creates person profiles for identified (logged-in) users to stay within free-tier limits.
- `capture_pageview: false` — disables automatic pageview capture in favor of the manual tracker below, which handles Next.js client-side navigation correctly.

#### `app/components/posthog-pageview.tsx`

Client component that listens to `pathname` and `searchParams` changes via Next.js hooks and fires a `$pageview` event on each navigation. Wrapped in `<Suspense>` in the layout because `useSearchParams()` requires it.

#### `app/layout.tsx`

The root layout wraps the entire app with `<PostHogProvider>` and includes `<PostHogPageview />` inside a `<Suspense>` boundary:

```tsx
<PostHogProvider>
  <Suspense>
    <PostHogPageview />
  </Suspense>
  <ThemeProvider ...>
    {children}
  </ThemeProvider>
</PostHogProvider>
```

### Usage

**Track a custom event:**

```tsx
"use client";

import { usePostHog } from "posthog-js/react";

export function SignupButton() {
  const posthog = usePostHog();

  return (
    <button onClick={() => posthog.capture("signup_clicked")}>
      Sign Up
    </button>
  );
}
```

**Identify a user (e.g. after login):**

```tsx
posthog.identify(user.id, {
  email: user.email,
  name: user.name,
});
```

**Feature flags:**

```tsx
import { useFeatureFlagEnabled } from "posthog-js/react";

const showBanner = useFeatureFlagEnabled("show-banner");
```

---

## Font Awesome

### Packages

```
@fortawesome/fontawesome-svg-core
@fortawesome/free-solid-svg-icons
@fortawesome/free-brands-svg-icons
@fortawesome/react-fontawesome
```

### Files

#### `lib/fontawesome.ts`

Configures Font Awesome for SSR compatibility:

```ts
import { config } from "@fortawesome/fontawesome-svg-core";
import "@fortawesome/fontawesome-svg-core/styles.css";

config.autoAddCss = false;
```

- **`autoAddCss = false`** prevents Font Awesome from injecting a `<style>` tag at runtime, which causes a flash of unstyled icons on initial load.
- The CSS is imported statically instead, so it's included in the Next.js build and available immediately.

This file is imported in `app/layout.tsx` so the config runs once at app startup.

### Usage

**Solid icons:**

```tsx
import { FontAwesomeIcon } from "@fortawesome/react-fontawesome";
import { faUser, faHeart, faCheck } from "@fortawesome/free-solid-svg-icons";

<FontAwesomeIcon icon={faUser} className="h-4 w-4" />
<FontAwesomeIcon icon={faHeart} className="h-5 w-5 text-red-500" />
```

**Brand icons:**

```tsx
import { faGithub, faTwitter } from "@fortawesome/free-brands-svg-icons";

<FontAwesomeIcon icon={faGithub} className="h-5 w-5" />
```

**Sizing with Tailwind:** Use `className` with Tailwind utility classes (`h-4 w-4`, `text-lg`, etc.) to control size and color.

---

## Framer Motion

### Packages

```
framer-motion (already installed at ^12.25.0)
```

No additional configuration is needed. Framer Motion works out of the box with Next.js.

> **Note:** Framer Motion components use client-side APIs, so they must be used in client components (files with `"use client"`).

### Usage

**Fade in:**

```tsx
"use client";

import { motion } from "framer-motion";

export function FadeIn({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.4 }}
    >
      {children}
    </motion.div>
  );
}
```

**Animate on scroll (viewport):**

```tsx
<motion.div
  initial={{ opacity: 0 }}
  whileInView={{ opacity: 1 }}
  viewport={{ once: true }}
>
  Appears when scrolled into view
</motion.div>
```

**Page transitions with `AnimatePresence`:**

```tsx
"use client";

import { AnimatePresence, motion } from "framer-motion";
import { usePathname } from "next/navigation";

export function PageTransition({ children }: { children: React.ReactNode }) {
  const pathname = usePathname();

  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={pathname}
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={{ opacity: 0 }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}
```

**Staggered list:**

```tsx
const container = {
  hidden: {},
  show: { transition: { staggerChildren: 0.1 } },
};

const item = {
  hidden: { opacity: 0, y: 20 },
  show: { opacity: 1, y: 0 },
};

<motion.ul variants={container} initial="hidden" animate="show">
  {items.map((i) => (
    <motion.li key={i} variants={item}>{i}</motion.li>
  ))}
</motion.ul>
```

---

## File Summary

| File | Type | Purpose |
|------|------|---------|
| `app/layout.tsx` | Modified | Added PostHog provider, pageview tracker, and Font Awesome config import |
| `app/components/posthog-provider.tsx` | New | PostHog SDK initialization and React context provider |
| `app/components/posthog-pageview.tsx` | New | Automatic pageview tracking on route changes |
| `lib/fontawesome.ts` | New | Font Awesome SSR configuration |
| `.env.example` | Modified | Added PostHog environment variable templates |
