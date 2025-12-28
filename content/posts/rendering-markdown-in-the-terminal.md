+++
title = 'Rendering Markdown in the Terminal'
date = 2025-11-05
description = 'Add markdown rendering with ANSI escape codes to display formatted text in the terminal.'
tags = ['Clojure', 'AI-Agents', 'Terminal']
category = 'Agent Series'
+++

This is part 3 of a series on building a terminal agent from scratch.

Previous parts:
- Part 1: [Building an Agent from Scratch](/building-an-agent-from-scratch)
- Part 2: [The Conversation Loop](/the-conversation-loop)

The complete source code is available on GitHub: https://github.com/dimiro1/agent-from-scratch/tree/main/03

In this part, we'll add markdown rendering to our terminal agent. LLMs return responses in markdown, so we need a way to display formatted text—bold, italic, headers, code, and more—all using ANSI escape codes.

Let's get started.

## How terminals render colors?

Terminals don't understand HTML or CSS. Instead, they use ANSI escape codes to apply colors and styles to text. These are special character sequences that tell the terminal "make the next text bold" or "color this red."

The basic structure is simple:

```
\033[<code>m<text>\033[0m
```

Where:
- `\033[` is the escape sequence (also written as `\x1b[` or `\e[`)
- `<code>` is a number representing a style or color
- `m` marks the end of the escape sequence
- `<text>` is your actual content
- `\033[0m` resets back to default styling

Here are the most useful codes:

**Text Colors:**
- `31` = Red
- `32` = Green
- `33` = Yellow
- `34` = Blue
- `35` = Magenta
- `36` = Cyan

**Styles:**
- `0` = Reset all formatting
- `1` = Bold
- `2` = Dim
- `3` = Italic
- `4` = Underline

You can combine multiple codes by separating them with semicolons. For example, `\033[1;31m` produces bold red text.

Let's see it in action:

```bash
$ echo -e "\033[1;32mBold Green Text\033[0m"
Bold Green Text  # (but in bold and green!)
```

## Creating an ANSI namespace

Before we dive into markdown rendering, let's create a dedicated namespace for ANSI formatting. This will give us a clean, reusable way to apply colors and styles.

Create `src/termagent/ansi.clj`:

```clojure
(ns termagent.ansi)

(def codes
  {:reset "\033[0m"
   :bold "\033[1m"
   :dim "\033[2m"
   :italic "\033[3m"
   :underline "\033[4m"
   :red "\033[31m"
   :green "\033[32m"
   :yellow "\033[33m"
   :blue "\033[34m"
   :magenta "\033[35m"
   :cyan "\033[36m"
   :white "\033[37m"})

(defn render
  "Renders text with ANSI codes. Takes a vector [style text] or [style1 style2 ... text]
   where styles are keywords and text is a string."
  [v]
  (if (vector? v)
    (let [styles (butlast v)
          text (last v)
          style-codes (apply str (map codes styles))]
      (str style-codes text (:reset codes)))
    v))
```

The `render` function takes a vector where the last element is the text, and everything before it are style keywords. Let's see it in action:

```clojure
(render [:red "Hello"])
;; => "\033[31mHello\033[0m"

(render [:bold :red "Bold Red"])
;; => "\033[1m\033[31mBold Red\033[0m"

(render [:underline :cyan "Underlined Cyan"])
;; => "\033[4m\033[36mUnderlined Cyan\033[0m"
```

If you pass a plain string (not a vector), it returns it unchanged. This makes it easy to conditionally apply styling.

Let's use this to improve our prompts. Update `termagent.core`:

```clojure
(ns termagent.core
  (:require [termagent.openai :as openai]
            [termagent.ansi :as ansi]))

(defn read-user-input []
  (print (ansi/render [:bold :cyan "User: "]))
  (flush)
  (read-line))

(defn -main []
  (loop [history []]
    (let [input (read-user-input)]
      (case input
        "exit" (println "Bye")
        nil? (println "Bye")
        (do
          (let [baseurl "https://api.openai.com/v1"
                api-key (System/getenv "OPENAI_API_KEY")
                model "gpt-4o-mini"
                history (conj history {:role "user" :content input})
                response (openai/generate baseurl api-key model history)
                ai-reply (openai/get-reply response)]
            (println (ansi/render [:bold :green "Assistant:"]))
            (println ai-reply)
            (recur (conj history {:role "assistant" :content ai-reply}))))))))
```

Much better! Now our prompts have color.

```bash
$ clj -M:run
User: Hello
Assistant:
Hello! How can I assist you today?
```

## The markdown renderer

Now let's create the markdown renderer. The approach is simple:

1. Use regex patterns to match markdown syntax
2. Replace matches with text wrapped in ANSI codes
3. Always reset formatting after each element

We're using regex because of its simplicity. You could use a complete markdown parser that handles corner cases and more complex elements, but since the focus here is learning, regex keeps things straightforward.

Create `src/termagent/markdown.clj`:

```clojure
(ns termagent.markdown
  (:require [clojure.string :as str]
            [termagent.ansi :as ansi]))
```

Now we can use the `ansi/render` function we created earlier. This keeps our markdown rendering functions clean and focused on pattern matching.

## Implementing Basic Markdown Elements

Let's start with the simplest markdown elements: bold and italic text.

### Bold Text

Bold text in markdown is surrounded by `**`:

```clojure
(defn render-bold [text]
  (str/replace text
               #"\*\*(.+?)\*\*"
               (fn [[_ content]]
                 (ansi/render [:bold content]))))
```

### Italic Text

Italic text uses `*`:

```clojure
(defn render-italic [text]
  (str/replace text
               #"\*(.+?)\*"
               (fn [[_ content]]
                 (ansi/render [:italic content]))))
```

### Strikethrough

Strikethrough text uses `~~`:

```clojure
(defn render-strikethrough [text]
  (str/replace text
               #"~~(.+?)~~"
               (fn [[_ content]]
                 (ansi/render [:dim content]))))
```

### Inline Code

Inline code uses backticks:

```clojure
(defn render-inline-code [text]
  (str/replace text
               #"`(.+?)`"
               (fn [[_ content]]
                 (ansi/render [:cyan content]))))
```

## Headers

Headers in markdown start with one or more `#` symbols. We'll render them in bold with different colors:

```clojure
(defn render-headers [text]
  (-> text
      (str/replace #"(?m)^###### (.+)$"
                   (fn [[_ content]]
                     (ansi/render [:bold :white content])))
      (str/replace #"(?m)^##### (.+)$"
                   (fn [[_ content]]
                     (ansi/render [:bold :white content])))
      (str/replace #"(?m)^#### (.+)$"
                   (fn [[_ content]]
                     (ansi/render [:bold :yellow content])))
      (str/replace #"(?m)^### (.+)$"
                   (fn [[_ content]]
                     (ansi/render [:bold :yellow content])))
      (str/replace #"(?m)^## (.+)$"
                   (fn [[_ content]]
                     (ansi/render [:bold :cyan content])))
      (str/replace #"(?m)^# (.+)$"
                   (fn [[_ content]]
                     (ansi/render [:bold :magenta content])))))
```

## Images

Markdown images use the format `![alt-text](url)`. We'll render them with a descriptive label:

```clojure
(defn render-images [text]
  (str/replace text
               #"!\[(.+?)\]\((.+?)\)"
               (fn [[_ alt-text url]]
                 (str "Image: " alt-text " -> " (ansi/render [:dim url])))))
```

## Links

Markdown links use the format `[text](url)`. We'll render the text in blue and underlined, followed by the URL in dim:

```clojure
(defn render-links [text]
  (str/replace text
               #"\[(.+?)\]\((.+?)\)"
               (fn [[_ link-text url]]
                 (str (ansi/render [:blue :underline link-text]) " -> " (ansi/render [:dim url])))))
```

## Task Items

Task items are a special type of list item with checkboxes. Checked items use `[x]` or `[X]`, and unchecked items use `[ ]`:

```clojure
(defn render-tasks [text]
  (-> text
      (str/replace #"(?m)^(\s*)[-*+] \[x\] (.+)$"
                   (fn [[_ indent content]]
                     (str indent (ansi/render [:green "[✓]"]) " " content)))
      (str/replace #"(?m)^(\s*)[-*+] \[X\] (.+)$"
                   (fn [[_ indent content]]
                     (str indent (ansi/render [:green "[✓]"]) " " content)))
      (str/replace #"(?m)^(\s*)[-*+] \[ \] (.+)$"
                   (fn [[_ indent content]]
                     (str indent "[ ] " content)))))
```

## Lists

Let's handle both unordered lists (starting with `-`, `*`, or `+`) and ordered lists (starting with numbers):

```clojure
(defn render-lists [text]
  (-> text
      (str/replace #"(?m)^(\s*)[-*+] (.+)$"
                   (fn [[_ indent content]]
                     (str indent (ansi/render [:cyan "• "]) content)))
      (str/replace #"(?m)^(\s*)(\d+)\. (.+)$"
                   (fn [[_ indent number content]]
                     (str indent (ansi/render [:cyan (str number ". ")]) content)))))
```

## Blockquotes

Blockquotes start with `>`:

```clojure
(defn render-blockquotes [text]
  (str/replace text
               #"(?m)^> (.+)$"
               (fn [[_ content]]
                 (ansi/render [:dim (str "| " content)]))))
```

## Horizontal Rules

Horizontal rules are created with `---`, `***`, or `___`:

```clojure
(defn render-horizontal-rules [text]
  (str/replace text
               #"(?m)^(---|\*\*\*|___)$"
               (fn [[_]]
                 (ansi/render [:dim "--------------------------------"]))))
```

## Putting It All Together

Now we need a main function that applies all these transformations:

```clojure
(defn render [text]
  (-> text
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

Order matters! We process block elements first (headers, blockquotes), then tasks before lists (to avoid conflicts), then inline elements last.

## Testing Our Renderer

Let's create a showcase function to demonstrate all our markdown features:

```clojure
(defn showcase
  "Demonstrates all markdown rendering features.

  Run with: clj -M -e \"(require '[termagent.markdown :as md]) (md/showcase)\""
  []
  (let [sample-text "# Markdown Elements Showcase

## Headers
# This is an H1
## This is an H2
### This is an H3
#### This is an H4
##### This is an H5
###### This is an H6

## Text Styling
- **Bold Text**
- *Italic Text*
- ~~Strikethrough~~

## Lists
### Unordered List
- Item 1
- Item 2
  - Subitem 2.1
  - Subitem 2.2

### Ordered List
1. First Item
2. Second Item
   1. Subitem 2.1
   2. Subitem 2.2

## Links
[OpenAI](https://openai.com)

## Images
![Clojure Logo](https://clojure.org/images/clojure-logo-120b.png)

## Tasks
- [x] Completed task
- [ ] Incomplete task
- [X] Another completed task

## Code
Here is some `inline code` example.

## Blockquotes
> This is a blockquote with important information.

## Horizontal Rule
---

End of showcase."]
    (println (render sample-text))))
```

You can test it:

```bash
$ clj -M -e "(require '[termagent.markdown :as md]) (md/showcase)"
```

![Markdown rendering](/images/markdown-rendering.png)

## Integrating with Our Agent

Now let's integrate this into our conversation loop. Update `termagent.core`:

```clojure
(ns termagent.core
  (:require [termagent.openai :as openai]
            [termagent.markdown :as md]
            [termagent.ansi :as ansi]))

(defn read-user-input []
  (print (ansi/render [:bold :cyan "User: "]))
  (flush)
  (read-line))

(defn -main []
  (loop [history []]
    (let [input (read-user-input)]
      (case input
        "exit" (println "Bye")
        nil? (println "Bye")
        (do
          (let [baseurl "https://api.openai.com/v1"
                api-key (System/getenv "OPENAI_API_KEY")
                model "gpt-4o-mini"
                history (conj history {:role "user" :content input})
                response (openai/generate baseurl api-key model history)
                ai-reply (openai/get-reply response)]
            (println (ansi/render [:bold :green "Assistant:"]))
            (println (md/render ai-reply))
            (recur (conj history {:role "assistant" :content ai-reply}))))))))
```

The changes:
1. Require the markdown and ANSI namespaces
2. Use ANSI styling for prompts
3. Wrap the AI's reply with `md/render`

Test it:

```bash
$ clj -M:run
User: Explain what Clojure is
Assistant:
# Clojure

Clojure is a **modern, functional programming language** that runs on the `Java Virtual Machine` (JVM).

It was created by *Rich Hickey* in 2007.

## Key Features

- Immutable data structures
- First-class functions
- Lisp syntax

> Clojure encourages a functional programming style.

Learn more at [clojure.org](https://clojure.org).
```

## What We've Built

You now have:
- ANSI escape codes for terminal styling
- A reusable ANSI namespace
- A markdown renderer that supports:
  - Headers (H1-H6)
  - Text styling (bold, italic, strikethrough)
  - Links and images
  - Task items
  - Lists
  - Blockquotes
  - Inline code
  - Horizontal rules
- Markdown rendering integrated into your agent
- Colored prompts

## What's Next

Our renderer handles most common markdown elements, but we're still missing one: code blocks with syntax highlighting. In the next part, we'll add syntax highlighting for different languages using regex patterns.

Hope to see you there!
