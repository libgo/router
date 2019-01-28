### router is combo of httprouter and negroni.

```
go get -u -v github.com/libgo/router
```

#### usage:

```go
package main

import (
	"net"
	"net/http"

	"github.com/libgo/router"
)

func Hello(rw http.ResponseWriter, r *http.Request) {
}

func MidLog(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
	// before
	next(rw, r)
	// after
}

// normal http handler
func UserGet(rw http.ResponseWriter, r *http.Request) {
	p := router.GetParams(r) // or router.GetParams(r.Context())
	_ = p.ByName("id")
}

// with params
func UserXGet(rw http.ResponseWriter, r *http.Request, p router.Params) {
	_ = p.ByName("id")
}

func main() {
	ln, err := net.Listen("tcp", "127.0.0.1:3000")
	if err != nil {
		panic(err)
	}
	defer ln.Close()

	r := router.New()
	r.GET("/hello", router.Wrap(Hello))

	// group with middleware
	api := r.Group("/api", MidLog)
	api.GET("/user/:id", router.Wrap(UserGet))
	api.GET("/user2/:id", router.WrapX(UserXGet))

	server := &http.Server{
		Handler: r,
	}

	err = server.Serve(ln)
	if err != nil {
		panic(err)
	}
}
```