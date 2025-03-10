# runner-go

A simple, dependency-free Go package to run tasks concurrently with automatic error propagation and cancellation support.

## Installation

```bash
go get github.com/bulatsan/runner-go
```

## Features

- Run multiple tasks concurrently
- Automatic cancellation propagation on errors
- Simple and intuitive API
- Zero external dependencies

## Usage

### Basic Usage

Here's a basic example demonstrating concurrent execution, automatic cancellation on errors, and graceful handling of SIGTERM:

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"os/signal"
	"syscall"
	"time"

	"github.com/bulatsan/runner-go"
)

func main() {
	// Create context cancelled on SIGTERM
	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGTERM)
	defer cancel()

	// Task that succeeds after 500ms
	successRunner := runner.New(func(ctx context.Context) error {
		select {
		case <-time.After(500 * time.Millisecond):
			fmt.Println("Task completed successfully")
			return nil
		case <-ctx.Done():
			fmt.Println("Task cancelled:", ctx.Err())
			return ctx.Err()
		}
	})

	// Task that fails intentionally after 100ms
	errorRunner := runner.New(func(ctx context.Context) error {
		time.Sleep(100 * time.Millisecond)
		return errors.New("intentional failure")
	})

	// Combine and run tasks concurrently
	combined := runner.Join(successRunner, errorRunner)

	// Execute combined tasks
	if err := combined.Run(ctx); err != nil {
		fmt.Printf("Execution stopped: %v\n", err)
	}
}
```

### Creating Simple Runners

You can quickly create runners with predefined behaviors:

```go
// Runner that always returns an error
errRunner := runner.Err(errors.New("something went wrong"))

// Runner that always succeeds
okRunner := runner.OK()
```

## License

See the [LICENSE](LICENSE) file for details.
