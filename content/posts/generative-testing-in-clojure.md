+++
title = 'Generative Testing in Clojure'
date = 2025-09-30
description = 'Using clojure.spec to generate tests dynamically from data specifications.'
tags = ['Clojure']
+++

Since I've been learning [Clojure](https://clojure.org/) for the past few weeks, I decided to try to use [`clojure.spec`](https://clojure.org/guides/spec) library to generate tests. With spec, I essentially just need to define the shape of my data, and with this meta-information the library will dynamically generate all the tests.

Below I have a pure function (no side effects) that takes exactly three arguments: the account-id (Cloudflare account ID), the gateway-id (Cloudflare AI gateway ID), and the AI provider. The goal of this function is to generate the Cloudflare gateway URL for the given provider. It's a straightforward function that just concatenates a few values together.

```clojure
(ns experiments.specs)

(defn build-gateway-url
  "Builds a Cloudflare AI Gateway URL for the specified provider.

  Used to route API requests through Cloudflare gateway
  which provides caching, rate limiting, analytics and evals."
  [account-id gateway-id provider]
  (let [base-url (str "https://gateway.ai.cloudflare.com/v1/" account-id "/" gateway-id)
        suffix (case provider
                 "google"    "/google-ai-studio/v1beta"
                 "anthropic" "/anthropic"
                 "mistral"   "/mistral"
                 "openai"    "/openai"
                 (throw (ex-info "invalid provider" {:provider provider})))]
    (str base-url suffix)))
```

If I were to write unit tests for this function, I would probably write something like this:


```clojure
(ns experiments.specs-test
  (:require [clojure.test :refer [deftest testing is]]
            [experiments.specs :refer [build-gateway-url]]))

(deftest build-gateway-url-test
  (let [account-id "fd2a389677248c3d839524df77a9e73c"
        gateway-id "gw-456"
        base (str "https://gateway.ai.cloudflare.com/v1/" account-id "/" gateway-id)]

    (testing "builds correct URLs for each provider"
      (is (= (str base "/google-ai-studio/v1beta")
             (build-gateway-url account-id gateway-id "google")))
      (is (= (str base "/anthropic")
             (build-gateway-url account-id gateway-id "anthropic")))
      (is (= (str base "/mistral")
             (build-gateway-url account-id gateway-id "mistral")))
      (is (= (str base "/openai")
             (build-gateway-url account-id gateway-id "openai"))))

    (testing "throws exception for invalid provider"
      (is (thrown? clojure.lang.ExceptionInfo
                   (build-gateway-url account-id gateway-id "invalid")))
      (try
        (build-gateway-url account-id gateway-id "aws")
        (catch clojure.lang.ExceptionInfo e
          (is (= "invalid provider" (ex-message e)))
          (is (= {:provider "aws"} (ex-data e))))))))
```

There's nothing special or wrong with the test above, but it's not particularly fun to write either.

## Enter clojure.spec

Writing the specs for the function:

```clojure
(ns experiments.specs
  (:require [clojure.spec.alpha :as s]
            [clojure.spec.gen.alpha :as gen]
            [clojure.spec.test.alpha :as st]))

(s/def ::non-empty-string (s/and string? #(not (clojure.string/blank? %))))
(s/def ::provider #{"google" "anthropic" "mistral" "openai"})
(s/def ::account-id
  (s/with-gen
    (s/and string? #(= 32 (count %)))
    #(gen/return
       (apply str (repeatedly 32 (fn [] (rand-nth "0123456789abcdef")))))))
(s/def ::gateway-id ::non-empty-string)
(s/def ::url-string (s/and ::non-empty-string
                           #(clojure.string/starts-with? % "https://gateway.ai.cloudflare.com/v1")))

(s/fdef build-gateway-url
  :args (s/cat :account-id ::account-id
               :gateway-id ::gateway-id
               :provider ::provider)
  :ret ::url-string
  :fn (fn [{:keys [args ret]}]
        (let [args-vals (vals args)]
          (every? #(clojure.string/includes? ret %) args-vals))))
```

## Testing the Function Using Spec

Now writing the test becomes trivial. I just need to use the function `clojure.spec.test.alpha/check` (abbreviated as `st/check` below), Clojure spec will then generate all the possible combination of inputs for my function.

```clojure
(ns experiments.specs-test
  (:require [clojure.test :refer [deftest is testing]]
            [clojure.spec.test.alpha :as st]
            [experiments.specs :as specs]))

(deftest generative-build-gateway-url-test
  (testing "build-gateway-url with generative tests"
    (let [results (st/check `specs/build-gateway-url {:clojure.spec.test.check/opts {:num-tests 100}})]
      (is (every? #(-> % :clojure.spec.test.check/ret :pass?) results)))))
```

The test above will run 100 scenarios using different combinations of parameters to cover many different cases.

## Adding a New Requirement

Now when I have a new requirement, for example, adding support for the new provider "xai". I just need to add the new provider to the spec:

```clojure
(s/def ::provider #{"google" "anthropic" "mistral" "openai" "xai"}) ;; updated spec
```

When we run the tests now, The output will be something like the following:

```txt
Test Summary
experiments.specs-test 74 ms
  generative-build-gateway-url-test 74 ms

Tested 1 namespaces in 74 ms
Ran 1 assertions, in 1 test functions
1 failures
cider-test-fail-fast: t

Results

experiments.specs-test
1 non-passing tests:

Fail in generative-build-gateway-url-test
build-gateway-url with generative tests

expected: (every?
           (fn* [p1__22116#] (-> p1__22116# :clojure.spec.test.check/ret :pass?))
           results)
  actual: (not
           (every?
            #function[experiments.specs-test/fn--22117/fn--22118]
            ({:failure #error {
           :cause "invalid provider" -> the error.
           :data {:provider "xai"}      -> the value not handled by the function.
           :via
           [{:type clojure.lang.ExceptionInfo
             :message "invalid provider"
             :data {:provider "xai"}
             :at [experiments.specs$build_gateway_url invokeStatic "specs.clj" 41]}]
           :trace
...
```

The message indicates that the error was caused by an invalid provider "xai", which triggered an "invalid provider" error. With this information, I just need to update the function and run the tests again:

```clojure
(ns experiments.specs)

(defn build-gateway-url
  "Builds a Cloudflare AI Gateway URL for the specified provider.

  Used to route API requests through Cloudflare gateway
  which provides caching, rate limiting, analytics and evals."
  [account-id gateway-id provider]
  (let [base-url (str "https://gateway.ai.cloudflare.com/v1/" account-id "/" gateway-id)
        suffix (case provider
                 "google"    "/google-ai-studio/v1beta"
                 "anthropic" "/anthropic"
                 "mistral"   "/mistral"
                 "openai"    "/openai"
                 "xai"       "/xai" ;; New provider
                 (throw (ex-info "invalid provider" {:provider provider})))]
    (str base-url suffix)))
```

Running the tests again, we get:

```txt
Test Summary
experiments.specs-test 8 ms

Tested 1 namespaces in 8 ms
Ran 1 assertions, in 1 test functions
1 passed
cider-test-fail-fast: t
```

Neat!

## What I Don't Like

Sometimes the error messages are hard to read. This seems to be a common complaint in the community, and there are community projects trying to address this issue, such as [Expound](https://github.com/bhb/expound).

## How Should I Use Specs?

It seems that specs are intended to be used in critical parts of the application and at the boundaries of the system where validation is needed.

I think they also make a lot of sense for testing purposes, especially when you want to thoroughly test functions with many possible input combinations without manually writing all the test cases.
