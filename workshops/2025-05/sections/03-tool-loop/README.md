# Chapter 3 - Process Tool Calls in a Loop
# 第 3 章 - 在迴圈中處理 Tool Calls

Now let's add a real agentic loop that can run the tools and get a final answer from the LLM.

現在讓我們加入一個真正的 agentic loop，能夠執行 tools 並從 LLM 取得最終答案。

First, lets update the agent to handle the tool call

首先，讓我們更新 agent 以處理 tool call。


```diff
src/agent.ts
 }
 
-// right now this just runs one turn with the LLM, but
-// 目前這只會讓 LLM 跑一個回合，但
-// we'll update this function to handle all the agent logic
-// 我們會更新這個函式來處理所有 agent 邏輯
-export async function agentLoop(thread: Thread): Promise<AgentResponse> {
-    const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
-    return nextStep;
+
+
+export async function agentLoop(thread: Thread): Promise<string> {
+
+    while (true) {
+        const nextStep = await b.DetermineNextStep(thread.serializeForLLM());
+        console.log("nextStep", nextStep);
+
+        switch (nextStep.intent) {
+            case "done_for_now":
+                // response to human, return the next step object
+                // 回應給人類，回傳 next step 物件
+                return nextStep.message;
+            case "add":
+                thread.events.push({
+                    "type": "tool_call",
+                    "data": nextStep
+                });
+                const result = nextStep.a + nextStep.b;
+                console.log("tool_response", result);
+                thread.events.push({
+                    "type": "tool_response",
+                    "data": result
+                });
+                continue;
+            default:
+                throw new Error(`Unknown intent: ${nextStep.intent}`);
+        }
+    }
 }
 
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/03-agent.ts src/agent.ts

</details>

Now, lets try it out

現在，來試試看。


    npx tsx src/index.ts 'can you add 3 and 4'

you should see the agent call the tool and then return the result

你應該會看到 agent 呼叫 tool，然後回傳結果。

    {
  intent: 'done_for_now',
  message: 'The sum of 3 and 4 is 7.'
}

For the next step, we'll do a more complex calculation, let's turn off the baml logs for more concise output

下一步我們要做更複雜的計算，先把 baml logs 關掉，讓輸出更精簡。

    export BAML_LOG=off

Try a multi-step calculation

試試多步驟計算

    npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'

you'll notice that tools like multiply and divide are not available

你會注意到像 `multiply` 和 `divide` 這類工具目前還不可用。

    npx tsx src/index.ts 'can you multiply 3 and 4'

next, let's add handlers for the rest of the calculator tools

接著，讓我們為其餘的 calculator tools 加上 handlers。


```diff
src/agent.ts
-import { b } from "../baml_client";
+import { AddTool, SubtractTool, DivideTool, MultiplyTool, b } from "../baml_client";
 
-// tool call or a respond to human tool
-// tool call 或回應給人類的 tool
-type AgentResponse = Awaited<ReturnType<typeof b.DetermineNextStep>>;
-
 export interface Event {
     type: string
 }
 
+export type CalculatorTool = AddTool | SubtractTool | MultiplyTool | DivideTool;
 
+export async function handleNextStep(nextStep: CalculatorTool, thread: Thread): Promise<Thread> {
+    let result: number;
+    switch (nextStep.intent) {
+        case "add":
+            result = nextStep.a + nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "subtract":
+            result = nextStep.a - nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "multiply":
+            result = nextStep.a * nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+        case "divide":
+            result = nextStep.a / nextStep.b;
+            console.log("tool_response", result);
+            thread.events.push({
+                "type": "tool_response",
+                "data": result
+            });
+            return thread;
+    }
+}
 
 export async function agentLoop(thread: Thread): Promise<string> {
         console.log("nextStep", nextStep);
 
+        thread.events.push({
+            "type": "tool_call",
+            "data": nextStep
+        });
+
         switch (nextStep.intent) {
             case "done_for_now":
                 return nextStep.message;
             case "add":
-                thread.events.push({
-                    "type": "tool_call",
-                    "data": nextStep
-                });
-                const result = nextStep.a + nextStep.b;
-                console.log("tool_response", result);
-                thread.events.push({
-                    "type": "tool_response",
-                    "data": result
-                });
-                continue;
-            default:
-                throw new Error(`Unknown intent: ${nextStep.intent}`);
+            case "subtract":
+            case "multiply":
+            case "divide":
+                thread = await handleNextStep(nextStep, thread);
         }
     }
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/03b-agent.ts src/agent.ts

</details>

Test subtraction

測試減法

    npx tsx src/index.ts 'can you subtract 3 from 4'

now, let's test the multiplication tool

現在，讓我們測試乘法工具。


    npx tsx src/index.ts 'can you multiply 3 and 4'

finally, let's test a more complex calculation with multiple operations

最後，讓我們測試一個包含多個運算的更複雜計算。


    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

