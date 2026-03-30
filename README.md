# digital.vasic.trainingcollector

Collects screenshot + action pairs during autonomous QA sessions for future vision model fine-tuning. Exports training data in JSONL format suitable for supervised learning pipelines.

## Installation

```bash
go get digital.vasic.trainingcollector
```

## Quick Start

```go
package main

import (
    "fmt"

    "digital.vasic.trainingcollector/pkg/training"
)

func main() {
    // Create a collector that stores screenshots in a directory.
    tc := training.NewTrainingCollector("./training-data")

    // Record a screenshot + action observation.
    err := tc.Record(
        screenshotPNG,          // raw PNG bytes
        []training.Action{
            {Type: "tap", Value: "500,300", Reason: "login button"},
            {Type: "type", Value: "user@example.com", Reason: "email field"},
        },
        "android",              // platform
        "curiosity",            // pipeline phase
        "gemini-2.0-flash",    // vision model used
        true,                   // success
    )
    if err != nil {
        panic(err)
    }

    // Export all pairs to JSONL for training pipelines.
    count, err := tc.Export("./output/training.jsonl")
    if err != nil {
        panic(err)
    }
    fmt.Printf("Exported %d training pairs\n", count)

    // Check statistics.
    stats := tc.Stats()
    fmt.Printf("Total: %d, Success: %d\n",
        stats.TotalPairs, stats.SuccessPairs)
    fmt.Printf("By platform: %v\n", stats.ByPlatform)
    fmt.Printf("By model: %v\n", stats.ByModel)
}
```

## API Reference

### TrainingCollector

The core type. All methods are safe for concurrent use.

| Method | Description |
|--------|-------------|
| `NewTrainingCollector(outputDir string)` | Create a collector writing screenshots to outputDir |
| `Record(screenshot, actions, platform, phase, model, success)` | Save a screenshot + action pair |
| `Export(path string) (int, error)` | Write all pairs to a JSONL file |
| `Stats() TrainingStats` | Get collection statistics |
| `Len() int` | Count collected pairs |
| `Pairs() []TrainingPair` | Get a copy of all collected pairs |

### TrainingPair

| Field | Type | Description |
|-------|------|-------------|
| `ScreenshotPath` | `string` | Path to the saved PNG file |
| `Actions` | `[]Action` | Actions taken when viewing this screenshot |
| `Platform` | `string` | Target platform (e.g. "android", "web") |
| `Phase` | `string` | Pipeline phase (e.g. "curiosity", "execute") |
| `Timestamp` | `time.Time` | When the observation was recorded |
| `ModelUsed` | `string` | Vision model identifier |
| `Success` | `bool` | Whether the actions succeeded |

### Action

| Field | Type | Description |
|-------|------|-------------|
| `Type` | `string` | Action type (e.g. "tap", "type", "dpad_down") |
| `Value` | `string` | Action value (coordinates, text, etc.) |
| `Reason` | `string` | Model's explanation for this action |

### TrainingStats

| Field | Type | Description |
|-------|------|-------------|
| `TotalPairs` | `int` | Total recorded pairs |
| `SuccessPairs` | `int` | Pairs where Success is true |
| `ByPlatform` | `map[string]int` | Count per platform |
| `ByModel` | `map[string]int` | Count per vision model |

### JSONL Export Format

Each line in the exported file is a JSON object representing one `TrainingPair`. This format is directly consumable by most ML training frameworks.

## License

Apache-2.0
