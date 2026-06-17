# Changelog

A living document tracking all work completed on this project, organized by commit.

---

<!-- New entries are added at the top, below this line -->

## 2026-06-17 — Add product catalog feature spec

- Add requirements document for product-catalog feature with 9 detailed requirements
- Covers API fetching, product display, category filtering, price range filtering, sorting, search with debounce, result count, combined filter logic, and responsive layout
- Add spec config (.kiro/specs/product-catalog/.config.kiro) for requirements-first workflow

## 2026-06-17 — Add merge-to-main hook and React best practices steering

- Add user-triggered hook to switch to main, merge feature branch, and update changelog
- Add steering file with modern React conventions including feature-based folder structure, hooks patterns, Redux Toolkit standards, TypeScript conventions, and common pitfall avoidance

## 2026-06-17 — Project scaffolding and automation setup

- Add agent hook for auto commit, changelog update, and push on agent stop
- Add agent hook for creating numbered feature branches (user-triggered)
- Add CHANGELOG.md as a living document to track work per commit
- Add steering file with documentation and changelog best practices
- Initialize project with readme.md
