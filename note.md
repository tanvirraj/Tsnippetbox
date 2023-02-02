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

- **(\*)** The sql.Open() function returns a sql.DB object. This isn’t a database connection — it’s a
  pool of many connections. This is an important difference to understand. Go manages the
  connections in this pool as needed, automatically opening and closing connections to the
  database via the driver.

- The internal directory is being used to hold ancillary non-applicationspecific code, which could potentially be reused. A database model which could be used
  by other applications in the future (like a command line interface application) fits the bill
  here.

- we’re returning the ErrNoRecord error from our SnippetModel.Get() method, instead of sql.ErrNoRows directly. The reason is to help encapsulate the model completely, so that our application isn’t concerned with the underlying datastore or reliant on datastore-specific errors for its behavior.

```
  if errors.Is(err, sql.ErrNoRows) {
		return nil, ErrNoRecord
	} else {
		return nil, err
	}
```

- As you’re probably starting to realize, the database/sql package essentially provides a
  standard interface between your Go application and the world of SQL databases.
  So long as you use the database/sql package, the Go code you write will generally be
  portable and will work with any kind of SQL database — whether it’s MySQL, PostgreSQL,
  SQLite or something else. This means that your application isn’t so tightly coupled to the
  database that you’re currently using, and the theory is that you can swap databases in the
  future without re-writing all of your code (driver-specific quirks and SQL implementations
  aside).
  It’s important to note that while database/sql generally does a good job of providing a
  standard interface for working with SQL databases, there are some idiosyncrasies in the way
  that different drivers and databases operate. It’s always a good idea to read over the
  documentation for a new driver to understand any quirks and edge cases before you begin
  using it.

- the Exec(), Query() and QueryRow() methods all use prepared
  statements behind the scenes to help prevent SQL injection attacks. They set up a prepared
  statement on the database connection, run it with the parameters provided, and then close
  the prepared statement.

- a simple middleware pattern code example

```
func myMiddleware(next http.Handler) http.Handler {
	fn := func(w http.ResponseWriter, r *http.Request) {
		// TODO: Execute our middleware logic here...
		next.ServeHTTP(w, r)
	}
	return http.HandlerFunc(fn)
}

```

in very simple terms `myMiddleware()` is a function that
accepts the next handler in a chain as a parameter. It returns a handler which executes some
logic and then calls the next handler.

very common middleware pattern in most app

```
func myMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// TODO: Execute our middleware logic here...
		next.ServeHTTP(w, r)
	})
}

```

- In any middleware handler, code which comes before `next.ServeHTTP()` will be executed on
  the way down the chain, and any code after `next.ServeHTTP()` — or in a deferred function —
  will be executed on the way back up.

```
func myMiddleware(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Any code here will execute on the way down the chain.
		next.ServeHTTP(w, r)
		// Any code here will execute on the way back up the chain.
	})
}

```

- It’s important to realise that our middleware will only recover panics that happen in the same
  goroutine that executed the recoverPanic() middleware.
  if you are spinning up additional goroutines from within your web application and there is
  any chance of a panic, you must make sure that you recover any panics from within those too.
