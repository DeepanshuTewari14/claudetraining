# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

This repository contains a single deliverable: `index.html`, a self-contained mockup of an internal IT Support Service Desk for a fictional "UOB Bank" training/demo. There is no build system, package manager, or test suite — the entire site (markup, CSS, and JavaScript) lives in one HTML file.

## Commands

There is no build, lint, or test tooling. To view the site, open `index.html` directly in a browser (double-click, or `open index.html` on macOS) — no server or dependencies required.

## Architecture

Everything is inline in `index.html`:
- All styles are in a single `<style>` block in `<head>`, using CSS custom properties (`:root`) for a bank-ledger/queue-ticket palette (ink navy `#0E2136`, brass `#C89550`/`#8F6420`, verdigris `#3F6B62`) and layout tokens (1120px max-width container). The "ticket stub" (perforated card) is the page's signature visual motif, used for the hero status card, the post-submission confirmation, and the WhatsApp widget panel.
- All behavior is in a single `<script>` block at the end of `<body>`, split into four self-contained IIFEs with no shared global state between them:
  1. Nav/hamburger toggle for the mobile menu.
  2. Ticket form validation, submission, and the in-memory (non-persisted) `submittedTickets` array — no `localStorage`, `sessionStorage`, or network calls are used anywhere on the page.
  3. FAQ accordion rendering (from an inline JS data array, via DOM APIs rather than `innerHTML`) and live search filtering.
  4. WhatsApp widget: floating action button toggling a suggested-queries panel (outbound `wa.me` links only — no JS-driven network calls).
- The page is organized into five sections marked with `<!-- ===== ... ===== -->` HTML comments (Header/Nav/Hero, Ticket Form, FAQ, Footer, WhatsApp Widget); matching comments mark the corresponding CSS and JS blocks. Keep this comment convention when adding new sections so the single file stays navigable.

## Constraints to preserve

- Keep the file dependency-free: no external scripts/stylesheets/fonts, no build step. It must keep working when opened directly via `file://`.
- The footer disclaimer ("Training demo — not affiliated with or endorsed by United Overseas Bank Limited") must remain visible — this is a training mockup, not a real bank property.
- Don't add real credential collection (passwords, OTPs, card numbers). The description field intentionally detects and warns on credential-like text via a regex in the form-validation IIFE.
- Don't introduce persistence or network calls (storage APIs, `fetch`, form `action` submission) — submitted ticket data must stay in the in-memory array only.
