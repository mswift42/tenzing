# Tenzing, the awesome Clojurescript application template. [![Get help on Clojureverse](https://img.shields.io/badge/tenzing-clojureverse-green.svg)](http://clojureverse.org/c/tenzing)

Tenzing is a [Chestnut](https://github.com/plexus/chestnut)-inspired
Clojurescript template offering the following features:

1.  Incremental Clojurescript compilation
2.  Live reloading of your Javascript, CSS, etc.
3.  Browser-REPL

**There are some significant differences though:**

1.  Tenzing uses [Boot](https://github.com/boot-clj/boot) instead of Leiningen (see below)
2.  Tenzing does not provide a backend layer (see below)
3.  Tenzing allows you to choose between Om, Reagent and others

# Rationale

## Why Boot?

In contrast to Leiningen Boot offers a clear strategy when it comes to
composing multi-step build processes such as compiling stylesheets and
Javascript whenever a relevant file changes.

Many Leinigen plugins come with an \`auto\` task that allows similar
behavior. If you want to run multiple of those tasks it's usually done
by starting multiple JVM instances which can lead to
[high memory usage](https://github.com/plexus/chestnut/issues/49). Boot
allows this sort of behaviour to reside in one JVM process while
making sure that build steps don't interfere with each other.

You can learn more about Boot in
[a blog post by one of the authors](http://adzerk.com/blog/2014/11/clojurescript-builds-rebooted/),
it's [github project](https://github.com/boot-clj/boot) or
[a blog post I wrote about it](http://www.martinklepsch.org/posts/why-boot-is-relevant-for-the-clojure-ecosystem.html).

## Why #noBackend?

Tenzing is designed with prototyping in mind. Instead of writing your
own backend you're encouraged to use services like
[Parse](https://parse.com), [Firebase](https://www.firebase.com),
[Usergrid](http://usergrid.incubator.apache.org) and others.

If you figure out that you need a Clojure based backend down the road
it's simple to either add it yourself or create it as a standalone
service that's being used by your clients.

Please, also consider
[offline first](http://alistapart.com/article/offline-first) as an
approach for building early iterations of your application.

> If you're wondering how files are served during development: there
> is a boot task \`serve\` that allows you to serve static files.

# Usage

## Create a Project

To create a new project we piggieback the existing `lein new` tooling:

    $ lein new tenzing your-app

There are a bunch of options that determine what your newly created
project will contain:

-   `+om` provides a basic [Om](https://github.com/omcljs/om)
    application and adds relevant dependencies
-   `+reagent` provides a basic
    [Reagent](https://github.com/reagent-project/reagent) application
    and adds relevant dependencies
-   `+divshot` adds divshot.json for easy deployment to
    [Divshot](https://divshot.com)
-   `+garden` sets up [Garden](https://github.com/noprompt/garden) and
    integrates into the build process
-   `+sass` sets up [Sass](http://sass-lang.com) and integrates into
    the build process (requires [libsass](http://libsass.org))
-   `+less` sets up [Less](http://lesscss.org/) and integrates into
    the build process.
-   `+test` adds a
    [cljs test-runner](https://github.com/crisptrutski/boot-cljs-test)
    and adds a `test` task.

If you want to add an option,
[pull-requests](https://github.com/martinklepsch/tenzing) are welcome.

## Running it

After you [installed Boot](https://github.com/boot-clj/boot#install)
you can run your Clojurescript application in "development mode" by
executing the following:

    $ boot dev

After a moment of waiting you can head to
[localhost:3000](http://localhost:3000) to see a small sample app. If
you now go and edit one of the Clojurescript source files or a SASS
file (if you've used the `+sass` option) this change will be picked up
by Boot and the respective source file will get compiled. When a
compiled file changes through that mechanism it will get pushed to the
browser.

If you've use the `+test` option, then you'll be able to run unit
tests via `boot test`. Use `boot auto-test` to have tests
automatically rerun on file changes.

### Connecting to the browser REPL

After you started your application with `boot dev` there will be a
line printed like the following:

    nREPL server started on port 63518 on host 0.0.0.0

This means there now is an nREPL server that you can connect to. You
can do this with your favorite editor or just by running `boot repl
--client` in the same directory.

Once you are connected you can get into a Clojurescript REPL by
running `(start-repl)`. At this point I usually reload my browser one
last time to make sure the REPL connection is properly setup.

Now you can run things like `(.log js/console "test")`, which should
print "test" in the console of your browser.

### How it works

If you look at the `build` and `run` tasks in the `build.boot` file of
your newly created project you will see something like the following:

```clojure
(deftask build [] (comp (speak) (cljs) (sass :output-dir "css")))

(deftask run [] (comp (serve) (watch) (cljs-repl) (reload)
    (build)))
```

Basically this composes all kinds of build steps into a unified `run`
task that will start our application. From top to bottom:

The `build` task consists of three other tasks:
-   `speak` gives us audible notifications about our build process
-   `cljs` will compile Clojurescript source files to Javascript
-   `sass` will compile Sass source files to CSS

Now if we just run `boot build` instead of the aforementioned `boot
dev` we will compile our Clojurescript and Sass exactly once and then
the program will terminate.

This is where the `run` task comes in:
-   `serve` starts a webserver that will serve our compiled JS, CSS
    and anything else that is in `resources/`
-   `watch` will watch our filesystem for changes and trigger new
    builds when they occur
-   `cljs-repl` sets up various things so we can connect to our
    application through a browser REPL
-   `reload` will watch the compiled files for changes and push them
    to the browser
-   `build` does the things already described above

**Please note that all tasks, except the one we defined ourselves have
  extensive documentation that you can view by running `boot
  <taskname> -h` (e.g. `boot cljs-repl -h`).**

## Deployment

The easiest way to deploy your app is using
[Divshot](https://divshot.com):

1.  `$ divshot login`
2.  add
    [divshot.json](https://github.com/martinklepsch/tenzing/blob/master/resources/leiningen/new/tenzing/divshot.json)
    (Only required if your project hasn't been created with the
    `+divshot` option.)

        {"name": "your-app",
         "root": "target",
         "clean_urls": true,
         "error_page": "error.html"}
3.  `$ divshot push`

Since Tenzing comes without a backend you can also easily deploy your
app to Amazon S3 or even host it in your Dropbox. To do that just copy
the files in `target/` to your desired location.

> PS. A nice tool to easily deploy to S3 from the command line is
> [stout](https://github.com/EagerIO/Stout).

# License

Copyright © 2014 Martin Klepsch

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.
