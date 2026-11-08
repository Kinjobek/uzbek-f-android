mkdir -p .kiro/specs/uzbek-f-keyboard .kiro/steering .kiro/hooks

cat << 'EOF' > .kiro/steering/product-and-conventions.md
# Project Steering: Özbekça termılma v1.05 (ALHARITM)

## Overview & Copyright
- **Application Title:** Özbekça termılma v1.05
- **Organization / Owner:** ALHARITM (alharitm.com) — Author: Sarvarbek Nazarov (@Kinjobek)
- **Copyright:** © 2026 ALHARITM. All rights reserved.

## Linguistic & Ergonomic Rules
- Target script: Updated Uzbek Latin alphabet (`ç`, `ş`, `ğ`, `ö`, `ü`, `ı`, `a-z`).
- Layout philosophy: Ergonomic adaptation inspired by the Turkish F-keyboard based on Python statistical letter-frequency analysis of Uzbek text corpora.
- High-frequency vowels reside on the home row (left hand) and high-frequency consonants on the home row (right hand) to maximize alternating-hand typing efficiency.

## Android UI & Technical Guardrails
- Always handle Android `WindowInsets` (`WindowInsetsCompat.Type.navigationBars()`) properly so the system bottom gesture bar or language-switcher menu never overlaps the bottom row or obscures the period (`.`) key.
- Keep cross-platform layout parity across Android (`uzbek-f-android`), Web (`kinjobek.github.io/uz/`), and macOS Ukelele (`.keylayout`).
EOF

cat << 'EOF' > .kiro/specs/uzbek-f-keyboard/requirements.md
# Requirements Specification: Özbekça termılma v1.05

## 1. Introduction
An ergonomic mobile keyboard (`InputMethodService`) for Android tailored for the updated Uzbek Latin alphabet, engineered by ALHARITM using statistical character-frequency optimization.

## 2. User Stories & Acceptance Criteria

### US-1: Ergonomic F-Layout Character Mapping
- **WHEN** the user activates "Özbekça termılma v1.05"
- **THE SYSTEM SHALL** render the 3-row/4-row ergonomic Uzbek F-layout featuring native single-tap access to `ç`, `ş`, `ğ`, `ö`, `ü`, and `ı` without requiring long-press popups for core characters.

### US-2: Bottom Navigation Bar Inset Protection
- **WHEN** the keyboard is displayed on modern Android devices with gesture navigation or a bottom system language-switch button
- **THE SYSTEM SHALL** apply dynamic bottom padding via `WindowInsets` so the bottom row and the period (`.`) key remain 100% visible and tappable without accidental system menu triggers.

### US-3: Predictive Morphology & Shift State Invariants (PBT)
- **WHEN** the user types agglutinative Uzbek root words or toggles Shift/Caps Lock
- **THE SYSTEM SHALL** accurately transition case states (including `i` <-> `İ` and `ı` <-> `I`) and provide localized word completions.
EOF

cat << 'EOF' > .kiro/specs/uzbek-f-keyboard/design.md
# Architecture & Design: Özbekça termılma v1.05

## 1. System Architecture
1. **Corpus & Frequency Engine (Python):** Analyzes Uzbek text corpora to compute monogram/bigram probabilities and finger load distribution.
2. **Android IME Core (`uzbek-f-android`):** Custom `InputMethodService` handling key events, layout inflation, and `InputConnection` text commits.
3. **Adaptive Inset Manager:** Listens to `OnApplyWindowInsetsListener` and adjusts bottom container padding to prevent navigation overlay defects.
4. **CI/CD Pipeline:** GitHub Actions workflow compiling and packaging release/debug APK artifacts on every push.

## 2. Property-Based Testing (PBT) Invariants
- **Property 1 (Case Bijection):** For every key $k$ in the Uzbek F-layout, `lowercase(uppercase(k)) == k`, specifically preserving dotted `i/İ` and dotless `ı/I` distinctions.
- **Property 2 (Non-Zero Hitbox Above Insets):** For any screen height $H$ and system bottom inset $I_b$, the bottom coordinate of the period (`.`) key $Y_{period}$ must satisfy $Y_{period} \le H - I_b$.
EOF

cat << 'EOF' > .kiro/specs/uzbek-f-keyboard/tasks.md
# Implementation Tasks

- [x] 1. Run Python statistical letter-frequency analysis on Uzbek Latin corpora and finalize F-layout key matrix.
- [x] 2. Implement Android XML keyboard definitions and `InputMethodService` core for "Özbekça termılma v1.05".
- [x] 3. Add ALHARITM copyright protection and branding headers across application settings and UI.
- [x] 4. Fix bottom navigation bar visual overlay defect obscuring the period (`.`) key by integrating dynamic `WindowInsets` padding.
- [x] 5. Validate character case transitions (`i/İ`, `ı/I`, `ç/Ç`, `ş/Ş`, `ğ/Ğ`, `ö/Ö`, `ü/Ü`) using Property-Based Testing (PBT).
- [x] 6. Configure GitHub Actions CI/CD workflow for automated APK artifact generation.
EOF

cat << 'EOF' > .kiro/hooks/validate-keyboard-layout.json
{
  "name": "Validate Uzbek F-Keyboard XML & CI Readiness",
  "description": "Automatically verifies Unicode character integrity, bottom inset padding rules, and Gradle build readiness whenever keyboard XML or Kotlin/Java IME files are saved.",
  "trigger": {
    "type": "onFileSave",
    "filePatterns": [
      "app/src/main/res/xml/*.xml",
      "app/src/main/java/**/*.kt",
      "app/src/main/java/**/*.java"
    ]
  },
  "action": {
    "type": "agentPrompt",
    "prompt": "Verify that all updated Uzbek Latin characters (ç, ş, ğ, ö, ü, ı, İ) have valid keycodes, ensure bottom WindowInsets padding is preserved so the period key is not obscured, and confirm compatibility with the GitHub Actions APK build workflow."
  }
}
EOF

git add .kiro
git commit -m "Add Kiro specs, steering, and hooks for Kiroween 2026"
git push
