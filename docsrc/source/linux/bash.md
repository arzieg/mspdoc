# Remarks on bash

https://quickref.me/bash


# Search String contain Substring
https://nickjanetakis.com/blog/check-if-a-string-contains-a-substring-in-bash

```
#!/usr/bin/env bash

set -o errexit
set -o pipefail
set -o nounset

ENV="${1:-prod}"
DEPLOYABLE_ENVS="${DEPLOYABLE_ENVS:-pen,prod}"

# This is the condition. We're using * to match anything on either side of the
# string Of course you can adjust this as needed for your case. It's 
# important that the wildcard is on the rigjt side of the condition.
if [[ "${DEPLOYABLE_ENVS}" == *"${ENV}"* ]]; then
	echo "Yes"
else
	echo "No"
	exit
fi
```

# Error Handling in bash

https://jsdev.space/error-handling-bash/

## Exit Status check

```
if ! command -v docker &> /dev/null; then
    echo "Error: Docker is not installed"
    exit 1
fi

# Check if file exists
if [ ! -f "config.txt" ]; then
    echo "Error: config.txt not found"
    exit 1
fi
```

## Exit on Error (set -e)

```
set -e
mkdir /root/test_dir
echo "Directory created successfully."
```

Best Practices:
* Place set -e at the beginning of your script
* Use || true for commands that are allowed to fail
* Consider combining with set -o pipefail

## Custom Error Handling with trap

```
# Define cleanup function
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/tempfile
    exit 1
}

# Set trap for script termination
trap cleanup EXIT ERR SIGINT SIGTERM

# Your script continues here
```

Common Signals to Handle:
* EXIT: Script exit (normal or abnormal)
* ERR: Any command returning non-zero
* SIGINT: Interrupt signal (Ctrl+C)
* SIGTERM: Termination signal

## Error Functions

```
error_exit() {
    echo "Error: $1" >&2
    exit "${2:-1}"
}

warn() {
    echo "Warning: $1" >&2
}

# Usage
[ -f "required_file.txt" ] || error_exit "Required file not found"
```

**Redirect errors to logfile**

```
exec 2>error_log.txt
mkdir /root/test_dir
```

**Line specific errors**

```
handle_error() {
  echo "Error on line $1"
  exit 1
}
trap 'handle_error $LINENO' ERR
mkdir /root/test_dir
```

## Verbose Mode and Debugging

```
# Enable debug mode with -v flag
if [[ "$1" == "-v" ]]; then
    set -x  # Print each command
    VERBOSE=true
    shift
fi

debug() {
    if [ "$VERBOSE" = true ]; then
        echo "DEBUG: $1" >&2
    fi
}

# Usage
debug "Checking system requirements..."
```

## Best Practices for Error Handling

1. Always Check Return Values

2. Provide Meaningful Error Messages

* Include specific details about what went wrong
* Mention which operation failed
* Include relevant file names or parameters

3. Clean Up on Exit

* Remove temporary files
* Reset system states
* Close network connections

4. Log Errors Appropriately

```
log_error() {
    local msg="$1"
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "$timestamp ERROR: $msg" >> "/var/log/myscript.log"
}
```

5. Handle Different Error Types

* Distinguish between fatal and non-fatal errors
* Implement different recovery strategies based on error type
* Consider retry mechanisms for transient failures

```
#!/bin/bash

# Error handling setup
set -e
set -o pipefail

# Global variables
VERBOSE=false
LOG_FILE="/var/log/myscript.log"

# Error handling functions
error_exit() {
    log_error "$1"
    exit "${2:-1}"
}

log_error() {
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "$timestamp ERROR: $1" >> "$LOG_FILE"
    echo "ERROR: $1" >&2
}

debug() {
    if [ "$VERBOSE" = true ]; then
        echo "DEBUG: $1" >&2
    fi
}

cleanup() {
    debug "Performing cleanup..."
    # Add cleanup tasks here
}

# Set trap
trap cleanup EXIT ERR SIGINT SIGTERM

# Main script logic with error handling
main() {
    debug "Starting script execution"
    
    # Check prerequisites
    if ! command -v required_command &> /dev/null; then
        error_exit "Required command not found" 2
    fi
    
    # Process with error handling
    if ! process_data; then
        error_exit "Data processing failed" 3
    fi
    
    debug "Script completed successfully"
}

# Run main function
main "$@"
```