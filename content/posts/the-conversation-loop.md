+++
title = 'The Conversation Loop'
date = 2025-11-04
description = 'Implement a conversation loop with message history for back-and-forth exchanges with the LLM.'
tags = ['Clojure', 'AI-Agents', 'Terminal']
category = 'Agent Series'
+++

This is part 2 of a series on building a fully functional terminal agent from scratch.

You can find the part 1 of this series here: [Building an Agent from Scratch](/building-an-agent-from-scratch)

The complete source code is available in my GitHub: https://github.com/dimiro1/agent-from-scratch/tree/main/02

In this second part, we'll implement the conversation loop so we can have real back-and-forth conversations with the LLM. The agent will remember our past messages from the same conversation, making interactions feel natural and contextual.

Let's get started.

## Understanding the Conversation Loop

The first thing we need to do is update the main function to maintain a conversation loop. When I say conversation loop, I mean:

1. Get user input
2. Send the user message to the LLM
3. Get the response from the LLM
4. Display the response to the user
5. Go back to step one

All conversational agents follow this loop. Some include complex cases like sub-conversations, but the basic pattern remains the same.

Let's modify the main file to implement the most basic loop. We'll expand from there.

## Implementing the Basic Loop

```clojure
(ns termagent.core
  (:require [termagent.openai :as openai]))

(defn -main []
  (loop [history []] ; (1)
    (let [input (read-line)] ; (2)
      (case input
        "exit" history ; (3)
        nil? history
        (do ; (4)
          (println (str "input " input))
          (println (str "history " history))
          (recur (conj history input))))))) ; (5)
```

Let's go step by step through the changes we've made:

1. We start the conversation loop using the `loop` function: [https://clojuredocs.org/clojure.core/loop](https://clojuredocs.org/clojure.core/loop)
2. We grab the user input and store it in a variable called `input`
3. We define the stop conditions—the loop stops if the user types `exit` or uses `Control-C` or `Control-D`
4. This is the else case of the `case` function. If nothing matches above, this code runs. We wrap it in a `do` function because we need to run multiple instructions. Currently, we're just printing the user message and all known messages so far, then restarting the loop using `recur` with the new set of messages

Let's run the program to test the implementation:

```sh
$ clj -M:run
```

```sh
Hello
input Hello
history []
this is my input
input this is my input
history ["Hello"]
this is another input
input this is another input
history ["Hello" "this is my input"]
exit
```

As you can see, we're able to collect user input, maintain a conversation history, and print that history to the terminal. Progress!

That's basically all we need to implement the loop. We're still missing the AI integration, but we'll work on that now.

## Integrating the LLM

Now let's integrate the conversation with the LLM. We need to make a few changes to the code. First, we'll create a new function called `get-reply` in the openai namespace:

```clojure
(defn get-reply [response]
  (->> (get response :output) ; (1)
       (filter #(= (get % :role) "assistant")) ; (2)
       first
       :content ; (3)
       first
       :text)) ; (4)
```

We'll use this function to extract only the text reply from the OpenAI response:

1. First, we get the `:output` from the response. You can always refer to the official documentation for response examples: [https://platform.openai.com/docs/api-reference/responses/create?api-mode=responses](https://platform.openai.com/docs/api-reference/responses/create?api-mode=responses)
2. We filter only the messages from the assistant
3. We take the first one and extract the `:content` map from the response
4. We extract the `:text` part of the response

Now let's make the necessary changes to the `-main` function:

```clojure
(defn -main []
  (loop [history []]
    (let [input (read-line)]
      (case input
        "exit" (println "Bye")
        nil? (println "Bye")
        (do
          (let [baseurl "https://api.openai.com/v1"
                api-key (System/getenv "OPENAI_API_KEY")
                model "gpt-4o-mini"
                response (openai/generate baseurl api-key model input) ; (1)
                ai-reply (openai/get-reply response) ; (2)
                history (conj history input ai-reply)] ; (3)
            (println ai-reply) ; (4)
            (recur history)))))))
```

It's not too different from what we had before. Hopefully the implementation is straightforward. We're declaring some variables to hold values and then calling the AI:

1. We call the AI model passing the user input
2. We use the function we just defined to extract the text response from the model
3. We update the history with both the user message and the AI's reply
4. We print the message to the terminal

Let's try this implementation:

```sh
$ clj -M:run
```

```sh
Hello
Hello! How can I assist you today?
My name is Claudemiro
Nice to meet you, Claudemiro! How can I assist you today?
What is my name?
I don't know your name. If you'd like to share it, feel free!
exit
Bye
```

## Adding Conversation Memory

Something is wrong. How come the model doesn't know my name? I just told it. The reason is that AI models don't keep any state—they can't hold a conversation on their own. That's why we're maintaining a history of messages (though we're not using it yet). For each call to the LLM, we need to pass the entire conversation history.

The official documentation explains that we need to pass the input as an array of key-value pairs, each containing a `role` and a `content` key: [https://platform.openai.com/docs/guides/conversation-state#manually-manage-conversation-state](https://platform.openai.com/docs/guides/conversation-state#manually-manage-conversation-state)

```clojure
[
  {:role "user" :content "knock knock."}
  {:role "assistant" :content "Who's there?"}
  {:role "user" :content "Orange."}
]
```

Great! This means we don't need to make many big changes to our implementation:

```clojure
(defn -main []
  (loop [history []]
    (let [input (read-line)]
      (case input
        "exit" (println "Bye")
        nil? (println "Bye")
        (do
          (let [baseurl "https://api.openai.com/v1"
                api-key (System/getenv "OPENAI_API_KEY")
                model "gpt-4o-mini"
                history (conj history {:role "user" :content input}) ; (1)
                response (openai/generate baseurl api-key model history) ; (2)
                ai-reply (openai/get-reply response)]
            (println ai-reply)
            (recur (conj history {:role "assistant" :content ai-reply})))))))) ; (3)
```

The changes are:

1. We append the user input to the history
2. We call the LLM with the full history
3. We append the LLM response to the history

That's essentially all we need to implement. Let's try it out:

```sh
$ clj -M:run
```

```sh
Hello
Hello! How can I assist you today?
My name is Claudemiro
Nice to meet you, Claudemiro! How can I help you today?
What is my name?
Your name is Claudemiro.
Nice, thanks
You're welcome! If you have any other questions or need assistance, feel free to ask.
exit
Bye
```

Perfect! Now the LLM can refer to previous messages.

## Improving the UI

Before we finish this chapter, let's improve the UI a bit. Currently, it's hard to distinguish user messages from assistant messages.

As always, we'll start simple and improve later:

```clojure
(defn read-user-input [] ; (1)
  (print "User: ")
  (flush)
  (read-line))

(defn -main []
  (loop [history []]
    (let [input (read-user-input)] ; (2)
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
            (println "Assistant:" ai-reply) ; (3)
            (recur (conj history {:role "assistant" :content ai-reply}))))))))
```

Here's what changed:

1. We created a new function that first prints a `User:` label, then reads the user input. The `flush` is needed—otherwise the message won't print right away. By default, output is buffered, but we want it to display immediately
2. We call the new function
3. We prepend the AI response with the `Assistant:` label

Let's run this to see what we have:

```sh
$ clj -M:run
```

```sh
User: Hello
Assistant: Hello! How can I assist you today?
User: Who created Clojure?
Assistant: Clojure was created by Rich Hickey. He first released it in 2007, and it is a functional programming language that runs on the Java Virtual Machine (JVM). It emphasizes immutability and functional programming principles. Would you like to know more about Clojure or its features?
User: exit
Bye
```

Much better! We now have a fully functional yet simple assistant that can hold a conversation.

## What You've Built

You now have a conversational agent that can:
- Maintain a conversation loop with proper user input handling
- Keep conversation history across multiple turns
- Send the full context to the LLM with each request
- Display clear distinctions between user and assistant messages

In the next chapters, we'll clean up the implementation by creating some helper functions. The biggest topics will be markdown rendering and syntax highlighting in code blocks.

Hope to see you there.
