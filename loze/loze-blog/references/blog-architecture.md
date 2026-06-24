# Next.js Blog Architecture — Reference

## Component Tree

```
RootLayout (server)
├── ThemeProvider (client)
├── Header (client) — nav links, theme toggle, search trigger
├── {children}
│   ├── HomePage (server) — Hero + FeaturedPosts + RecentTimeline
│   ├── BlogListPage (server) → BlogClientPage (client)
│   │   ├── WriteDialog (client)
│   │   ├── ContentManager (client)
│   │   ├── PostCard (client)
│   │   └── TagCloud (client)
│   ├── ArticlePage (server)
│   │   ├── ReadingProgress (client)
│   │   └── TableOfContents (client)
│   ├── DiaryPage (server) → DiaryClientPage (client)
│   ├── PhotosPage (server) → PhotosClientPage (client)
│   ├── FriendsPage (server) → FriendsList (client)
│   └── NotesPage (server)
├── Footer (server)
└── QuickActions (client) — floating ⊕ button
```

## Data Flow

```
content/posts/*.mdx
  → posts.ts (gray-matter parse)
    → getAllPosts() / getPostBySlug()
      → page.tsx (server) — fetch data
        → client-page.tsx (client) — render + interact
          → API routes — write/delete
            → fs.writeFileSync / fs.unlinkSync
```

## Key TypeScript Interfaces

```typescript
interface PostFrontmatter {
  title: string; description?: string; date: string;
  tags?: string[]; category?: string; cover?: string;
  draft?: boolean; featured?: boolean; toc?: boolean;
}
interface Post {
  slug: string; frontmatter: PostFrontmatter; content: string;
}
```

## Dark Mode Architecture

```
ThemeProvider
  ├── Reads localStorage("theme") || system preference
  ├── Sets <html class="dark"> (or removes it)
  └── Provides useTheme() hook → { theme, toggleTheme }

globals.css
  ├── :root { --bg: #faf8f5; ... }     ← light defaults
  └── .dark { --bg: #2c2416; ... }     ← dark overrides

Components use:
  - CSS variables: bg-bg, text-text, border-border, text-text-muted
  - Tailwind dark: variant: dark:bg-zinc-900
```

## Build Output

Route types:
- ○ Static — pre-rendered at build time (/, /about, /diary, /friends, /notes, /photos)
- ● SSG — static with generateStaticParams (/blog/[slug])
- ƒ Dynamic — server-rendered on demand (/blog, /api/*, /feed.xml)
