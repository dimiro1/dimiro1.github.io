+++
title = 'Building an Agent from Scratch'
date = 2025-11-01
description = 'Start building a terminal coding agent using Clojure with minimal dependencies.'
tags = ['Clojure', 'AI-Agents', 'Terminal']
category = 'Agent Series'
+++

There's currently an explosion of coding agents available. Some are deeply integrated into text editors like GitHub Copilot in VS Code or the flexible ones in the Zed editor. My favorites are the ones I can run directly in my terminal, such as Anthropic's Claude Code and Google's Gemini CLI. Excellent open-source projects exist too—Crush from Charm is one that comes to mind.

My goal is to create a series of posts where I build a fully functional terminal coding agent completely from scratch. I'll use minimal libraries and focus on teaching how a coding agent actually works. This isn't complex software, and I want to demystify that.

## Why Clojure?

I'm using Clojure because its minimal syntax keeps us focused on how agents work, not language mechanics. The concepts transfer directly to any language.

## What I'll Build

In this first tutorial, I'll start simple: make a single call to an LLM. Remember, programming is a step-by-step process. I build the simplest thing that works, then iterate and improve.

I'll use an OpenAI compatible API since most providers offer a compatibility layer, making this a practical choice for a general-purpose agent.

Let's get started.

## Prerequisites

I'll assume you have Clojure set up on your computer. Installing Clojure is out of scope here since the process varies by operating system. I recommend visiting the official website and following the instructions for your system:

https://clojure.org/guides/install_clojure

You can verify your Clojure installation by running this in your terminal:

```sh
$ which clj
```

On my machine, I get:

```sh
/opt/homebrew/bin/clj
```

Great! Now let's create the project.

## Setting Up the Project

I'll create the project directory and configuration file:

```sh
$ mkdir termagent
$ cd termagent
$ touch deps.edn
```

Open the `deps.edn` file and add the following:

```clojure
{:deps {org.clojure/clojure {:mvn/version "1.12.3"}}
 :paths ["src"]
 :aliases {:run {:main-opts ["-m" "termagent.core"]}
           :repl {:main-opts ["-r"]}}}
```

Now create the source directory and main file:

```sh
$ mkdir -p src/termagent
$ touch src/termagent/core.clj
```

Open the `core.clj` file and add the following:

```clojure
(ns termagent.core)

(defn -main []
  (println "Termagent"))
```

With this setup in place, I can run the project to verify everything works:

```sh
$ clj -M:run
```

If everything worked, you should see this in the terminal:

```sh
Termagent
```

Perfect! Now that I have the initial project structure validated, I can implement the AI model call.

## Understanding the OpenAI API

Before I write code, I need to understand what I'm working with. I need to know how the API works, which endpoints are available, and what limitations exist.

The official OpenAI SDK documentation is available at: https://platform.openai.com/docs/api-reference/introduction

### Authentication

The first requirement is authentication. Every request to the API needs a token in the `Authorization` header as a Bearer token:

```http
Authorization: Bearer OPENAI_API_KEY
```

You'll need to create an account on OpenAI and generate your API token. This is out of scope for this tutorial, so please follow the instructions on the OpenAI website.

Before continuing, set up your API key as an environment variable. In your terminal:

```sh
export OPENAI_API_KEY=sk-proj-your-key-here
```

You can verify it's set by running:

```sh
echo $OPENAI_API_KEY
```

### The Responses Endpoint

Looking at the available endpoints, `https://api.openai.com/v1/responses` is the simplest one for generating conversation responses:

```sh
curl https://api.openai.com/v1/responses \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-5-mini",
    "input": "Tell me what is great about clojure."
  }'
```

Let me test if my API key works. In my case, this was the response:

```json
{
  "id": "resp_0ddb...",
  "object": "response",
  "created_at": 1762001636,
  "status": "completed",
  "model": "gpt-5-mini-2025-08-07",
  "output": [
    {
      "id": "rs_...",
      "type": "reasoning",
      "summary": []
    },
    {
      "id": "msg_...",
      "type": "message",
      "status": "completed",
      "content": [
        {
          "type": "output_text",
          "text": "Short answer\nClojure is great because..."
        }
      ],
      "role": "assistant"
    }
  ],
  "usage": {
    "input_tokens": 16,
    "output_tokens": 1101,
    "total_tokens": 1117
  }
}
```

I've truncated some IDs for readability, your actual response will have complete identifiers.

### Understanding the Response Structure

That's a lot of data! The good news: we only need to understand a small part of it. Let me walk through what matters for our agent.

Let me focus on the `output` section:

```json
"output": [
  {
    "id": "rs_0ddb3e9e...",
    "type": "reasoning",
    "summary": []
  },
  {
    "id": "msg_0ddb3e...",
    "type": "message",
    "status": "completed",
    "content": [
      {
        "type": "output_text",
        "text": "The ai response..."
      }
    ],
    "role": "assistant"
  }
]
```

The `output` array contains two types here. The `reasoning` type captures the model's internal thought process (used by reasoning models). The `message` type contains the actual response we want. We'll focus on extracting messages for now.

The type tells me which properties are available in the message. For now, I'm interested in the `message` type. You can see all available types and properties in the documentation: https://platform.openai.com/docs/api-reference/responses/object

## Adding Dependencies

Now I need an HTTP client library. Yes, I said minimal dependencies, but this one is worth it, it makes HTTP requests much more natural in Clojure.

I'll use `clj-http` to interact with HTTP endpoints: https://github.com/dakrone/clj-http

I also need a JSON parsing library. I'll use the official Clojure library for that.

Add both dependencies to your `deps.edn` file:

```clojure
{:deps {org.clojure/clojure {:mvn/version "1.12.3"}
        clj-http/clj-http {:mvn/version "3.13.1"}
        org.clojure/data.json {:mvn/version "2.5.1"}}
 :paths ["src"]
 :aliases {:run {:main-opts ["-m" "termagent.core"]}
           :repl {:main-opts ["-r"]}}}
```

Now I can make HTTP requests using idiomatic Clojure:

```clojure
(ns example
  (:require [clj-http.client :as http]))

(http/get "http://example.com/resource" {:accept :json})
```

## Implementing the OpenAI Client

Let me create a basic client that can interact with the API. Create a new file:

```sh
$ touch src/termagent/openai.clj
```

Now I'll write a function that makes the HTTP request:

```clojure
(ns termagent.openai
  (:require [clj-http.client :as http]
            [clojure.data.json :as json]))

(defn generate [base-endpoint api-key model message]
  (let [url (str base-endpoint "/responses")                    ; (1)
        payload {:model model
                 :input message}                                ; (2)
        request {:url url
                 :method :post
                 :headers {"Authorization" (str "Bearer " api-key)
                           "Content-Type" "application/json"}
                 :accept "application/json"
                 :body (json/write-str payload)}                ; (3)
        response (http/request request)                         ; (4)
        body (json/read-str (:body response) :key-fn keyword)]  ; (5)
    body))                                                       ; (6)
```

This function is straightforward. I'll expand it in future chapters. Let me walk through what's happening at each step:

1. Build the final URL: `https://api.openai.com/v1/responses`
2. Construct the payload following the OpenAI API specification
3. Build the HTTP request with headers and body
4. Make the actual HTTP request
5. Parse the JSON response body into Clojure data structures
6. Return the parsed response

At this point, your project structure should look like this:

```sh
$ tree
.
├── deps.edn
└── src
    └── termagent
        ├── core.clj
        └── openai.clj

3 directories, 3 files
```

### Testing the Client

I'll update the `-main` function to try out my new client:

```clojure
(ns termagent.core
  (:require [termagent.openai :as openai]))

(defn -main []
  (let [response (openai/generate "https://api.openai.com/v1"
                                  (System/getenv "OPENAI_API_KEY")
                                  "gpt-5-mini"
                                  "What is clojure?")]
    (println (->> response :output))))
```

Run the code:

```sh
$ clj -M:run
```

You should see output similar to this:

```sh
[{:id rs_0adf68c263, :type reasoning, :summary []}
 {:id msg_0adf68c2, :type message, :status completed,
  :content [{:type output_text, :text Clojure is a modern, functional dialect of the Lisp programming language...}],
  :role assistant}]
```

## What You've Built

You now have a working OpenAI client that can:
- Authenticate with the API
- Send prompts to GPT models
- Parse and extract responses

This is the core of any AI agent. In the next post, we'll add a conversation loop that maintains context across multiple turns, letting you have real back-and-forth exchanges with the AI.
