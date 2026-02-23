# 📉 Content Decay Monitor

<div align="center">

![Version](https://img.shields.io/badge/version-1.0.0-1a4fff?style=flat-square)
![WordPress](https://img.shields.io/badge/WordPress-6.5%2B-21759b?style=flat-square&logo=wordpress&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-7.4%2B-777bb4?style=flat-square&logo=php&logoColor=white)
![License](https://img.shields.io/badge/license-GPL%20v2-green?style=flat-square)
![Tested up to](https://img.shields.io/badge/tested%20up%20to-WP%206.7-blue?style=flat-square)

**Detect and fix outdated content before it hurts your rankings.**

Scans every post on your site and produces a Content Freshness Score (0–100) based on publish age, modification date, stale year references, word count, and internal linking. Surfaces at-risk posts in a dashboard and the posts list — no external APIs, no page slowdown.

[🐛 Report a Bug](https://github.com/your-org/content-decay-monitor/issues) · [💡 Request a Feature](https://github.com/your-org/content-decay-monitor/issues)

</div>

---

## Table of Contents

- [Why Content Decay Monitor?](#-why-content-decay-monitor)
- [Features](#-features)
- [How the Score Works](#-how-the-score-works)
- [Installation](#-installation)
- [Usage](#-usage)
- [Risk Levels](#-risk-levels)
- [WP-Cron Automation](#-wp-cron-automation)
- [Security](#-security)
- [Technical Details](#-technical-details)
- [Customization](#-customization)
- [Changelog](#-changelog)
- [License](#-license)

---

## 🎯 Why Content Decay Monitor?

Content goes stale. A post published three years ago with no updates, thin word count, and year references like "2019" sends trust signals to search engines — and to readers. Most teams don't find out until rankings drop.

**Content Decay Monitor flags it automatically.**

| Without CDM | With CDM |
|---|---|
| No visibility into which posts are outdated | Every post has a live Freshness Score |
| Manual audits that happen too rarely | Daily automated scans in the background |
| Guessing which posts to refresh first | Dashboard sorted by lowest score |
| Checking posts one-by-one | Colour-coded badge on every row of the posts list |

---

## ✨ Features

- 🔢 **Content Freshness Score (0–100)** — weighted scoring across five decay signals
- 📅 **Publish & modified age detection** — configurable staleness thresholds in months
- 🗓️ **Stale year regex** — flags content containing year references from 2018–2023
- 📖 **Word count penalty** — penalises posts below a configurable minimum
- 🔗 **Internal link detection** — penalises posts with no links to other pages on the same domain
- 📊 **Admin dashboard** — sortable table of all scanned posts with score, risk level, and last scan time
- 🏷️ **Posts list column** — colour-coded Freshness badge on every row, sortable by score
- ⚡ **AJAX scan triggers** — run a full scan or re-scan a single post without page reload
- ⏰ **Daily WP-Cron job** — scans posts automatically in configurable batches of 20
- ⚙️ **Settings panel** — tune every threshold directly from the admin UI

---

## 📐 How the Score Works

Every post starts at **100**. Deductions are applied independently and the result is clamped to 0–100.

| Signal | Max deduction | Logic |
|---|---|---|
| Publish age over threshold | −30 pts | Linear ramp from threshold to 2× threshold |
| Modified age over threshold | −20 pts | Same linear ramp |
| Stale year found in content | −20 pts | Flat penalty (any match of 2018–2023) |
| Word count below minimum | −15 pts | Linear from minimum down to 0 words |
| Zero internal links | −15 pts | Flat penalty |

**Example:** A post published 18 months ago (threshold: 12), last modified 14 months ago, containing "In 2021…", with 180 words and no internal links on a 300-word minimum site would score roughly **20 / 100 → High Risk**.

---

## 📦 Installation

### Via WordPress Admin (recommended)

1. Download **[content-decay-monitor-v1.0.0.zip](https://github.com/your-org/content-decay-monitor/releases/latest)**
2. In your WordPress admin go to **Plugins → Add New → Upload Plugin**
3. Select the `.zip` file and click **Install Now**
4. Click **Activate Plugin**

The **Content Decay** entry appears immediately under **Tools** in your admin sidebar.

### Manual Installation

```bash
cd wp-content/plugins
unzip content-decay-monitor-v1.0.0.zip
```

Then activate via **Plugins → Installed Plugins**.

### Via WP-CLI

```bash
wp plugin install content-decay-monitor.zip --activate
```

> **No database migrations required.** Scores are stored as standard post meta (`_cdm_score`, `_cdm_last_scan`) and removed automatically on uninstall.

---

## 📖 Usage

### Running Your First Scan

1. Go to **Tools → Content Decay** in the admin sidebar
2. Click the **Run Scan** button
3. Wait for the confirmation — the table populates with scores immediately

### Reading the Dashboard

| Column | Description |
|---|---|
| Post Title | Linked to the post editor |
| Score | 0–100 Freshness badge (colour-coded) |
| Risk Level | Low / Medium / High derived from score |
| Last Scan | Date and time the score was last calculated |
| Actions | **Re-scan** a single post · **View** on the front end |

### Scanning from the Posts List

The **Freshness** column appears on all Posts list screens. The colour-coded badge shows the current score at a glance. Click column header to sort by freshness score ascending.

### Adjusting Thresholds

Go to **Tools → Content Decay → Settings** to configure:

| Setting | Default | Description |
|---|---|---|
| Publish age threshold | 12 months | Posts older than this begin losing points |
| Modified age threshold | 12 months | Posts not updated within this window lose points |
| Minimum word count | 300 words | Posts below this count are penalised |
| Batch size | 20 posts | Posts per WP-Cron batch |

---

## 🎨 Risk Levels

| Badge | Score range | Label | Recommended action |
|---|---|---|---|
| 🟢 Green | 80–100 | Low | Content is fresh — monitor quarterly |
| 🟠 Orange | 50–79 | Medium | Review and consider updating within 30 days |
| 🔴 Red | 0–49 | High | Prioritise for immediate refresh or consolidation |

---

## ⏰ WP-Cron Automation

A daily cron event (`cdm_daily_scan`) is registered on plugin activation. It scans posts in configurable batches and resumes from where it left off across multiple fires — safe for large sites.

- **Batch size:** configurable (default 20 posts per cron fire)
- **Offset persistence:** stored in a transient; resets when the full cycle completes
- **Logging:** writes progress to the PHP error log when `WP_DEBUG_LOG` is enabled

No manual scheduling required. The cron is unscheduled cleanly on plugin deactivation.

---

## 🔒 Security

| Measure | Implementation |
|---|---|
| Capability check | `current_user_can('manage_options')` on every entry point |
| Nonce verification | `check_ajax_referer()` on all three AJAX endpoints |
| Data sanitization | `sanitize_key()`, `absint()`, `array_map` throughout |
| Output escaping | `esc_html()`, `esc_attr()`, `esc_url()`, `wp_kses_post()` on all output |
| Front-end isolation | Zero output on the public site — no hooks, no assets |
| SQL safety | All queries use `$wpdb->prepare()` with explicit type placeholders |

Score data is stored under underscore-prefixed meta keys (`_cdm_*`), hiding them from the standard Custom Fields UI.

---

## 🛠️ Technical Details

### Plugin Architecture

```
content-decay-monitor/
├── content-decay-monitor.php        Bootstrap, constants, activation/deactivation
│
├── includes/
│   ├── class-cdm-helpers.php        Stateless utilities (word count, link count, regex, date math)
│   ├── class-cdm-score.php          Pure scoring logic — weighted deductions, no I/O
│   ├── class-cdm-scanner.php        Batch post fetching, signal gathering, meta persistence
│   ├── class-cdm-admin.php          Tools submenu, dashboard table, settings form, AJAX handlers
│   ├── class-cdm-columns.php        Posts list column, badge rendering, sort query handler
│   └── class-cdm-cron.php           WP-Cron event, transient-based offset, batch resumption
│
└── assets/
    └── admin.css                    Badge colours, table layout, settings panel (~120 lines)
```

### Post Meta Keys

| Key | Type | Description |
|---|---|---|
| `_cdm_score` | `int` | Freshness score 0–100 |
| `_cdm_last_scan` | `datetime` | UTC timestamp of the most recent scan |

### Requirements

| | Minimum | Recommended |
|---|---|---|
| WordPress | 6.5 | 6.7+ |
| PHP | 7.4 | 8.1+ |
| MySQL / MariaDB | 5.6 | 8.0+ |

---

## 🔧 Customization

### Add Support for Custom Post Types

By default CDM scans `post`. Extend it with a filter in your theme or a mu-plugin:

```php
add_filter( 'cdm_post_types', function( $types ) {
    $types[] = 'product';   // WooCommerce products
    $types[] = 'event';     // Custom events CPT
    return $types;
} );
```

### Override the Stale Year Range

```php
add_filter( 'cdm_stale_year_pattern', function() {
    return '/\b(201[5-9]|202[0-4])\b/'; // Widen the range
} );
```

### Hook Into Scan Events

```php
// Fires after a single post is scanned
add_action( 'cdm_post_scanned', function( $post_id, $score, $signals ) {
    // e.g. log to a custom table or push to an analytics service
}, 10, 3 );

// Fires after a full batch completes
add_action( 'cdm_batch_complete', function( $scanned_count, $offset ) {
    // e.g. send a Slack summary
}, 10, 2 );
```

---

## 📋 Changelog

### v1.0.0 — Initial release

- Content Freshness Score (0–100) with five weighted decay signals
- Configurable thresholds for age, modification date, word count, and batch size
- Admin dashboard under Tools → Content Decay with sortable results table
- Colour-coded Freshness column in the Posts list (sortable)
- AJAX full-scan and single-post re-scan buttons
- Daily WP-Cron job with transient-based batch offset resumption
- Settings panel for all thresholds
- Full nonce, capability, sanitization and escaping coverage
- Zero front-end output

---

## 📄 License

Licensed under the **GNU General Public License v2.0 or later**.

```
This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.
```

Full license text: [https://www.gnu.org/licenses/gpl-2.0.html](https://www.gnu.org/licenses/gpl-2.0.html)
