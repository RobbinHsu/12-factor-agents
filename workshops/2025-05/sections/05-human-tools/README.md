# Chapter 5 - Multiple Human Tools
# 第 5 章 - 多個 Human Tools

In this section, we'll add support for multiple tools that serve to 
contact humans.

在本章節中，我們會加入對多個用於
聯絡人類的 tools 的支援。


for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

first, let's add a tool that can request clarification from a human 

首先，讓我們加入一個可向人類請求澄清的 tool。

this will be different from the "done_for_now" tool,
and can be used to more flexibly handle different types of human interactions
in your agent.

這會與 `done_for_now` tool 不同，
並可用來更彈性地處理 agent 中
不同類型的人類互動。


```diff
baml_src/agent.baml
+// human tools are async requests to a human
+// human tools 是對人類發出的非同步請求
+type HumanTools = ClarificationRequest | DoneForNow
+
+class ClarificationRequest {
+  intent "request_more_information" @description("you can request more information from me")
+  message string
+}
+
 class DoneForNow {
   intent "done_for_now"
-  message string 
+
+  message string @description(#"
+    message to send to the user about the work that was done. 
+  "#)
 }
 
 function DetermineNextStep(
     thread: string 
-) -> CalculatorTools | DoneForNow {
+) -> HumanTools | CalculatorTools {
     client "openai/gpt-4o"
 
 }
 
+
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/05-agent.baml baml_src/agent.baml

</details>

next, let's re-generate the client code

接著，讓我們重新產生 client 程式碼。

NOTE - if you're using the VSCode extension for BAML,
the client will be regenerated automatically when you save the file
in your editor.

注意：如果你使用的是 BAML 的 VSCode extension，
當你在編輯器中儲存檔案時，client 會自動重新產生。


    npx baml-cli generate

now, let's update the agent to use the new tool

現在，讓我們更新 agent 以使用這個新 tool。


```diff
src/agent.ts
 }
 
-export async function agentLoop(thread: Thread): Promise<string> {
+export async function agentLoop(thread: Thread): Promise<Thread> {
 
     while (true) {
         switch (nextStep.intent) {
             case "done_for_now":
-                // response to human, return the next step object
-                // 回應給人類，回傳 next step 物件
-                return nextStep.message;
+            case "request_more_information":
+                // response to human, return the thread
+                // 回應給人類，回傳 thread
+                return thread;
             case "add":
             case "subtract":
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/05-agent.ts src/agent.ts

</details>

next, let's update the CLI to handle clarification requests
by requesting input from the user on the CLI

接著，讓我們更新 CLI，透過在 CLI 上向使用者請求輸入
來處理 clarification requests。


```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line
 // cli.ts 讓你可以從命令列啟動 agent loop
 
-import { agentLoop, Thread, Event } from "./agent";
+import { agentLoop, Thread, Event } from "../src/agent";
 
+
+
 export async function cli() {
     // Get command line arguments, skipping the first two (node and script name)
     // 取得命令列參數，略過前兩個（node 與 script 名稱）
     // Run the agent loop with the thread
     // 使用該 thread 執行 agent loop
     const result = await agentLoop(thread);
-    console.log(result);
+    let lastEvent = result.events.slice(-1)[0];
+
+    while (lastEvent.data.intent === "request_more_information") {
+        const message = await askHuman(lastEvent.data.message);
+        thread.events.push({ type: "human_response", data: message });
+        const result = await agentLoop(thread);
+        lastEvent = result.events.slice(-1)[0];
+    }
+
+    // print the final result
+    // 印出最終結果
+    // optional - you could loop here too
+    // 可選：你也可以在這裡繼續迴圈
+    console.log(lastEvent.data.message);
+    process.exit(0);
 }
+
+async function askHuman(message: string) {
+    const readline = require('readline').createInterface({
+        input: process.stdin,
+        output: process.stdout
+    });
+
+    return new Promise((resolve) => {
+        readline.question(`${message}\n> `, (answer: string) => {
+            resolve(answer);
+        });
+    });
+}
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/05-cli.ts src/cli.ts

</details>

let's try it out

讓我們試試看。


    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

next, let's add a test that checks the agent's ability to handle
a clarification request

接著，讓我們加入一個測試，檢查 agent
處理 clarification request 的能力。


```diff
baml_src/agent.baml
 
 
+
+test MathOperationWithClarification {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+          [{"type":"user_input","data":"can you multiply 3 and feee9ff10"}]
+      "#
+  }
+  @@assert(intent, {{this.intent == "request_more_information"}})
+}
+
+test MathOperationPostClarification {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+        [
+        {"type":"user_input","data":"can you multiply 3 and FD*(#F&& ?"},
+        {"type":"tool_call","data":{"intent":"request_more_information","message":"It seems like there was a typo or mistake in your request. Could you please clarify or provide the correct numbers you would like to multiply?"}},
+        {"type":"human_response","data":"lets try 12 instead"},
+      ]
+      "#
+  }
+  @@assert(intent, {{this.intent == "multiply"}})
+  @@assert(a, {{this.b == 12}})
+  @@assert(b, {{this.a == 3}})
+}
+        
+
+
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/05b-agent.baml baml_src/agent.baml

</details>

and now we can run the tests again

現在我們可以再次執行測試。


    npx baml-cli test

you'll notice the new test passes, but the hello world test fails

你會注意到新的測試通過了，但 hello world 測試失敗了。

This is because the agent's default behavior is to return "done_for_now"

這是因為 agent 的預設行為是回傳 `done_for_now`。


```diff
baml_src/agent.baml
     "#
   }
-  @@assert(intent, {{this.intent == "done_for_now"}})
+  @@assert(intent, {{this.intent == "request_more_information"}})
 }
 
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/05c-agent.baml baml_src/agent.baml

</details>

Verify tests pass

確認測試通過

    npx baml-cli test

