# gofast [![GoDoc](https://godoc.org/github.com/yookoala/gofast?status.svg)][godoc] [![Travis CI results][travis]](https://travis-ci.org/yookoala/gofast?branch=master)

[godoc]: https://godoc.org/github.com/yookoala/gofast
[travis]: https://api.travis-ci.org/yookoala/gofast.svg?branch=master


**gofast** is a [FastCGI][fastcgi]
"client" library written purely in go.

[fastcgi]: http://www.mit.edu/~yandros/doc/specs/fcgi-spec.html


## What does it do, really?

In FastCGI specification, a FastCGI system has 2 components: (a) web
server; and (b) application server. A web server should hand over
request information to the application server through socket. The
application server always listens to the socket and response to
socket request accordingly.

```
visitor → web server → application server → web server → visitor
```

In ordinary Apache + php-fpm hosting environment, this would be:
```
visitor → Apache → php-fpm → Apache → visitor
```

**gofast** help you to write the code on the web server part of this
picture. It helps you to pass the request to application server and
receive response from it.

Why?
----
Many popular languages (e.g. [Python][python/webservers],
[PHP][php-fpm], [Ruby][rubygem/fcgi]) has FastCGI server
implementations. With **gofast**, you may mix the languages
without too much complication.

Also, this is fun to do :-)

[php-fpm]: http://php.net/manual/en/install.fpm.php
[rubygem/fcgi]: https://rubygems.org/gems/fcgi/versions/0.9.2.1
[python/webservers]: https://docs.python.org/2/howto/webservers.html


## How to Use?

You basically would use the `Handler` as [http.Handler]. You can further mux it
with [default ServeMux][http.NewServeMux] or other compatible routers. You then
serve your fastcgi within this golang http server.

[http.Handler]: https://golang.org/pkg/net/http/#Handler
[mux]: https://golang.org/pkg/net/http/#ServeMux
[http.NewServeMux]: https://golang.org/pkg/net/http/#NewServeMux

### Simple Example

Please note that this is only the **web server** component. You need to start
your **application** component elsewhere.

```go
// this is a very simple fastcgi web server
package main

import (
	"fmt"
	"net/http"
	"os"

	"github.com/yookoala/gofast"
)

// NewHandler returns a fastcgi web server implementation as an http.Handler
func NewHandler(root, network, address string) http.Handler {
	h := gofast.NewHandler(gofast.NewPHPFS(root), network, address)
	return h
}

func main() {
	// get fastcgi address from env FASTCGI_ADDR
	fcgiAddr := os.Getenv("FASTCGI_ADDR")

	// handles static assets in the assets folder
	http.Handle("/assets/",
		http.StripPrefix("/assets/",
			http.FileSystem(http.Dir("/var/www/html/assets"))))

	// handle all scripts in document root
	http.Handle("/", NewHandler("/var/www/html", "tcp", fcgiAddr))

	// serve at 8080 port
	log.Fatal(http.ListenAndServe(":8080", nil))
}

```

### Full Examples

Please see the example usages:

* [PHP]
* [Python3]

[PHP]: example/php
[Python3]: example/python3


### Author

This library is written by [Koala Yeung][author@github].

[author@github]: https://github.com/yookoala/


## Contributing

Your are welcome to contribute to this library.

To report bug, please use the [issue tracker][issue tracker].

To fix an existing bug or implement a new feature, please:

1. Check the [issue tracker][issue tracker] and [pull requests][pull requests] for existing discussion.
2. If not, please open a new issue for discussion.
3. Write tests.
4. Open a pull request referencing the issue.
5. Have fun :-)

[issue tracker]: https://github.com/yookoala/gofast/issues
[pull requests]: https://github.com/yookoala/gofast/pulls


## Licence

This library is release under a BSD-like licence. Please find the
[LICENCE][LICENCE] file in this repository

[LICENCE]: /LICENCE
