---
name: apple-health-sync
description: Import Apple Health export data (ZIP with XML/CSV), parse steps, weight, workouts, heart rate, and sleep, sync to Obsidian daily notes and the fitness-nutrition skill, generate weekly health dashboards, and handle deduplication.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  tags:
    - health
    - apple-health
    - data-import
    - obsidian
    - fitness
    - sleep
    - heart-rate
    - steps
  related_skills:
    - fitness-nutrition
    - routine-optimizer
    - habits
commands:
  - name: import
    description: Import an Apple Health export ZIP file
    args:
      - name: file
        type: string
        required: true
        description: Path to the Apple Health export ZIP file
      - name: start_date
        type: string
        default: ""
        description: Only import data from this date onward (YYYY-MM-DD)
      - name: end_date
        type: string
        default: ""
        description: Only import data up to this date (YYYY-MM-DD)
      - name: dry_run
        type: boolean
        default: false
        description: Preview what would be imported without writing anything
  - name: sync
    description: Sync imported data to Obsidian daily notes and fitness-nutrition skill
    args:
      - name: date
        type: string
        default: ""
        description: Specific date to sync (YYYY-MM-DD), or blank for all unsynced
      - name: target
        type: string
        default: all
        description: Sync target (all, obsidian, fitness-nutrition)
      - name: force
        type: boolean
        default: false
        description: Re-sync even already-synced data
  - name: dashboard
    description: Generate a weekly health dashboard
    args:
      - name: week
        type: string
        default: current
        description: Week to generate dashboard for (current, last, or ISO week)
      - name: format
        type: string
        default: full
        description: Dashboard format (brief, full)
  - name: status
    description: Show import status and deduplication info
    args:
      - name: detail
        type: string
        default: summary
        description: Level of detail (summary, full)
  - name: dedup
    description: Manually trigger deduplication check on imported data
    args:
      - name: metric
        type: string
        default: all
        description: Specific metric to dedup (all, steps, weight, hr, sleep, workouts)
---

# Apple Health Sync

Import your Apple Health data into your productivity system. This skill parses Apple Health exports, extracts key metrics, syncs them to your Obsidian daily notes and fitness-nutrition skill, and generates weekly health dashboards — all with smart deduplication to prevent double-importing.

## How It Works

1. **Import**: Reads the Apple Health export ZIP (from the Health app → Settings → Export All Health Data)
2. **Parse**: Extracts structured data from `export.xml` and CSV files for steps, weight, workouts, heart rate, and sleep
3. **Dedup**: Tracks what's already been imported using a state file, so re-importing or partial imports don't create duplicates
4. **Sync**: Writes parsed data into Obsidian daily note frontmatter and fitness-nutrition skill data files
5. **Dashboard**: Generates weekly summary dashboards with trends and correlations

## Commands Reference

### `import`

Import an Apple Health export ZIP file.

```bash
# Import full Apple Health export
hermes apple-health-sync import --file ~/Downloads/apple_health_export.zip

# Import only recent data
hermes apple-health-sync import --file ~/Downloads/apple_health_export.zip --start_date 2026-04-01

# Import a specific date range
hermes apple-health-sync import --file ~/Downloads/apple_health_export.zip \
  --start_date 2026-04-01 --end_date 2026-04-30

# Preview what would be imported without changing anything
hermes apple-health-sync import --file ~/Downloads/apple_health_export.zip --dry_run
```

**Import Process**:
1. Extracts ZIP to a temporary directory
2. Locates `export.xml` (or `export_cz.xml` for Czech locale — locale suffix varies)
3. Parses XML using fast streaming parser (handles files >1GB)
4. Also checks for CSV alternatives in newer iOS versions
5. Extracts the following record types:

| HKQuantityTypeIdentifier | Metric | Unit | Aggregation |
|---|---|---|---|
| `HKQuantityTypeIdentifierStepCount` | Steps | count | Daily sum |
| `HKQuantityTypeIdentifierBodyMass` | Weight | kg | Latest per day |
| `HKQuantityTypeIdentifierHeartRate` | Heart Rate | bpm | Avg/min/max per day |
| `HKQuantityTypeIdentifierActiveEnergyBurned` | Active Calories | kcal | Daily sum |
| `HKQuantityTypeIdentifierDistanceWalkingRunning` | Walk/Run Distance | km | Daily sum |
| `HKCategoryTypeIdentifierSleepAnalysis` | Sleep | hours | Time-in-bed per day |
| `HKQuantityTypeIdentifierAppleExerciseTime` | Exercise Time | min | Daily sum |
| `HKWorkoutTypeIdentifier*` | Workouts | various | Per workout |

6. Stores parsed data in `~/.hermes/data/apple-health/` as JSON
7. Updates the deduplication state file

### `sync`

Sync imported data to Obsidian and fitness-nutrition skill.

```bash
# Sync all unsynced data to both Obsidian and fitness-nutrition
hermes apple-health-sync sync

# Sync only today's data
hermes apple-health-sync sync --date 2026-05-01

# Sync only to Obsidian daily notes
hermes apple-health-sync sync --target obsidian

# Force re-sync everything (overwrites existing data)
hermes apple-health-sync sync --force
```

**Obsidian Daily Note Integration**:

After syncing, daily notes get frontmatter updates:

```yaml
---
date: 2026-05-01
steps: 8947
weight: 81.2
heart_rate_avg: 68
heart_rate_min: 52
heart_rate_max: 142
active_calories: 487
sleep_hours: 7.3
exercise_min: 45
---
```

And an appended health section:

```markdown
## Health Data (Apple Health)
- 🚶 **8,947 steps** (avg 6,200 for this day of week)
- ⚖️ **81.2 kg** (↑0.1 from yesterday)
- ❤️ **68 bpm avg** (range: 52-142)
- 🔥 **487 active cal** burned
- 😴 **7h 18min** sleep (in bed 7h 42min)
- 🏋️ **45 min** exercise time
```

**Fitness-Nutrition Sync**:

Pushes workout data and weight measurements to the fitness-nutrition skill's data files, enabling cross-referencing with nutrition data for:
- Workout intensity vs. calorie intake correlation
- Weight trend vs. calorie compliance
- Protein intake vs. exercise recovery metrics

### `dashboard`

Generate a weekly health dashboard.

```bash
# This week's dashboard
hermes apple-health-sync dashboard

# Last week
hermes apple-health-sync dashboard --week last

# Specific ISO week
hermes apple-health-sync dashboard --week 2026-W17

# Brief format
hermes apple-health-sync dashboard --format brief
```

**Dashboard Output** (saved to `~/obsidian-vault/3-Resources/health-dashboards/`):

```markdown
# Health Dashboard — Week 18, 2026 (Apr 28 - May 4)

## Overview
| Metric | Week Avg | vs Last Week | Target |
|--------|----------|-------------|--------|
| Steps | 8,234 | +532 ↑ | 10,000 |
| Weight | 81.2 kg | -0.3 ↓ | 80.0 kg |
| Sleep | 7h 12min | +18min ↑ | 7h 30min |
| Heart Rate | 67 bpm | -2 ↓ | <70 bpm |
| Active Cal | 492 | +28 ↑ | 500 |
| Exercise | 42 min/day | +7 ↑ | 45 min |

## Daily Breakdown
- **Mon 28**: 9,102 steps · 81.4 kg · 7h 05min sleep · 48min exercise
- **Tue 29**: 7,890 steps · 81.3 kg · 7h 22min sleep · 35min exercise
- **Wed 30**: 8,456 steps · 81.2 kg · 6h 55min sleep · 52min exercise
- **Thu 01**: 8,947 steps · 81.2 kg · 7h 18min sleep · 45min exercise
- **Fri 02**: 6,734 steps · 81.1 kg · 7h 30min sleep · 0min exercise (rest day)
- **Sat 03**: 9,871 steps · 81.2 kg · 7h 45min sleep · 65min exercise
- **Sun 04**: 6,538 steps · 81.0 kg · 8h 05min sleep · 48min exercise

## Weight Trend
📈 Downward trend this week (-0.3 kg)
- Projected to reach 80.0 kg target in ~4 weeks at current rate

## Workouts
- **Mon**: Upper body push (home) — 42min, 187 avg HR
- **Wed**: Lower body (home) — 52min, 172 avg HR
- **Thu**: Morning walk + stretch — 45min, 98 avg HR
- **Sat**: Full body circuit (home) — 65min, 158 avg HR
- **Sun**: Yoga + mobility — 48min, 82 avg HR

## Correlations (with [[routine-optimizer]] data)
- Steps >8,000 days → avg energy +1.3 points
- Sleep >7.5h nights → morning focus +0.9 points
- Rest day (Fri) → next-day performance +12%
```

### `status`

Check import status and deduplication state.

```bash
# Summary status
hermes apple-health-sync status

# Full detail including per-date sync status
hermes apple-health-sync status --detail full
```

**Output**:
```
Apple Health Import Status
─────────────────────────
Last import: 2026-05-01 08:30
Date range: 2025-01-15 to 2026-05-01
Total records: 487,231
Records imported: 487,231
Duplicates skipped: 12

Metrics available:
  Steps:           ✓ 476 days
  Weight:          ✓ 342 days
  Heart Rate:      ✓ 476 days
  Active Calories: ✓ 476 days
  Sleep:           ✓ 380 days
  Workouts:        ✓ 89 sessions
  Exercise Time:   ✓ 476 days

Sync status:
  Obsidian:        ✓ Up to date (476/476 days)
  Fitness-Nutrition: ✓ Up to date (342/342 days)
```

### `dedup`

Manually trigger deduplication.

```bash
# Check all metrics for duplicates
hermes apple-health-sync dedup

# Check only step data
hermes apple-health-sync dedup --metric steps
```

## Configuration

Add to your Hermes config (`~/.hermes/config.yaml`):

```yaml
apple-health-sync:
  # Paths
  obsidian_vault: ~/obsidian-vault
  daily_note_path: "0-Daily"
  dashboard_path: "3-Resources/health-dashboards"
  data_path: ~/.hermes/data/apple-health
  # Sync preferences
  sync_to_obsidian: true
  sync_to_fitness_nutrition: true
  # What to import
  import_metrics:
    - steps
    - weight
    - heart_rate
    - active_calories
    - sleep
    - exercise_time
    - workouts
  # User profile (used for dashboard targets)
  profile:
    weight_kg: 81.2
    target_weight_kg: 80.0
    target_steps: 10000
    target_sleep_hours: 7.5
    target_exercise_min: 45
    target_calories: 1930
    target_protein_g: 162
  # Heart rate zones (for workout analysis)
  hr_zones:
    resting_max: 65
    fat_burn_min: 66
    cardio_min: 110
    peak_min: 150
  # Deduplication
  dedup_strategy: skip  # skip or merge (skip = skip duplicates, merge = average values)
```

## Data Storage

### Parsed Data Files

Located at `~/.hermes/data/apple-health/`:

```
apple-health/
├── state.json              # Dedup and sync state tracking
├── steps/
│   └── 2026-05.json        # Monthly step data
├── weight/
│   └── 2026-05.json        # Monthly weight data
├── heart-rate/
│   └── 2026-05.json        # Monthly HR data
├── sleep/
│   └── 2026-05.json        # Monthly sleep data
├── workouts/
│   └── 2026-W18.json       # Weekly workout data
└── active-calories/
    └── 2026-05.json        # Monthly calorie data
```

### State Tracking (`state.json`)

```json
{
  "last_import": "2026-05-01T08:30:00+01:00",
  "import_hash": "a1b2c3d4",
  "date_range": {
    "start": "2025-01-15",
    "end": "2026-05-01"
  },
  "synced": {
    "obsidian_last_date": "2026-05-01",
    "fitness_nutrition_last_date": "2026-05-01"
  },
  "metrics": {
    "steps": { "days_imported": 476, "hash": "e5f6..." },
    "weight": { "days_imported": 342, "hash": "g7h8..." },
    "heart_rate": { "days_imported": 476, "hash": "i9j0..." },
    "sleep": { "days_imported": 380, "hash": "k1l2..." },
    "workouts": { "sessions_imported": 89, "hash": "m3n4..." }
  }
}
```

## Deduplication Strategy

### Hash-Based Dedup

Each record in the Apple Health export has a unique combination of:
- Type identifier (metric type)
- Start date
- End date
- Value

The skill computes a hash of these fields per record. On re-import:
1. Compute hash for each record
2. Check against existing hash set in `state.json`
3. If hash exists → skip (or merge, depending on config)
4. If hash is new → import and add to hash set

### Date-Range Dedup

For daily aggregated data (steps, calories):
- If `start_date` is provided, only process records on or after that date
- Existing data before `start_date` is left untouched
- Data within the range is checked record-by-record via hash dedup

### Merge Strategy

When `dedup_strategy: merge`:
- For daily totals (steps, calories): new values replace old values
- For weight: latest measurement wins
- For heart rate: averages are recalculated including new data points

When `dedup_strategy: skip` (default):
- Exact duplicates are skipped
- New records within previously-imported date ranges are added
- Conflicting records keep the original value

## Workout Parsing

Apple Health workouts are parsed from `HKWorkoutActivityType` identifiers:

| Apple Type | Parsed As | Data Extracted |
|---|---|---|
| `HKWorkoutActivityTypeTraditionalStrengthTraining` | Strength Training | Duration, HR, Calories |
| `HKWorkoutActivityTypeRunning` | Running | Duration, Distance, Pace, HR, Calories |
| `HKWorkoutActivityTypeWalking` | Walking | Duration, Distance, HR, Calories |
| `HKWorkoutActivityTypeYoga` | Yoga | Duration, HR, Calories |
| `HKWorkoutActivityTypeHighIntensityIntervalTraining` | HIIT | Duration, HR, Calories |
| `HKWorkoutActivityTypeFunctionalStrengthTraining` | Home Workout | Duration, HR, Calories |

Unknown workout types are imported as "Other" with all available data preserved.

## Pitfalls

1. **Large export files**: Apple Health exports can be 500MB+ for long-term users. The skill uses a streaming XML parser and processes records in batches to keep memory usage under 200MB. If your export is very large, the initial import may take 5-10 minutes.
2. **Locale-specific file names**: Newer iOS versions export CSV files alongside or instead of XML. The file name varies by locale (e.g., `export.xml`, `export_en.xml`, `export_cz.xml`). The skill checks all known variants.
3. **Timezone shifts**: Apple Health stores dates in UTC. The skill converts to your configured timezone (Europe/London) but daylight saving transitions may cause apparent hourly offsets in timestamps.
4. **Sleep tracking gaps**: Apple Watch sleep tracking can produce multiple sleep analysis records per night (deep sleep, light sleep, awake). The skill aggregates these into total time-in-bed and estimated sleep duration, but the numbers may differ slightly from the Health app's display.
5. **Weight measurement frequency**: If you only weigh in sporadically, weight days will have data and non-weight days won't. The dashboard shows the last measurement per day and interpolates for weekly averages when needed.
6. **Duplicate imports from partial exports**: If you export multiple times (e.g., monthly exports), the deduplication system handles this, but always use `--dry_run` first to verify the import scope.
7. **Data conflicts**: If you manually edit Obsidian frontmatter and then sync Apple Health data, the sync will overwrite manual edits to health metric fields. Use `--force` explicitly if this is intentional.
8. **Fitness-nutrition skill dependency**: If the fitness-nutrition skill isn't installed, syncing to that target is skipped gracefully. The `--target obsidian` option works standalone.

## Verification Steps

1. **Dry run import**: Run `hermes apple-health-sync import --file ~/Downloads/apple_health_export.zip --dry_run` — should show record counts and date range without writing anything.
2. **Full import**: Run the import without `--dry_run` — should create data files in `~/.hermes/data/apple-health/`.
3. **Status check**: Run `hermes apple-health-sync status` — should show imported record counts and date ranges.
4. **Obsidian sync**: Run `hermes apple-health-sync sync --target obsidian` — verify daily notes have been updated with health frontmatter. Check `~/obsidian-vault/0-Daily/` for a recent date.
5. **Dashboard generation**: Run `hermes apple-health-sync dashboard --format brief` — should create a file in `~/obsidian-vault/3-Resources/health-dashboards/`.
6. **Deduplication test**: Run the import command twice — second run should report "0 new records" or skip all duplicates.
7. **Date filtering**: Import with `--start_date 2026-04-01` and verify only data from April 2026 onward was processed.
8. **Fitness-nutrition sync**: If that skill is installed, run `hermes apple-health-sync sync --target fitness-nutrition` and verify workout/weight data appears in fitness-nutrition skill's data files.