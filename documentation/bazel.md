# Google Bazel

Google Bazel is a scalable system for building and testing large amounts of code
highly concurrently and distributed. It also enables fast incremental
compilation.

### Java library

The [java-tutorial](../examples/java-tutorial) directory contains a simple Java
application.
[com.example.Greeting](../examples/java-tutorial/src/main/java/com/example/Greeting.java]
is a Java library exporting a `sayHi` function. In the
[BUILD file](../examples/java-tutorial/BUILD) file, it has the name `greeter`.

### Java CLI app

[com.example.ProjectRunner](../examples/java-tutorial/src/main/java/com/example/ProjectRunner.java)
is a CLI app that uses the `greeter` Java library as a dependency. This could
also be a micro-service exposing a REST API. In the BUILD file, it has the name
`ProjectRunner`. Let's build the app:

```
bazel build //:ProjectRunner
```

Let's run the app:

```
bazel-bin/ProjectRunner
```

More details at https://docs.bazel.build/versions/3.2.0/tutorial/java.html.

### C++ library and application

The [C++ tutorial stage 3](../examples/cpp-tutorial/stage3) consists of a
package [lib](../examples/cpp-tutorial/stage3/lib/BUILD) that contains a C++
library called `hello-time`. It exports a function `print_localtime()` that
prints the local time to STDOUT. It also contains a package
[main](../examples/cpp-tutorial/stage3/main/BUILD) that contains a C++ library
`hello-greet` that exports a `get_greet` function as well as a CLI app
`hello-world` that uses both the `hello-greet` library from this package and the
`hello-time` library from the `lib` package.

Compile the CLI app including all its dependencies:

```
bazel build //main:hello-world
```

### TypeScript library

- Angular app
- Python app
