# Seyong Kim's Personal Website

This repository hosts the source code for the personal academic homepage of **Seyong Kim** (Ph.D. Student at Yonsei University), built as a lightweight Jekyll site compatible with GitHub Pages.

Website URL: [https://kim-seyong.github.io](https://kim-seyong.github.io)

---

## Structure

- **Homepage & Bio**: `index.html` (Main profile, research interests, and publication list)
- **Site Configurations**: `_config.yml` (Metadata, social links, base URLs)
- **Layouts**: `_layouts/`
- **Shared Components**: `_includes/` (Header, navigation, footer)
- **Stylesheets**: `assets/css/` (or `css/`)
- **Images**: `Seyong.jpg` and `assets/img/` (Profile photo and post assets)

---

## Updating Content

### 1. Publications & Research
Open `index.html` and update the `<section id="publications">` or `<section id="experience">` blocks directly with new papers, conference proceedings, or research updates.

### 2. Basic Information
Edit `_config.yml` to update personal details, Google Scholar links, and contact information.

---

## Local Preview (Optional)

If you have Ruby and Jekyll installed locally:

```bash
bundle exec jekyll serve
