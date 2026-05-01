# Apple Health Sync

> Import and sync Apple Health data from iPhone exports into Obsidian with trend analysis and health dashboards.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/Marrowleaf/hermes-apple-health-sync)

## Features

- Import Apple Health export ZIP files (export.zip)
- Parse step count, heart rate, sleep analysis, workouts, and more
- Daily, weekly, and monthly health dashboards in Obsidian
- Trend analysis across health metrics over time
- XML and CSV parsing support for Apple Health exports
- Automatic data normalization and deduplication
- Integration with fitness-nutrition and habits skills
- Health metric correlations (e.g., steps vs. heart rate)
- Configurable metric types and date ranges

## Installation

```bash
hermes skills install health/apple-health-sync
```

Or manually clone into `~/.hermes/skills/health/apple-health-sync/`.

## Usage

```
apple-health import /path/to/export.zip
apple-health summary --week
apple-health dashboard --month
apple-health trend steps --days 90
apple-health compare --metric heart-rate --period1 jan --period2 feb
```

## Configuration

- `APPLE_HEALTH_EXPORT_PATH`: Default path for Apple Health export ZIP
- Store parsed data in Obsidian at `~/obsidian-vault/3-Resources/health/`
- Configure which metrics to track and dashboard layout in `config.md`

## Requirements

- Hermes Agent v0.12+
- Python 3.8+ (for XML/CSV parsing)
- Obsidian vault (for data storage and dashboards)
- Apple Health export file from iPhone

## License

MIT