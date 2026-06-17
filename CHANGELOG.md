# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2026-06-17

### Added
- **Modular Architecture**: Split expertise into `backend.md`, `volto.md`, and `classic.md` for zero context pollution and token efficiency.
- **Autonomous Router**: `SKILL.md` now uses active discovery (`ls -R`) to automatically detect project types and load relevant modules.
- **TypeScript-First Volto**: Modernized all Volto examples to TypeScript (.tsx) with interface patterns.
- **Pytest Standard**: Adopted `pytest-plone` as the primary backend testing standard.
- **Core Contributor Patterns**:
  - Deep patterns for `@@plone`, `@@plone_portal_state`, and `@@plone_context_state`.
  - Modern image rendering via `@@images` (`srcset`, `picture`).
  - Manual i18n/l10n via `plone.base.i18nl10n`.
- **Advanced GenericSetup**: Patterns for `catalog.xml` and mandatory `plonecli` Upgrade Steps.
- **Classic UI Support**: Integrated Bootstrap 5, Diazo, Mockup Patterns, and Mosaic Layouts.

### Changed
- Refactored `SKILL.md` into a lightweight project dispatcher.
- Purged legacy (pre-Plone 6) patterns.

## [1.1.0] - 2026-03-26

### Changed
- Integrated official Plone LLM documentation (`llms.txt`) as the mandatory knowledge retrieval protocol.
- Replaced hardcoded reference lists with dynamic `webfetch` instructions.
