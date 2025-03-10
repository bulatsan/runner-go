# runner-go

A lightweight and flexible runner abstraction for Go applications that makes concurrent task execution simple.

## Installation

```bash
go get github.com/bulatsan/runner-go
```

## Features

- Simple interface with a single `Run` method
- Create runners from any function
- Combine multiple runners to execute in parallel
- Automatic cancellation propagation when errors occur
- Easy error handling
- Zero dependencies

## Usage

### Basic Usage

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/bulatsan/runner-go"
)

func main() {
	// Create a simple runner
	simpleRunner := runner.New(func(ctx context.Context) error {
		fmt.Println("Running a task...")
		time.Sleep(100 * time.Millisecond)
		fmt.Println("Task completed!")
		return nil
	})

	// Run it
	err := simpleRunner.Run(context.Background())
	if err != nil {
		fmt.Printf("Error: %v\n", err)
	}
}
```

### Returning Errors

```go
// Create a runner that always returns an error
errRunner := runner.Err(fmt.Errorf("something went wrong"))

// Or create a runner that always succeeds
okRunner := runner.OK()
```

### Running Multiple Tasks in Parallel

```go
package main

import (
	"context"
	"fmt"
	"time"

	"github.com/bulatsan/runner-go"
)

func main() {
	// Create several runners
	runner1 := runner.New(func(ctx context.Context) error {
		fmt.Println("Runner 1 started")
		time.Sleep(100 * time.Millisecond)
		fmt.Println("Runner 1 completed")
		return nil
	})

	runner2 := runner.New(func(ctx context.Context) error {
		fmt.Println("Runner 2 started")
		time.Sleep(200 * time.Millisecond)
		fmt.Println("Runner 2 completed")
		return nil
	})

	// Join them to run in parallel
	combined := runner.Join(runner1, runner2)
	
	// Run them all at once
	err := combined.Run(context.Background())
	if err != nil {
		fmt.Printf("Error: %v\n", err)
	} else {
		fmt.Println("All runners completed successfully")
	}
}
```

### Error Handling and Cancellation

When using `Join`, if any runner returns an error, all other runners will be cancelled:

```go
package main

import (
	"context"
	"fmt"
	"time"
	"errors"

	"github.com/bulatsan/runner-go"
)

func main() {
	// Create a runner that fails quickly
	failingRunner := runner.New(func(ctx context.Context) error {
		fmt.Println("Failing runner started")
		time.Sleep(50 * time.Millisecond)
		fmt.Println("Failing runner returning error")
		return errors.New("something went wrong")
	})

	// Create a runner that takes longer
	slowRunner := runner.New(func(ctx context.Context) error {
		fmt.Println("Slow runner started")
		select {
		case <-time.After(1 * time.Second):
			fmt.Println("Slow runner completed")
			return nil
		case <-ctx.Done():
			fmt.Println("Slow runner was cancelled:", ctx.Err())
			return ctx.Err()
		}
	})

	// Join them
	combined := runner.Join(failingRunner, slowRunner)
	
	// The failing runner will cause the slow runner to be cancelled
	err := combined.Run(context.Background())
	fmt.Printf("Error: %v\n", err) // Will print the error from failingRunner
}
```

## License

See the [LICENSE](LICENSE) file for details.