- During development the `go run` command is a convenient way to try out your code.
  It’s essentially a shortcut that compiles your code,
  creates an executable binary in your /tmp directory, and then runs this binary in one step.

- So, for the sake of security, it’s generally a good idea to avoid `DefaultServeMux` and
  the corresponding helper functions. Use your own locally-scoped servemux instead,
  like we have been doing in this project so far.

- When it comes to pattern matching, any host-specific patterns will be checked first and if there is a match the request will be dispatched to the corresponding handler. Only when there isn’t a host-specific match found will the non-host specific patterns also be checked.

- If you don’t call w.WriteHeader() explicitly, then the first call to w.Write() will automaticallysenda200 OKstatuscodetotheuser.So,ifyouwanttosendanon-200 status code, you must call w.WriteHeader() before any call to w.Write().

- When our server receives a new HTTP request, it calls
  the servemux’s ServeHTTP() method. This looks up the relevant handler based on the
  request URL path, and in turn calls that handler’s ServeHTTP() method. You can think of a Go
  web application as a chain of ServeHTTP() methods being called one after another

- (\*)all incoming HTTP requests are
  served in their own goroutine. For busy servers, this means it’s very likely that the code in or
  called by your handlers will be running concurrently.

- for env variable we can use `os.Getenv()` but it has some drawbacks

  - (the return value from os.Getenv() is the empty string if the environment
    variable doesn’t exist)
  - you don’t get the -help functionality that you do with command-line
    flags, and the return value from os.Getenv() is always a string — you don’t get automatic
    type conversions like you do with flag.Int() and the other command line flag functions

- As a rule of thumb, you should avoid using the `Panic()` and `Fatal()` variations outside of
  your `main()` function — it’s good practice to return errors instead, and only panic or exit
  directly from `main()`.
