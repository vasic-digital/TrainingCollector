# Architecture -- TrainingCollector

## Purpose

Collects screenshot + action pairs during autonomous QA sessions for future vision model fine-tuning. Exports training data in JSONL format suitable for supervised learning pipelines. No ML dependencies -- this module collects data only.

## Structure

```
pkg/
  training/   TrainingCollector with sequential file naming, JSONL export, and statistics
```

## Key Components

- **`training.TrainingCollector`** -- Core type: Record, Export, Stats, Len, Pairs. Thread-safe via mutex
- **`training.TrainingPair`** -- ScreenshotPath, Actions, Platform, Phase, Timestamp, ModelUsed, Success
- **`training.Action`** -- Type (tap/type/dpad_down), Value (coordinates/text), Reason (model's explanation)
- **`training.TrainingStats`** -- TotalPairs, SuccessPairs, ByPlatform (map), ByModel (map)

## Data Flow

```
Record(screenshot, actions, platform, phase, model, success)
    |
    save PNG to outputDir with sequential numbering (000001.png, 000002.png, ...)
    |
    create TrainingPair with metadata -> append to in-memory list
    |
Export(path) -> for each TrainingPair:
    JSON marshal -> write line to JSONL file
    -> return count of exported pairs

Stats() -> aggregate: count total, successes, group by platform and model
```

## Dependencies

- `github.com/stretchr/testify` -- Test assertions (only dependency)

## Testing Strategy

Table-driven tests with `testify` and race detection. Tests cover Record with filesystem output, sequential file naming, JSONL export format, Stats aggregation by platform and model, concurrent recording, and Pairs() copy-on-read behavior.
