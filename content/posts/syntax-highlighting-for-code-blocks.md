+++
title = 'Syntax Highlighting for Code Blocks'
date = 2025-12-25T09:00:00-03:00
draft = false
tags = ['Clojure', 'Terminal', 'AI Agents']
description = 'Part 4 of building a terminal agent from scratch. Adding syntax highlighting for code blocks using regex-based patterns.'
category = 'Agent Series'
+++

This is part 4 of a series on building a terminal agent from scratch.

Previous parts:
- Part 1: [Building an Agent from Scratch](https://dimiro1.dev/building-an-agent-from-scratch)
- Part 2: [The Conversation Loop](https://dimiro1.dev/the-conversation-loop)
- Part 3: [Rendering Markdown in the Terminal](https://dimiro1.dev/rendering-markdown-in-the-terminal)

The complete source code is available on GitHub: [https://github.com/dimiro1/agent-from-scratch/tree/main/04](https://github.com/dimiro1/agent-from-scratch/tree/main/04)

In this part, we'll add syntax highlighting for code blocks. When LLMs return code snippets, they use markdown code blocks with a language hint like `clojure` or `python`. We'll parse these blocks and apply colors to keywords, strings, comments, and numbers.

Let's get started.

## The Approach

Our syntax highlighting strategy is simple:

1. Extract code blocks from the markdown (the part between triple backticks)
2. Identify the language from the code fence (e.g., ```clojure)
3. Apply regex-based highlighting for that language
4. Return the colorized code

We're using regex for highlighting because it's straightforward and keeps our implementation simple. Production syntax highlighters use proper parsers, but regex works well enough for our purposes.

## Creating the Code Namespace

Create `src/termagent/code.clj`:

```clojure
(ns termagent.code
  (:require [clojure.string :as str]
            [termagent.ansi :as ansi]))
```

We'll build individual highlighters for each language, then combine them with a dispatcher function.

## Highlighting Clojure

Let's start with Clojure since that's what we're writing:

```clojure
(defn highlight-clojure [code]
  (-> code
      ;; Strings (must come before keywords to avoid conflicts)
      (str/replace #"\".*?\"" ; (1)
                   (fn [match]
                     (ansi/render [:green match])))
      ;; Comments
      (str/replace #"(?m);.*$" ; (2)
                   (fn [match]
                     (ansi/render [:dim match])))
      ;; Keywords (Clojure :keyword syntax)
      (str/replace #":[a-zA-Z][a-zA-Z0-9\-]*" ; (3)
                   (fn [match]
                     (ansi/render [:magenta match])))
      ;; Numbers
      (str/replace #"\b\d+\.?\d*\b" ; (4)
                   (fn [match]
                     (ansi/render [:yellow match])))
      ;; Common functions
      (str/replace #"\b(println|print|str|map|reduce|filter|conj|assoc|dissoc|get|first|rest|count|into|vec|list|seq|apply|partial|comp)\b" ; (5)
                   (fn [[match]]
                     (ansi/render [:blue match])))
      ;; Language keywords
      (str/replace #"\b(def|defn|defn-|defmacro|let|if|when|cond|fn|loop|recur|do|for|doseq|require|ns|use)\b" ; (6)
                   (fn [[match]]
                     (ansi/render [:blue match])))))
```

Let's break down the patterns:

1. Strings are matched with `\".*?\"`. The `?` makes it non-greedy, so it stops at the first closing quote
2. Comments use `(?m);.*$`. The `(?m)` enables multiline mode so `$` matches end of each line
3. Keywords match the Clojure `:keyword` syntax
4. Numbers match integers and decimals with `\b` word boundaries
5. Common core functions get highlighted in blue
6. Special forms like `def`, `defn`, `let` also get blue

The order matters! We process strings first because they might contain characters that look like keywords or comments.

## Adding More Languages

The pattern for adding new languages is the same: match strings first, then comments, then language-specific elements like keywords and numbers. The repository includes highlighters for Python, JavaScript, Java, Elixir, Lua, and Ruby. Each follows the same structure with language-specific regex patterns.

## The Dispatcher Function

Now we need a function that picks the right highlighter based on the language:

```clojure
(defn highlight
  "Applies syntax highlighting to code based on the language"
  [code language]
  (case (str/lower-case (str/trim language))
    "clojure" (highlight-clojure code)
    "clj" (highlight-clojure code)
    "python" (highlight-python code)
    "py" (highlight-python code)
    "javascript" (highlight-javascript code)
    "js" (highlight-javascript code)
    "java" (highlight-java code)
    "elixir" (highlight-elixir code)
    "ex" (highlight-elixir code)
    "lua" (highlight-lua code)
    "ruby" (highlight-ruby code)
    "rb" (highlight-ruby code)
    (ansi/render [:dim code]))) ; (1)
```

1. For unknown languages, we render the code in dim (the ANSI code that makes text appear faded). This provides a visual distinction from regular text without breaking anything

## Integrating with Markdown

Now we need to update our markdown renderer to handle code blocks. Update `src/termagent/markdown.clj`:

```clojure
(ns termagent.markdown
  (:require [clojure.string :as str]
            [termagent.ansi :as ansi]
            [termagent.code :as code])) ; (1)
```

1. Add the new code namespace

Add the code block renderer:

```clojure
(defn render-code-blocks [text]
  (str/replace text
               #"(?s)```(\w+)?\n(.*?)```" ; (1)
               (fn [[_ language code-content]] ; (2)
                 (let [lang (or language "") ; (3)
                       highlighted (code/highlight code-content lang) ; (4)
                       lines (str/split-lines highlighted)
                       indented (map #(str "  " %) lines)] ; (5)
                   (str "\n" (str/join "\n" indented) "\n")))))
```

1. The regex matches fenced code blocks. `(?s)` enables dotall mode so `.` matches newlines. We capture the optional language identifier and the code content
2. We destructure the match to get the language and code separately
3. If no language is specified, we default to an empty string
4. Apply syntax highlighting
5. Indent each line with two spaces for visual separation

Update the `render` function to process code blocks first:

```clojure
(defn render [text]
  (-> text
      render-code-blocks ; (1)
      render-horizontal-rules
      render-headers
      render-blockquotes
      render-tasks
      render-lists
      render-images
      render-links
      render-bold
      render-strikethrough
      render-italic
      render-inline-code))
```

1. Process code blocks first, before any other transformations. This prevents inline code rendering from interfering with code blocks

## Testing the Highlighter

Let's test the Clojure highlighter:

```bash
$ clj -M -e "(require '[termagent.code :as code]) (println (code/highlight \"(defn greet [name] (println \\\"Hello\\\" name))\" \"clojure\"))"
```

You should see colorized output with:
- Green strings
- Dim comments
- Blue keywords
- Yellow numbers
- Magenta Clojure keywords

## Testing with the Agent

Now let's see it in action with our agent:

```bash
$ clj -M:run
User: Write a hello world function in Clojure
Assistant:
Here's a simple hello world function in Clojure:

  (defn hello-world []
    (println "Hello, World!"))

  (hello-world)

This defines a function called `hello-world` that prints "Hello, World!" to the console.
```

The code block is now syntax highlighted with colors!

## What We've Built

You now have:
- A syntax highlighting system using regex patterns
- Support for multiple languages (Clojure, Python, JavaScript, Java, Elixir, Lua, Ruby)
- A dispatcher that selects the right highlighter based on language
- Code block rendering integrated with markdown

To add more languages, create a `highlight-<language>` function following the same pattern and register it in the `highlight` dispatcher.

## What's Next

Our terminal agent now renders markdown beautifully with syntax-highlighted code blocks. In the next part, we'll clean up the implementation and improve the user experience. After that, we'll add tool support so the agent can execute actions like reading files and running commands.

Hope to see you there!
