# Drission-Page Skills

A curated skill set that equips Claude Code with precise knowledge of the [DrissionPage](https://github.com/g1879/DrissionPage) library — a Python browser automation and HTTP client that unifies Selenium-style control with requests-style speed.

## Structure

```
Drission-Page/
├── README.md                        This file (English)
├── README_CN.md                     Chinese readme
├── SKILL.md                         Router (entry point)
├── quickstart/SKILL.md              Quick start
├── find-elements/SKILL.md           Element location
├── browser-ops/SKILL.md             Browser operations
├── element-ops/SKILL.md             Element interaction
├── network/SKILL.md                 Network monitoring
├── advanced/SKILL.md                Advanced features
├── update/SKILL.md                  Version check & upgrade
└── evals/                           Evaluation cases
```

## Skill Overview

| Skill | Lines | Triggers When |
|-------|:-----:|---------------|
| `Drission-Page` | 148 | Router — compound task planning, sub-skill dispatch |
| `Drission-Page:quickstart` | 176 | Installation, choosing page objects (ChromiumPage / SessionPage / WebPage) |
| `Drission-Page:find-elements` | 284 | Locator syntax (`#id` / `.class` / `@attr=value` / `text:` / `css:` / `xpath:`) |
| `Drission-Page:browser-ops` | 292 | Tab management, navigation, screenshots, iframes, dialogs, cookies, JS execution |
| `Drission-Page:element-ops` | 295 | Click, input, select, file upload, drag, keyboard, attribute/text extraction |
| `Drission-Page:network` | 323 | Network packet listening (`listen` API), HTTP requests, downloads, session cookies |
| `Drission-Page:advanced` | 395 | Smart waiting, anti-detection, Shadow DOM, ChromiumOptions, headless mode, concurrency |
| `Drission-Page:update` | 203 | Version check, PyPI comparison, upgrade, breaking changes, environment conflicts |
| **Total** | **2116** | |

> Verified against DrissionPage **4.1.1.4** API.

## Installation

### Option 1: Clone (recommended)

```bash
git clone https://github.com/bg1cnw/DrissionPage-Skill.git ~/.claude/skills/Drission-Page
```

Restart Claude Code. Skills are auto-discovered — just mention DrissionPage in conversation.

### Option 2: Manual

Download the repository and place all files under `~/.claude/skills/Drission-Page/`.

### Verify

Type `/Drission-Page` in Claude Code. If you see the routing prompt, installation succeeded.

### Library dependency

```bash
pip install DrissionPage>=4.1.1.4
```

## Usage

When you describe a task involving DrissionPage, Claude automatically invokes the right skill:

```
"Write a crawler that logs in and scrapes data"  → Drission-Page (router)
"ele() can't find my element"                     → Drission-Page:find-elements
"How to manage multiple tabs"                     → Drission-Page:browser-ops
"Is there a new DrissionPage version"             → Drission-Page:update
```

Or invoke manually with slash commands:

```
/Drission-Page
/Drission-Page:quickstart
/Drission-Page:find-elements
/Drission-Page:browser-ops
/Drission-Page:element-ops
/Drission-Page:network
/Drission-Page:advanced
/Drission-Page:update
```

## Version Coverage

- Installed: **4.1.1.4**
- Latest PyPI: **4.1.1.4** (as of 2026-05)
- New in 4.1.1.4: **DrissionGet** (multi-threaded downloads), **DrissionRecord** (data recording)

## Evaluation

| Scenario | With Skill | Without Skill | Gain |
|----------|:----------:|:-------------:|:----:|
| Quick start + API correctness | 100% | 40% | +60pp |
| Locator syntax completeness | 100% | 40% | +60pp |
| Ajax monitoring (`listen` API) | 100% | 25% | +75pp |
| **Average** | **100%** | **35%** | **+65pp** |

Without skills, Claude tends to emit Selenium-style APIs (`find_element(By.ID, ...)`, `send_keys()`) that don't exist in DrissionPage.

## Links

- DrissionPage docs: https://DrissionPage.cn
- DrissionPage repo: https://github.com/g1879/DrissionPage
- [中文 README](README_CN.md)
