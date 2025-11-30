# TLS

https://go-monk.beehiiv.com/p/playing-with-tls-and-go

## https

```go
# generate self signed certificates
mkcert localhost

# server.go
package main

import (
	"io"
	"log"
	"net/http"
)

func main() {
	http.HandleFunc("/", echo)
	log.Fatal(http.ListenAndServeTLS("localhost:4430", "localhost.pem", "localhost-key.pem", nil))
}

func echo(w http.ResponseWriter, r *http.Request) {
	reqBody, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Error reading request body", http.StatusInternalServerError)
		return
	}
	defer r.Body.Close()
	w.Write(reqBody)
}

# run program and send text to program
curl https://localhost:4430 --data hello --cacert localhost.pem
```
