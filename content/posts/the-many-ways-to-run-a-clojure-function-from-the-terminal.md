+++
title = 'The Many Ways to Run a Clojure Function from the Terminal'
date = 2025-12-19
description = 'Different approaches to execute Clojure functions from the command line.'
tags = ['Clojure']
+++

Let's say we have a simple project with the following structure:

```shell
.
├── deps.edn
└── src
    └── example
        └── core.clj
```

Our `deps.edn` file contains:

```clojure
{:paths ["src"]
 :deps {org.clojure/clojure {:mvn/version "1.12.0"}}}
```

And `core.clj` defines a simple `-main` function:

```clojure
(ns example.core)

(defn -main [& args]
  (println args))
```

There are several ways to run this function from the terminal. Let's explore each one.

## Using `-M` with `-m`

The simplest option is to combine the `-M` and `-m` flags:

```shell
$ clj -M -m example.core 1 2 3
;; => (1 2 3)
```

The `-M` flag tells the Clojure CLI to run in `clojure.main` mode, which gives us access to the `-m` flag. The `-m` flag loads the specified namespace and executes its `-main` function, passing any additional arguments as strings.

## Using `-X`

Another option is `-X`, though it requires changing how your function receives arguments. Unlike `-M`, which passes strings directly, `-X` always passes a single map:

```shell
$ clj -X example.core/-main :args '[1 2 3]'
;; => {:args [1 2 3]}
```

This means your function needs to destructure its arguments from that map:

```clojure
(defn -main [{:keys [args]}]
  (println args))  ;; => [1 2 3]
```

This approach is more verbose for simple scripts, but becomes useful when defining aliases with default arguments.

## Using `deps.edn` Aliases

Rather than typing long commands each time, we can define aliases in `deps.edn`.

For the `-M` approach:

```clojure
{:paths ["src"]
 :deps {org.clojure/clojure {:mvn/version "1.12.0"}}
 :aliases
 {:run {:main-opts ["-m" "example.core"]}}}
```

Now we can simply run:

```shell
$ clj -M:run
$ clj -M:run arg1 arg2  # additional arguments are passed through
```

For the `-X` approach:

```clojure
{:paths ["src"]
 :deps {org.clojure/clojure {:mvn/version "1.12.0"}}
 :aliases
 {:run {:exec-fn example.core/-main
        :exec-args {:args [1 2 3]}}}}
```

```shell
$ clj -X:run                   # uses default :args [1 2 3]
$ clj -X:run :args '[4 5 6]'   # overrides with [4 5 6]
```

## Using `-e` for Inline Evaluation

Lastly, the `-e` flag lets us evaluate any Clojure expression directly. We can require a namespace and call a function in one go:

```shell
$ clj -M -e "(require 'example.core) (example.core/-main 1 2 3)"
;; => (1 2 3)
```

A cleaner alternative is `requiring-resolve`, which combines the require and lookup into a single step:

```shell
$ clj -M -e "((requiring-resolve 'example.core/-main) 1 2 3)"
;; => (1 2 3)
```

This is handy for quick one-off calls without modifying any files.

---

Note that `-main` is just an example throughout this article, these techniques work with any function in your codebase.
