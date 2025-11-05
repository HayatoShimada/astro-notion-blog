# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

astro-notion-blog is a static blog generator that uses Notion as a CMS and Astro as the static site generator. Content is fetched from a Notion database and rendered as static pages with lightning-fast performance.

## Environment Setup

Required environment variables (set in `.env` for local development or in deployment platform):
- `NOTION_API_SECRET` - Your Notion integration token
- `DATABASE_ID` - The ID of your Notion database (from the database URL)

Optional environment variables:
- `CUSTOM_DOMAIN` - Your custom domain (e.g., example.com)
- `BASE_PATH` - Sub-directory path for deployment (e.g., /blog)
- `PUBLIC_GA_TRACKING_ID` - Google Analytics tracking ID
- `REQUEST_TIMEOUT_MS` - Timeout for API requests (default: 10000)
- `ENABLE_LIGHTBOX` - Enable image lightbox functionality
- `CACHE_CONCURRENCY` - Number of concurrent cache fetches (default: 1)

## Development Commands

```bash
# Install dependencies
npm install

# Start development server (runs on http://localhost:4321)
npm run dev

# Build for production
npm run build

# Build with cached Notion content (faster subsequent builds)
npm run build:cached

# Preview production build
npm run preview

# Lint code
npm run lint

# Format code with Prettier
npm run format

# Fetch Notion content and cache locally
npm run cache:fetch

# Clear cache and Nx build cache
npm run cache:purge
```

## Architecture

### Core Data Flow

1. **Notion API Integration** (`src/lib/notion/client.ts`)
   - Fetches posts, pages, and blocks from Notion API
   - Handles image downloads and caching
   - Transforms Notion API responses into internal data structures
   - Uses retry logic for resilience

2. **Caching Strategy**
   - Blog content is cached in `tmp/` directory using Nx caching
   - `scripts/blog-contents-cache.cjs` fetches all published posts in parallel
   - Cache is keyed by page ID and last edited time
   - Run `npm run cache:purge` to clear all caches

3. **Static Site Generation**
   - Astro generates static pages at build time
   - Dynamic routes use `getStaticPaths()` to pre-render all pages
   - Main routes: `/posts/[slug]`, `/posts/page/[page]`, `/posts/tag/[tag]/[page]`

### Directory Structure

- `src/lib/notion/` - Notion API client and response transformers
- `src/lib/blog-helpers.ts` - Helper functions for blog operations (extracting blocks, generating links, etc.)
- `src/components/notion-blocks/` - Astro components for rendering each Notion block type
- `src/integrations/` - Custom Astro integrations that run during build
- `src/pages/` - Astro page routes (file-based routing)
- `scripts/` - Build and caching scripts
- `tmp/` - Cached Notion content (gitignored)

### Custom Astro Integrations

The project uses custom integrations defined in `astro.config.mjs`:
- `CoverImageDownloader` - Downloads and caches cover images
- `CustomIconDownloader` - Downloads and caches custom icons
- `FeaturedImageDownloader` - Downloads and caches featured images
- `PublicNotionCopier` - Copies Notion assets to public directory

### Notion Block Rendering

Each Notion block type has a corresponding Astro component in `src/components/notion-blocks/`:
- Structural: `Paragraph`, `Heading1-3`, `BulletedListItems`, `NumberedListItems`, `ColumnList`, `Table`
- Media: `Image`, `Video`, `Embed`, `Bookmark`
- Special: `Code`, `Equation`, `Callout`, `ToDo`, `SyncedBlock`
- Rich text annotations handled by components in `src/components/notion-blocks/annotations/`

The main `NotionBlocks.astro` component recursively renders blocks and their children.

## Key Configuration

- `src/server-constants.ts` - Central configuration file for environment variables and constants
- `astro.config.mjs` - Astro configuration with site URL detection for Vercel/Cloudflare/local
- `tailwind.config.mjs` - Tailwind CSS configuration

## Notion Database Schema

Expected properties in Notion database:
- `Published` (checkbox) - Whether the post is published
- `Date` (date) - Publication date
- `Slug` (rich text) - URL slug for the post
- `Tags` (multi-select) - Post tags
- Additional properties are defined in `src/lib/interfaces.ts`

## Testing Changes

1. Set environment variables for local development
2. Run `npm run dev` to start development server
3. Test locally at http://localhost:4321
4. Run `npm run build` to ensure production build works
5. Use `npm run preview` to preview production build

## Deployment

The project is designed for deployment on:
- **Cloudflare Pages** (primary target)
- Vercel (also supported)

Build settings:
- Framework preset: Astro
- Build command: `npm run build` or `npm run build:cached`
- Output directory: `dist/`
- Node version: v18.16.0 or higher

Note: Manual deployment is required after publishing new posts in Notion. Consider using scheduled deployments via GitHub Actions or similar CI tools.
