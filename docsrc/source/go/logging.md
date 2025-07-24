# Logging

https://www.dash0.com/guides/logging-in-go-with-slog

## Understanding slog fundamentals

The log/slog package is built around three core types: the 
* Logger, is the frontend you’ll interact with 
* Handler, is the backend that does the actual logging work
* Record, is the data passed between them. A Record represents a single log event.

## Handler
A Handler is an interface that’s responsible for processing Records. It’s the engine that determines how and where logs are written. It’s responsible for:

* Formatting the Record into a specific output, like JSON or plain text.
* Writing the formatted output to a destination like the console or a file.

The log/slog package includes built-in concrete TextHandler and JSONHandler implementations, but you can create custom handlers to meet any requirement. This interface is what makes slog so flexible.

## Logger
The Logger is the entry point for creating logs. 

```go
// Creates a new Logger that uses a JSONHandler to write to standard output
logger := slog.New(slog.NewJSONHandler(os.Stdout,nil))

// This call creates a Record and passes it to the JSONHandler
logger.Info("user logged in","user_id",123)
```

### context

slog also provides a context-aware version for each level, such as InfoContext(). These variants accept a context.Context type as their first argument, allowing context-aware handlers (if configured) to extract and log values carried within the context:

```go
logger.InfoContext(context.Background(),"an info message")
```

Note that slog’s context-aware methods will not automatically pull values from the provided context when using the built-in handlers. You must use a context-aware handler for this pattern to work.

```go
logger.Log(context.Background(), slog.LevelInfo,"an info message")
logger.LogAttrs(context.Background(), slog.LevelInfo,"an info message")
```

## Adding contextual attributes to your logs

```go
logger.Info("incoming request","method","GET","status",200)
```

It is possible, that the log entry is corrupt, if f.i. a key is missing (only the value entry)

To guarantee correctness, you must use the strongly-typed slog.Attr helpers. They makes it impossible to create an unbalanced pair by catching errors at compile time:

```go
logger.Warn(
    "permission denied",
    slog.Int("user_id", 12345),
    slog.String("resource", "/api/admin"),
)
```

## Log Level

```go
if logger.Enabled(context.Background(), slog.LevelDebug) {
    // This code will not run when the logger's level is INFO or greater
    // function getExpensiveDebugData must return error
    logger.Debug("operation complete", "data", getExpensiveDebugData())
}
```

### Setting the minimum level

```go
handler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.WarnLevel,
})
logger := slog.New(handler)
```

To set the level based on an environmental variable, you may use this pattern:

```go
func getLogLevelFromEnv() slog.Level {
    levelStr := os.Getenv("LOG_LEVEL")

    switch strings.ToLower(levelStr) {
    case "debug":
        return slog.LevelDebug
    case "warn":
        return slog.LevelWarn
    case "error":
        return slog.LevelError
    default:
        return slog.LevelInfo
    }
}

func main() {
    logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
        Level: getLogLevelFromEnv(),
    }))
}
```

### Dynamically updating log verbosity

```go
var logLevel slog.LevelVar // INFO is the zero value
// the initial value is set from the environment and you can call Set() anytime
// to update this value
logLevel.Set(getLogLevelFromEnv())

logger := slog.New(slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level: &logLevel,
}))
```

For even greater control of severity levels on a per-package basis, you can use the slog-env package which provides a handler that allows setting the log level via the GO_LOG environmental variable:

```go
logger := slog.New(slogenv.NewHandler(slog.NewJSONHandler(os.Stderr,nil)))
```

enable DEBUG f.i. with 

```go
GO_LOG=debug ./myapp
```

### creating custom levels

```go
const (
	LevelTrace = slog.Level(-8) // More verbose than DEBUG
	LevelFatal = slog.Level(12) // More severe than ERROR
)
```

To use these custom levels, you must use the generic logger.Log()  method:

```go
logger.Log(context.Background(), LevelFatal,"database connection lost")
```

However, their default output name isn’t ideal (DEBUG-4,ERROR+4). You can fix this by providing a 
ReplaceAttr() function in your HandlerOptions to map the level’s integer value to a custom string:

```go
opts := &slog.HandlerOptions{
    ReplaceAttr: func(groups []string, a slog.Attr) slog.Attr {
        if a.Key == slog.LevelKey {
            level := a.Value.Any().(slog.Level)
            switch level {
                case LevelTrace:
                    a.Value = slog.StringValue("TRACE")
                case LevelFatal:
                    a.Value = slog.StringValue("FATAL")
            }
        }
        return a
    },
}

handler := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{
    Level:       slog.LevelDebug,
    ReplaceAttr: opts.ReplaceAttr,
})
logger := slog.New(handler)
```

