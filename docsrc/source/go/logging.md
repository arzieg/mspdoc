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

## Controlling the logger output with Handlers

The Handler is the backend of the logging system that’s responsible for taking a Record, formatting it, and writing it to a destination.

A key feature of slog handlers is their composability. Since handlers are just interfaces, it’s easy to create “middleware” handlers that wrap other handlers.

This allows you to build a processing pipeline to enrich, filter, or modify log records before they are finally written. You’ll see some examples of this pattern as we go along.

The log/slog package ships with two built-in handlers:

* JSONHandler, which formats logs as JSON.
* TextHandler, which formats logs as key=value pairs.

```go
jsonLogger := slog.New(slog.NewJSONHandler(os.Stdout,nil))
textLogger := slog.New(slog.NewTextHandler(os.Stdout,nil))

jsonLogger.Info("database connected","db_host","localhost","port",5432)
textLogger.Info("database connected","db_host","localhost","port",5432)
```

### Customizing handlers with HandlerOptions

You can configure the behavior of the built-in handlers using slog.HandlerOptions, and you’ve already seen this approach for setting the 
**Level** and using **ReplaceAttrs** to provide custom level names.

The final option is **AddSource**, which automatically includes the source code file, function, and line number in the log output:


```go
opts := &slog.HandlerOptions{
    AddSource: true,
}
logger := slog.New(slog.NewJSONHandler(os.Stdout, opts))

logger.Warn("storage space is low")
```

While source information is handy to have, it comes with a performance penalty because slog must call runtime.Caller() to get the source code information, so keep that in mind.

### Logging to files
The best practice for modern applications is often to log to stdout or stderr, and allow the runtime environment to manage the log stream.

However, if your application needs to write directly to a file, you can simply pass an *os.File instance to the slog  handler:

```go
logFile, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
if err != nil {
    panic(err)
}

defer logFile.Close()

logger := slog.New(slog.NewJSONHandler(logFile, nil))

logger.Info("Starting server...", "port", 8080)
logger.Warn("Storage space is low", "remaining_gb", 15)
logger.Error("Database connection failed", "db_host", "10.0.0.5")
```

## Contextual logging patterns with slog

This guide explores the three most common patterns for contextual logging in Go: 
1. using a global logger, 
2. embedding the logger in the context, 
3. and passing the logger explicitly as a dependency.

### 1. Using a global logger with a context handler

Using the global logger via slog.Info() is a convenient approach as it avoids the need to pass a logger instance through every function call.

You only need to configure the default logger once at the entry point of the program, and then you’re free to use it anywhere in your application:

```go
func main() {
    // Configure the default logger once.
    slog.SetDefault(slog.New(slog.NewJSONHandler(os.Stdout, nil)))

    doSomething()
}

func doSomething() {
    // Use it anywhere without passing it.
    slog.Info("doing something")
}
```

When you want to log contextual attributes across scopes, you only need to use the context.Context type to wrap the attributes and then use the Context variants of the level methods accordingly.

This requires the use of a context-aware handler, and there are a few of these already created by the community. One example is slog-context which allows you place slog attributes into the context and have them show up anywhere that context is used.

Here’s a detailed example showing this pattern:

```go
package main

import (
	"log/slog"
	"net/http"
	"os"

	"github.com/google/uuid"
	slogctx "github.com/veqryn/slog-context"
)

const (
	correlationHeader = "X-Correlation-ID"
)

func requestID(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		ctx := r.Context()

		correlationID := r.Header.Get(correlationHeader)

		if correlationID == "" {
			correlationID = uuid.New().String()
		}

		ctx = slogctx.Prepend(ctx, slog.String("correlation_id", correlationID))

		r = r.WithContext(ctx)

		w.Header().Set(correlationHeader, correlationID)

		next.ServeHTTP(w, r)
	})
}

func requestLogger(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		slog.InfoContext(
			r.Context(),
			"incoming request",
			slog.String("method", r.Method),
			slog.String("path", r.RequestURI),
			slog.String("referrer", r.Referer()),
			slog.String("user_agent", r.UserAgent()),
		)

		next.ServeHTTP(w, r)
	})
}

func hello(w http.ResponseWriter, r *http.Request) {
	slog.InfoContext(r.Context(), "hello world!")
}

func main() {
	h := slogctx.NewHandler(slog.NewJSONHandler(os.Stdout, nil), nil)
	slog.SetDefault(slog.New(h))

	mux := http.NewServeMux()

	mux.HandleFunc("/", hello)

	wrappedMux := requestID(requestLogger(mux))

	http.ListenAndServe(":3000", wrappedMux)
}
```

The 
requestID()
 middleware intercepts every incoming request, generates a unique 
correlation_id
, and uses 
slogctx.Prepend()
 to attach this ID as a logging attribute to the request’s context.

The requestLogger() middleware and the final hello() handler both use slog.InfoContext(). They don’t need to know about the 
correlation_id explicitly; they just pass the request’s context to the global logger.

When slog.InfoContext() is called, the configured slogctx.Handler intercepts the call, inspects the provided context, finds the 
correlation_id attribute, and automatically adds it to the log record before it’s written out by the JSONHandler.

This pattern ensures that every log statement related to a single HTTP request is tagged with the same correlation_id, making it possible to connect a set of logs to a single request.

### 2. Embedding the logger in the context

Another common pattern is placing the logger itself in a context.Context instance. You can also use the slog-context package to implement this pattern:

```go
package main

import (
	"log/slog"
	"net/http"
	"os"

	"github.com/google/uuid"
	slogctx "github.com/veqryn/slog-context"
)

const (
	correlationHeader = "X-Correlation-ID"
)

func requestID(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		ctx := r.Context()

		correlationID := r.Header.Get(correlationHeader)

		if correlationID == "" {
			correlationID = uuid.New().String()
		}

		ctx = slogctx.With(ctx, slog.String("correlation_id", correlationID))

		r = r.WithContext(ctx)

		w.Header().Set(correlationHeader, correlationID)

		next.ServeHTTP(w, r)
	})
}

func requestLogger(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		logger := slogctx.FromCtx(r.Context())
		logger.Info(
			"incoming request",
			slog.String("method", r.Method),
			slog.String("path", r.RequestURI),
			slog.String("referrer", r.Referer()),
			slog.String("user_agent", r.UserAgent()),
		)

		next.ServeHTTP(w, r)
	})
}

func ctxLogger(logger *slog.Logger, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		ctx := slogctx.NewCtx(r.Context(), logger)

		r = r.WithContext(ctx)

		next.ServeHTTP(w, r)
	})
}

func hello(w http.ResponseWriter, r *http.Request) {
	logger := slogctx.FromCtx(r.Context())
	logger.Info("hello world!")
}

func main() {
	h := slogctx.NewHandler(slog.NewJSONHandler(os.Stdout, nil), nil)

	logger := slog.New(h)

	mux := http.NewServeMux()

	mux.HandleFunc("/", hello)

	wrappedMux := ctxLogger(logger, requestID(requestLogger(mux)))

	http.ListenAndServe(":3000", wrappedMux)
}
```

Here, The outermost middleware, ctxLogger(), takes the application’s base logger and uses slogctx.NewCtx() to place it into the request’s context. This makes the logger available to all subsequent handlers.

Next, the requestID middleware retrieves the logger from the context. It then uses slogctx. With to create a new child logger that includes the correlation_id. This new, more contextual logger is then placed back into the context, replacing the base logger.

Any subsequent middleware or handler, like requestLogger() and hello(), can now retrieve the fully contextualized child logger using 
slogctx.FromCtx(). They can log messages without needing to know anything about the correlation_id; it’s automatically included because it’s part of the logger instance that was retrieved.

### 3. Explicitly passing the logger

This approach treats the logger as a formal dependency, which is provided to components either through function parameters or as a field in a struct.

The logger is provided once when the struct is created, and all its methods can then access it via the receiver:

```go
type UserService struct {
    logger *slog.Logger
    db     *sql.DB
}

func NewUserService(logger *slog.Logger, db *sql.DB) *UserService {
    return &UserService{
        logger: logger.With(slog.String("component", "UserService")), // Create a child logger for the component
        db:     db,
    }
}

func (s *UserService) CreateUser(ctx context.Context, user *User) {
    l := s.logger.With(slog.Any("user", user))

    l.InfoContext(ctx, "creating new user")

    // ...

    l.InfoContext(ctx, "user created successfully")
}
```

For context-aware logging, you would then rely on adding attributes to the context with slogctx.Prepend() as shown earlier.


### Which should you use? 
slog’s design encourages handlers to read contextual values from a context.Context. This makes putting the 
Logger instance itself in the context unnecessary, and thus **not** recommended.

The key decision is thus between two patterns: using a global logger or using dependency injection. The former is extremely convenient but adds a hidden dependency that’s hard to test, while the latter is more verbose but makes dependencies explicit, resulting in highly testable and flexible code.

## Controlling log output with the LogValuer interface

The LogValuer interface provides a powerful mechanism for controlling how your custom types appear in log output.

This becomes particularly important when dealing with sensitive data, complex structures, or when you want to provide consistent representation of domain objects across your logging.

```go
type LogValuer interface {
   LogValue() slog.Value
}
```

By implementing LogValuer, you can control exactly what information appears. For example, you can limit it to just the 
id.

```go
// Implement LogValuer to control log representation
func (u *User) LogValue() slog.Value {
   return slog.GroupValue(
       slog.String("id", u.ID),
   )
}
```

## Error logging with slog

Error logging in slog requires thoughtful consideration of what information will be most valuable during debugging. Unlike simple string-based logging, structured error logging allows you to capture rich context alongside the error itself.

The most straightforward approach uses slog.Any() to log error values:

```go
err := errors.New("payment gateway unreachable")
if err != nil {
    logger.Error("Payment processing failed", slog.Any("error", err))
}
```

If you’re using a custom error type, you can implement the LogValuer interface to enrich your error logs:

```go
type PaymentError struct {
   Code    string
   Message string
   Cause   error
}

func (pe PaymentError) Error() string {
   return pe.Message
}

func (pe PaymentError) LogValue() slog.Value {
   return slog.GroupValue(
       slog.String("code", pe.Code),
       slog.String("message", pe.Message),
       slog.String("cause", pe.Cause.Error()),
   )
}

func main() {
   logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))

   causeErr := errors.New("network timeout")
   err := PaymentError{
       Code:    "GATEWAY_UNREACHABLE",
       Message: "Failed to reach payment gateway",
       Cause:   causeErr,
   }

   logger.Error("Payment operation failed", slog.Any("error", err))
}
```

## Bringing your logs into an observability pipeline

Centralizing your logs transforms them from simple diagnostic records into a powerful, queryable dataset. More importantly, it allows you to correlate slog entries with other critical telemetry signals, like distributed traces and metrics, to get a complete picture of your system’s health.

Modern observability platforms can ingest the structured JSON output from slog’s JSONHandler. They provide powerful tools for searching, creating dashboards, and alerting on your log data.

To unlock true correlation, however, your logs must share a common context (like a TraceID) with your traces. The standard way to achieve this is by integrating slog with OpenTelemetry using the otelslog bridge.

