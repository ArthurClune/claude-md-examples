# clune.org blog

## Architecture

This project is a Hugo personal blog, follwing the standard Hugo conventions
Posts go in `content/posts`

## Build Commands
- Serve locally with auto-rebuild: `hugo server -D`
- View docs at: http://localhost:1313

## Project Structure
- Documentation source files in `.md` (Markdown) and `.rst` (reStructuredText)
- Configuration in `conf.py`
- Built docs go to `_build/` directory (not committed to repo)

## Style Guidelines
- Use Markdown (`.md`) for new content when possible
- Follow existing heading structure (# for top-level, ## for sections)
- NEVER include a top level header on a content page. Use the title instead
- ALWAYS create new blog posts with `draft: true` in the header
- Include alt text for images
- Use relative links for internal references
- Keep line length reasonable for text readability
- Code blocks use triple backticks with language specified

## Deployment
- Automatically deployed via Cloudflare Pages from main branch
