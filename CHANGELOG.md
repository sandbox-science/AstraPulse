# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).  
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- Improved accessibility semantics by introducing proper `role="timer"` usage for the countdown display.
- Added two new presets (5 and 30 minutes).
- Set custom timer by pressing the `Enter` key on your keyboard.

### Changed
- Hardened timer lifecycle management to prevent duplicate `requestAnimationFrame` scheduling.
- Made timer start/stop operations idempotent and state-safe.
- Updated countdown formatting to use floor-based second calculations so the display reaches `00:00` during the final second.
- Improved internal timer control flow for clarity and maintainability.
- Ensured timer cleanup occurs reliably on component unmount.
- Center the presets button.

### Fixed
- Fixed potential background animation frame leaks when stopping or unmounting the timer.
- Prevented accidental timer rescheduling after completion.
- Eliminated edge cases that could leave the timer in an inconsistent running state.

---

## [0.1.0] – 2025-12-22

### Added
- Initialized AstraPulse with Tauri and Svelte
- Added application icons in multiple resolutions
- Included demo content and demo assets
- Added LICENSE file
- Added license information to the About section

### Changed
- Refined project description
- Improved README formatting and centered the AstraPulse logo
- Adjusted UI presentation (removed timer meta display, refined gradient opacity)
- Updated and aligned production documentation instructions
- Updated documentation to reflect frontend migration from Leptos to Svelte

### Fixed
- Corrected icon paths in Tauri configuration
- Fixed README errors in production documentation instructions

### Merged
- Initial public release branch into main
