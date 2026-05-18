# Chapter 11 - Human Approvals over email
# 第 11 章 - 透過 Email 進行 Human Approvals

in this section, we'll add support for human approvals over email.

在本章節中，我們會加入透過 email 進行 human approval 的支援。

This will start a little bit contrived, just to get the concepts down - 

一開始的設定會稍微有點刻意，目的是先把核心概念建立起來。

We'll start by invoking the workflow from the CLI but approvals for `divide`
and `request_more_information` will be handled over email,
then the final `done_for_now` answer will be printed back to the CLI

我們會先從 CLI 啟動 workflow，但 `divide`
與 `request_more_information` 的 approvals 會透過 email 處理，
接著再把最後的 `done_for_now` 答案輸出回 CLI。

While contrived, this is a great example of the flexibility you get from
[factor 7 - contact humans with tools](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md)

雖然這樣的例子有點刻意，但它很好地展示了
[factor 7 - contact humans with tools](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md) 所帶來的彈性。


for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

Install HumanLayer

安裝 HumanLayer

    npm install humanlayer

Update CLI to send `divide` and `request_more_information` to a human via email

更新 CLI，讓 `divide` 與 `request_more_information` 透過 email 傳送給人類處理。

```diff
src/cli.ts
 // cli.ts lets you invoke the agent loop from the command line
 // cli.ts 讓你可以從命令列啟動 agent loop
 
+import { humanlayer } from "humanlayer";
 import { agentLoop, Thread, Event } from "../src/agent";
 
-
-
 export async function cli() {
     // Get command line arguments, skipping the first two (node and script name)
     // 取得命令列參數，略過前兩個（node 與 script 名稱）
 
     // Run the agent loop with the thread
     // 使用該 thread 執行 agent loop
-    const result = await agentLoop(thread);
-    let lastEvent = result.events.slice(-1)[0];
+    let newThread = await agentLoop(thread);
+    let lastEvent = newThread.events.slice(-1)[0];
 
-    while (lastEvent.data.intent === "request_more_information") {
-        const message = await askHuman(lastEvent.data.message);
-        thread.events.push({ type: "human_response", data: message });
-        const result = await agentLoop(thread);
-        lastEvent = result.events.slice(-1)[0];
+    while (lastEvent.data.intent !== "done_for_now") {
+        const responseEvent = await askHuman(lastEvent);
+        thread.events.push(responseEvent);
+        newThread = await agentLoop(thread);
+        lastEvent = newThread.events.slice(-1)[0];
     }
 
     // print the final result
     // 印出最終結果
     console.log(lastEvent.data.message);
     process.exit(0);
 }
 
-async function askHuman(message: string) {
+async function askHuman(lastEvent: Event): Promise<Event> {
+    if (process.env.HUMANLAYER_API_KEY) {
+        return await askHumanEmail(lastEvent);
+    } else {
+        return await askHumanCLI(lastEvent.data.message);
+    }
+}
+
+async function askHumanCLI(message: string): Promise<Event> {
     const readline = require('readline').createInterface({
         input: process.stdin,
     return new Promise((resolve) => {
         readline.question(`${message}\n> `, (answer: string) => {
-            resolve(answer);
+            resolve({ type: "human_response", data: answer });
         });
     });
 }
+
+export async function askHumanEmail(lastEvent: Event): Promise<Event> {
+    if (!process.env.HUMANLAYER_EMAIL) {
+        throw new Error("missing or invalid parameters: HUMANLAYER_EMAIL");
+    }
+    const hl = humanlayer({ //reads apiKey from env
+        // name of this agent
+        // 這個 agent 的名稱
+        runId: "12fa-cli-agent",
+        verbose: true,
+        contactChannel: {
+            // agent should request permission via email
+            // agent 應透過 email 請求許可
+            email: {
+                address: process.env.HUMANLAYER_EMAIL,
+            }
+        }
+    }) 
+
+    if (lastEvent.data.intent === "divide") {
+        // fetch approval synchronously - this will block until reply
+        // 以同步方式取得 approval - 這會阻塞直到收到回覆
+        const response = await hl.fetchHumanApproval({
+            spec: {
+                fn: "divide",
+                kwargs: {
+                    a: lastEvent.data.a,
+                    b: lastEvent.data.b
+                }
+            }
+        })
+
+        if (response.approved) {
+            const result = lastEvent.data.a / lastEvent.data.b;
+            console.log("tool_response", result);
+            return {
+                "type": "tool_response",
+                "data": result
+            };
+        } else {
+            return {
+                "type": "tool_response",
+                "data": `user denied operation ${lastEvent.data.intent}
+                with feedback: ${response.comment}`
+            };
+        }
+    }
+    throw new Error(`unknown tool: ${lastEvent.data.intent}`)
+}
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/11-cli.ts src/cli.ts

</details>

Run the CLI

執行 CLI

    npx tsx src/index.ts 'can you divide 4 by 5'

The last line of your program should mention human review step

程式最後一行應該會提到 human review 步驟。

    nextStep { intent: 'divide', a: 4, b: 5 }
HumanLayer: Requested human approval from HumanLayer cloud

go ahead and respond to the email with some feedback:

接著請回覆該 email，提供一些 feedback：

![reject-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-reject.png?raw=true)


you should get another email with an updated attempt based on your feedback!

你應該會收到另一封 email，其中包含依據你 feedback 更新後的新嘗試！

You can go ahead and approve this one:

你可以直接核准這一次：

![approve-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-approve.png?raw=true)


and your final output will look like

你的最終輸出會長得像這樣：

    nextStep {
 intent: 'done_for_now',
 message: 'The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!'
}
The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!

lets implement the `request_more_information` flow as well

讓我們也實作 `request_more_information` flow。


```diff
src/cli.ts
     }) 
 
+    if (lastEvent.data.intent === "request_more_information") {
+        // fetch response synchronously - this will block until reply
+        // 以同步方式取得回應 - 這會阻塞直到收到回覆
+        const response = await hl.fetchHumanResponse({
+            spec: {
+                msg: lastEvent.data.message
+            }
+        })
+        return {
+            "type": "tool_response",
+            "data": response
+        }
+    }
+    
     if (lastEvent.data.intent === "divide") {
         // fetch approval synchronously - this will block until reply
         // 以同步方式取得 approval - 這會阻塞直到收到回覆
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/11b-cli.ts src/cli.ts

</details>

lets test the require_approval flow as by asking for a calculation
with garbled input:

讓我們透過輸入含有亂碼的計算要求，來測試 `require_approval` flow：


    npx tsx src/index.ts 'can you multiply 4 and xyz'

You should get an email with a request for clarification

你應該會收到一封請求澄清的 email。

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

you can response with something like

你可以回覆類似以下內容：

    use 8 instead of xyz

you should see a final result on the CLI like

你應該會在 CLI 上看到類似以下的最終結果：

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

as a final step, lets explore using a custom html template for the email

作為最後一步，讓我們試著為 email 使用自訂的 html template。


```diff
src/cli.ts
             email: {
                 address: process.env.HUMANLAYER_EMAIL,
+                // custom email body - jinja
+                // 自訂 email 內容 - jinja
+                template: `{% if type == 'request_more_information' %}
+{{ event.spec.msg }}
+{% else %}
+agent {{ event.run_id }} is requesting approval for {{event.spec.fn}}
+with args: {{event.spec.kwargs}}
+<br><br>
+reply to this email to approve
+{% endif %}`
             }
         }
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/11c-cli.ts src/cli.ts

</details>

first try with divide:

先從 `divide` 開始試試看：


    npx tsx src/index.ts 'can you divide 4 by 5'

you should see a slightly different email with the custom template

你應該會看到一封使用自訂 template、略有不同的 email。

![custom-template-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

feel free to run with the flow and then you can try updating the template to your liking

你可以實際跑完整個 flow，然後再試著把 template 改成你喜歡的樣式。

(if you're using cursor, something as simple as highlighting the template and asking to "make it better"
should do the trick)

（如果你使用的是 cursor，只要反白 template 並要求它「make it better」，
通常就能達到效果。）

try triggering "request_more_information" as well!

也試著觸發 `request_more_information` 吧！


thats it - in the next chapter, we'll build a fully email-driven 
workflow agent that uses webhooks for human approval 

以上就是本章內容——在下一章中，我們會建立一個完全由 email 驅動、
並使用 webhook 進行 human approval 的 workflow agent。


