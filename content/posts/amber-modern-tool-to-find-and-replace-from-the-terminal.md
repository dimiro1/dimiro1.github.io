+++
title = 'Amber – modern tool to find and replace from the terminal'
date = 2025-11-19
description = 'A Rust tool for quick find and replace operations from the terminal.'
tags = ['Terminal']
+++

I was looking for a modern way to quickly find and replace text from the terminal, and I stumbled upon a Rust tool called **amber**. It's quite interesting, especially for quick replacements.

Amber provides two commands: `ambs` for search and `ambr` for replacement (as you can guess from the suffixes).

It's super simple to use:

```sh
ambs keyword                  // Recursively search 'keyword' from the current directory
ambs keyword path             // Recursively search 'keyword' from 'path'
ambr keyword replacement      // Recursively search and replace 'keyword' with 'replacement' interactively
ambr keyword replacement path // Same as above, but starting from 'path'
```

## Replacement in action

```sh
$ ambr 'hello' 'hi' 'internal/'
internal/runner/runner_test.go:         local original = "hello world & test"
                             ->         local original = "hi world & test"
Replace keyword? [Y]es/[n]o/[a]ll/[q]uit:
```

The interactive prompt lets you review each replacement before applying it, giving you fine-grained control.

## Search in action

```sh
$ ambs -r '[Hh]ello'
./frontend/js/components/function-docs.js:                `local hash = crypto.sha256("hello")
./frontend/js/components/function-docs.js:                `local encoded = url.encode("hello world")
```

The `-r` flag enables regex support, making it easy to handle case-insensitive searches or more complex patterns.
