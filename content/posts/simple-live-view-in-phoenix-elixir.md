+++
title = 'Simple Live View in Phoenix Elixir'
date = 2025-10-21
description = 'Phoenix LiveView lets you build interactive interfaces without writing JavaScript.'
tags = ['Elixir']
+++

[Phoenix](https://www.phoenixframework.org/) is a web framework written in Elixir. It's great for solo developers or small teams.

[Phoenix LiveView](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html) lets you build interactive interfaces without writing JavaScript. Everything runs on the server. This is really useful when you need to ship fast.

The idea is similar to React, but it all happens server side. Phoenix handles the render loop:

- Receive Event
- Change State
- Render State

![Loop phoenix framework](/images/phoenix-loop.png)

As a developer, I just need to fill in the gaps, the framework handles the rest for me:

- Keeping a live WebSocket connection
- Handling the events
- Patching the DOM

As an example, I'll develop a simple application that converts any input text from markdown to HTML as soon as the user types in a text area. Everything is done on the backend side, and not a single line of JavaScript is needed on the frontend. This is quite powerful.

You can see below how the final application will look:

![Example of the application converting markdown to html in a phoenix live view.](/images/phoenix-liveview-example.png)

And the code is straightforward:

```elixir
defmodule DevToolsWeb.MarkdownConverter do
  use DevToolsWeb, :live_view

  def mount(_params, _session, socket) do
    {:ok, assign(socket, markdown_text: "", html_output: "")}
  end

  def handle_event("markdown_changed", %{"markdown" => ""}, socket) do
    {:noreply, assign(socket, markdown_text: "", html_output: "")}
  end

  def handle_event("markdown_changed", %{"markdown" => markdown_text}, socket) do
    html_output = MDEx.to_html!(markdown_text, extension: [shortcodes: true])

    {:noreply,
     assign(socket,
       markdown_text: markdown_text,
       html_output: html_output
     )}
  end

  def render(assigns) do
    ~H"""
    <div class="container mx-auto max-w-6xl px-4 py-8">
      <div class="card bg-base-100 shadow-xl">
        <div class="card-body">
          <.header>
            <.icon name="hero-document-text" class="size-8 inline-block mr-2" />
            Markdown to HTML Converter
            <:subtitle>
              Convert your markdown into HTML in real-time
            </:subtitle>
          </.header>

          <form class="space-y-6">
            <div>
              <.input
                type="textarea"
                label="Markdown Input"
                rows="12"
                name="markdown"
                value={@markdown_text}
                phx-change="markdown_changed"
                placeholder="Enter markdown text...&#10;# Hello World&#10;&#10;This is **bold** and this is *italic*."
                class="textarea textarea-bordered w-full font-mono text-sm"
              />
            </div>

            <div :if={@html_output != ""} class="alert alert-info">
              <.icon name="hero-information-circle" class="size-5" />
              <div class="flex-1">
                <span class="font-semibold">Output Size:</span>
                {String.length(@html_output)} characters
              </div>
            </div>
          </form>
        </div>
      </div>

      <div :if={@html_output != ""} class="card bg-base-100 shadow-xl mt-8">
        <div class="card-body">
          <h3 class="card-title text-lg">
            <.icon name="hero-eye" class="size-6" /> Preview
          </h3>
          <div class="markdown-preview">
            {Phoenix.HTML.raw(@html_output)}
          </div>
        </div>
      </div>

      <div class="card bg-base-100 shadow-xl mt-8">
        <div class="card-body">
          <h3 class="card-title text-lg">
            <.icon name="hero-information-circle" class="size-6" /> What is Markdown?
          </h3>
          <div class="text-sm text-base-content/80 space-y-3">
            <p>
              Markdown is a lightweight markup language that you can use to add formatting elements to plaintext documents.
              Created by John Gruber in 2004, Markdown is now one of the world's most popular markup languages.
            </p>
            <p>
              <strong>Common syntax examples:</strong>
            </p>
            <ul class="list-disc list-inside space-y-1">
              <li><code># Heading 1</code> - Creates a top-level heading</li>
              <li><code>**bold text**</code> - Makes text bold</li>
              <li><code>*italic text*</code> - Makes text italic</li>
              <li><code>[Link](url)</code> - Creates a hyperlink</li>
              <li><code>- List item</code> - Creates an unordered list item</li>
            </ul>
            <p>
              This converter uses the MDEx library, which supports CommonMark and various extensions including emoji shortcodes like :smile:
            </p>
            <p>
              <a
                href="https://www.markdownguide.org/"
                target="_blank"
                rel="noopener noreferrer"
                class="link link-primary"
              >
                Learn more about Markdown →
              </a>
            </p>
          </div>
        </div>
      </div>

      <div class="mt-8 text-sm text-base-content/70 text-center">
        <.icon name="hero-sparkles" class="size-4 inline-block" />
        Powered by MDEx - A fast CommonMark-compliant Markdown parser
      </div>
    </div>
    """
  end
end
```

And add it to your router:

```elixir
scope "/", DevToolsWeb do
    pipe_through(:browser)
    live("/markdown", MarkdownConverter)
end
```
