# Chapter 4 - Add Tests to agent.baml
# 第 4 章 - 為 agent.baml 加入測試

Let's add some tests to our BAML agent.

讓我們為 BAML agent 加入一些測試。

to start, leave the baml logs enabled

一開始請保持 baml logs 為啟用狀態。

    export BAML_LOG=debug

next, let's add some tests to the agent

接著，讓我們為 agent 加入一些測試。

We'll start with a simple test that checks the agent's ability to handle
a basic calculation.

我們會先從一個簡單測試開始，用來檢查 agent
處理基本計算的能力。


```diff
baml_src/agent.baml
     "#
   }
+
+test MathOperation {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      {
+        "type": "user_input",
+        "data": "can you multiply 3 and 4?"
+      }
+    "#
+  }
+}
+
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/04-agent.baml baml_src/agent.baml

</details>

Run the tests

執行測試

    npx baml-cli test

now, let's improve the test with assertions!

現在，讓我們用 assertions 來強化這個測試！

Assertions are a great way to make sure the agent is working as expected,
and can easily be extended to check for more complex behavior.

Assertions 是確保 agent 行為符合預期的絕佳方式，
也能輕鬆擴充來檢查更複雜的行為。


```diff
baml_src/agent.baml
     "#
   }
+  @@assert(hello, {{this.intent == "done_for_now"}})
 }
 
     "#
   }
+  @@assert(math_operation, {{this.intent == "multiply"}})
 }
 
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

</details>

Run the tests

執行測試

    npx baml-cli test

as you add more tests, you can disable the logs to keep the output clean. 
You may want to turn them on as you iterate on specific tests.

隨著你加入更多測試，可以關閉 logs 以保持輸出乾淨。
當你針對特定測試反覆調整時，可能又會想把它們打開。


    export BAML_LOG=off

now, let's add some more complex test cases,
where we resume from in the middle of an in-progress
agentic context window

現在，讓我們加入一些更複雜的測試案例，
模擬從一個進行中的
agentic context window 中途恢復。


```diff
baml_src/agent.baml
     "#
   }
-  @@assert(hello, {{this.intent == "done_for_now"}})
+  @@assert(intent, {{this.intent == "done_for_now"}})
 }
 
     "#
   }
-  @@assert(math_operation, {{this.intent == "multiply"}})
+  @@assert(intent, {{this.intent == "multiply"}})
 }
 
+test LongMath {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      [
+        {
+          "type": "user_input",
+          "data": "can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result?"
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "multiply",
+            "a": 3,
+            "b": 4
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 12
+        },
+        {
+          "type": "tool_call", 
+          "data": {
+            "intent": "divide",
+            "a": 12,
+            "b": 2
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 6
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "add", 
+            "a": 6,
+            "b": 12
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 18
+        }
+      ]
+    "#
+  }
+  @@assert(intent, {{this.intent == "done_for_now"}})
+  @@assert(answer, {{"18" in this.message}})
+}
+
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

</details>

let's try to run it

讓我們試著執行它。


    npx baml-cli test

