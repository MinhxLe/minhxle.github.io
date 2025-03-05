# CLAUDE.md - Jekyll GitHub Pages Site Guide

## Build/Run Commands
- `bundle install` - Install dependencies
- `bundle exec jekyll serve` - Run local development server
- `bundle exec jekyll serve --livereload` - Run with live reloading
- `bundle exec jekyll build` - Build site for production
- `bundle exec jekyll doctor` - Check for configuration issues

## Code Style Guidelines
- **Posts**: Create in `_posts/` directory with format `YYYY-MM-DD-title.md`
- **Front Matter**: Include proper YAML front matter in all posts with `layout`, `title`, `date`, and optional `author`
- **Markdown**: Use GitHub-flavored Markdown
- **Math**: Use MathJax for equations (see `_includes/mathjax.html`)
- **Formatting**: Maintain consistent heading structure (H1 for title, H2 for sections)
- **File Names**: Use kebab-case for all files
- **Images**: Store in `assets/images/` referenced with relative paths
- **Link Style**: Prefer relative links within the site

## Repository Structure
- `_posts/` - Blog post markdown files
- `_layouts/` - HTML templates
- `_includes/` - Reusable components
- `assets/` - Static files (images, css, js)