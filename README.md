# harryjohn.org Website Migration

Project to recover my old blog at harryjohn.org using the archive from Wayback Machine, and republish its content to the original URLs.

## Project overview

This project involves migrating an old WordPress website (harryjohn.org) to something modern, probably a static site generator like Hugo. I'll be using Cursor AI to assist, but with [rules in place](.cursor/rules/learning-guidelines.mdc) to ensure the AI assistance prioritises teaching me along the way.

## Repository Strategy

**Current approach**: All migration work (WordPress backup, converted content, Hugo site) is kept in this single repository until the site is ready for publishing.

**Future approach**: Once the Hugo site is complete and ready for deployment, we'll create a separate repository containing only the Hugo site files for deployment to Cloudflare Pages.

## Current state

### Source materials
- **Original WordPress Site**: `harryjohn-org-wayback/` - Downloaded backup of the original harryjohn.org website
- **Converted Content**: `content/posts/` - Markdown files converted from WordPress using pandoc
- **Hugo Site**: `hugo/` - New Hugo site with PaperMod theme and working content

## Requirements

### Minimum viable product requirements (Must Have)
1. **Readable articles**: Posts display properly with good typography
2. **Images**: Images from original posts are included and display correctly
3. **Basic navigation**: Home page and individual post pages work
4. **Clean, original URLs**: Proper URL structure for posts, and the original URLs are restored (with a redirect if neccessary)

### Nice-to-have
- Tags and categories ✅
- Comments system
- SEO optimization
- Search functionality
- Dark mode

## Implementation plan

### Phase 0: Planning ✅
1. ✅ Define requirements for hosting
2. ✅ Compare options 
Done - see [docs/planning.md](docs/planning.md)

### Phase 1: Hugo site setup ✅
1. ✅ Understand hugo basics
2. ✅ Configure basic site
3. ✅ Set basic theme (PaperMod)
4. ✅ Create some basic content
5. ✅ Run the site locally

### Phase 2: Content migration ✅
1. ✅ Add front matter to hugo content pages
2. ✅ Cleanup and process markdown conversion
3. ✅ Fix assets/images/URLs
4. ✅ Import remaining WordPress posts

**Progress**: ✅ Successfully imported all 6 WordPress posts with proper front matter, clean markdown, and working images. Content migration complete!

### Phase 3: Publishing 🔄
1. 🔄 Create separate Hugo-only repository for deployment
2. ⏳ Connect the Hugo repo to Cloudflare
3. ⏳ Publish the site to harryjohn.dev
4. ⏳ Setup redirects and forwarders for old links at harryjohn.org
