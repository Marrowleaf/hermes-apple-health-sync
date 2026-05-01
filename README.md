# Fitness & Nutrition

> Track workouts, nutrition, and body metrics with comprehensive analytics and Obsidian integration.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/Marrowleaf/hermes-fitness-nutrition)

## Features

- Workout logging with exercise sets, reps, and weights
- Nutrition tracking with macro and calorie counting
- Body metrics tracking (weight, measurements, body fat)
- Progressive overload tracking and PR detection
- Weekly and monthly fitness reports with trend analysis
- Exercise library with muscle group categorization
- Meal logging with automatic macro calculation
- Integration with habits skill for streak tracking
- Obsidian-integrated data storage with PARA structure

## Installation

```bash
hermes skills install health/fitness-nutrition
```

Or manually clone into `~/.hermes/skills/health/fitness-nutrition/`.

## Usage

```
fitness log "bench press 3x10 @ 80kg"
fitness log "ran 5km in 25 minutes"
nutrition log "chicken breast 200g, rice 150g"
nutrition log "oats 80g with milk and banana"
fitness report week
nutrition summary --today
fitness pr bench-press
```

## Configuration

Store data in your Obsidian vault at `~/obsidian-vault/3-Resources/fitness/`. Configure exercise library, nutrition goals, and body metrics targets in `config.md`.

## Requirements

- Hermes Agent v0.12+
- Obsidian vault (for data storage)

## License

MIT