# Chapter XX - HumanLayer Webhook Integration
# 第 XX 章 - HumanLayer Webhook 整合

the previous sections used the humanlayer SDK in "synchronous mode" - that 
means every time we wait for human approval, we sit in a loop 
polling until the human response if received.

前面的章節使用的是 humanlayer SDK 的「synchronous mode」——這表示
每次我們等待 human approval 時，都會停在一個迴圈中，
持續 polling，直到收到人類回應為止。

That's obviously not ideal, especially for production workloads,
so in this section we'll implement [factor 6 - launch / pause / resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)
by updating the server to end processing after contacting a human, and use webhooks to receive the results. 

這顯然不是理想做法，尤其對 production workloads 來說更是如此，
因此在本章節中，我們會透過更新 server，讓它在聯絡人類後先結束處理，並改用 webhook 接收結果，
以實作 [factor 6 - launch / pause / resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-6-launch-pause-resume.md)。


add code to initialize humanlayer in the server

在 server 中加入初始化 humanlayer 的程式碼。


```diff
src/server.ts
 import { Thread, agentLoop, handleNextStep } from '../src/agent';
 import { ThreadStore } from '../src/state';
+import { humanlayer } from 'humanlayer';
 
 const app = express();
 const store = new ThreadStore();
 
+const getHumanlayer = () => {
+    const HUMANLAYER_EMAIL = process.env.HUMANLAYER_EMAIL;
+    if (!HUMANLAYER_EMAIL) {
+        throw new Error("missing or invalid parameters: HUMANLAYER_EMAIL");
+    }
+
+    const HUMANLAYER_API_KEY = process.env.HUMANLAYER_API_KEY;
+    if (!HUMANLAYER_API_KEY) {
+        throw new Error("missing or invalid parameters: HUMANLAYER_API_KEY");
+    }
+    return humanlayer({
+        runId: `12fa-agent`,
+        contactChannel: {
+            email: { address: HUMANLAYER_EMAIL }
+        }
+    });
+}
+
 // POST /thread - Start new thread
 // POST /thread - 開始新的 thread
 app.post('/thread', async (req, res) => {
     
     // loop until stop event
     // 持續迴圈直到遇到停止事件
-    const newThread = await agentLoop(thread);
+    const result = await agentLoop(thread);
 
-    store.update(req.params.id, newThread);
+    store.update(req.params.id, result);
 
-    lastEvent = newThread.events[newThread.events.length - 1];
+    lastEvent = result.events[result.events.length - 1];
     lastEvent.data.response_url = `/thread/${req.params.id}/response`;
 
     console.log("returning last event from endpoint", lastEvent);
     
-    res.json(newThread);
+    res.json(result);
 });
 
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/12-1-server-init.ts src/server.ts

</details>

next, lets update the /thread endpoint to 

接著，讓我們更新 `/thread` endpoint，使其能夠：
  
1. handle requests asynchronously, returning immediately
1. 以非同步方式處理請求，並立即回傳
2. create a human contact on request_more_information and done_for_now calls
2. 在 `request_more_information` 與 `done_for_now` 呼叫時建立 human contact



Update the server to be able to handle request_clarification responses

更新 server，使其能夠處理 `request_clarification` 回應。

- remove the old /response endpoint and types
- 移除舊的 `/response` endpoint 與相關 types
- update the /thread endpoint to run processing asynchronously, return immediately
- 更新 `/thread` endpoint，使處理流程以非同步方式執行並立即回傳
- send a state.threadId when requesting human responses
- 在請求人類回應時傳送 `state.threadId`
- add a handleHumanResponse function to process the human response
- 新增 `handleHumanResponse` 函式以處理人類回應
- add a /webhook endpoint to handle the webhook response
- 新增 `/webhook` endpoint 以處理 webhook 回應



```diff
src/server.ts
-import express from 'express';
+import express, { Request, Response } from 'express';
 import { Thread, agentLoop, handleNextStep } from '../src/agent';
 import { ThreadStore } from '../src/state';
-import { humanlayer } from 'humanlayer';
+import { humanlayer, V1Beta2HumanContactCompleted } from 'humanlayer';
 
 const app = express();
     });
 }
-
 // POST /thread - Start new thread
 // POST /thread - 開始新的 thread
-app.post('/thread', async (req, res) => {
+app.post('/thread', async (req: Request, res: Response) => {
     const thread = new Thread([{
         type: "user_input",
     }]);
     
-    const threadId = store.create(thread);
-    const newThread = await agentLoop(thread);
-    
-    store.update(threadId, newThread);
+    // run agent loop asynchronously, return immediately
+    // 以非同步方式執行 agent loop，並立即回傳
+    Promise.resolve().then(async () => {
+        const threadId = store.create(thread);
+        const newThread = await agentLoop(thread);
+        
+        store.update(threadId, newThread);
 
-    const lastEvent = newThread.events[newThread.events.length - 1];
-    // If we exited the loop, include the response URL so the client can
-    // 如果我們跳出了迴圈，就附上 response URL，讓 client 可以
-    // push a new message onto the thread
-    // 將新的訊息推送到該 thread
-    lastEvent.data.response_url = `/thread/${threadId}/response`;
+        const lastEvent = newThread.events[newThread.events.length - 1];
 
-    console.log("returning last event from endpoint", lastEvent);
-
-    res.json({ 
-        thread_id: threadId,
-        ...newThread 
+        if (thread.awaitingHumanResponse()) {
+            const hl = getHumanlayer();
+            // create a human contact - returns immediately
+            // 建立 human contact - 會立即回傳
+            hl.createHumanContact({
+                spec: {
+                    msg: lastEvent.data.message,
+                    state: {
+                        thread_id: threadId,
+                    }
+                }
+            });
+        }
     });
+
+    res.json({ status: "processing" });
 });
 
 // GET /thread/:id - Get thread status
 // GET /thread/:id - 取得 thread 狀態
-app.get('/thread/:id', (req, res) => {
+app.get('/thread/:id', (req: Request, res: Response) => {
     const thread = store.get(req.params.id);
     if (!thread) {
 });
 
+type WebhookResponse = V1Beta2HumanContactCompleted;
 
-type ApprovalPayload = {
-    type: "approval";
-    approved: boolean;
-    comment?: string;
-}
+const handleHumanResponse = async (req: Request, res: Response) => {
 
-type ResponsePayload = {
-    type: "response";
-    response: string;
 }
 
-type Payload = ApprovalPayload | ResponsePayload;
+app.post('/webhook', async (req: Request, res: Response) => {
+    console.log("webhook response", req.body);
+    const response = req.body as WebhookResponse;
 
-// POST /thread/:id/response - Handle clarification response
-// POST /thread/:id/response - 處理 clarification 回應
-app.post('/thread/:id/response', async (req, res) => {
-    let thread = store.get(req.params.id);
+    // response is guaranteed to be set on a webhook
+    // 在 webhook 上可保證 response 一定會被設定
+    const humanResponse: string = response.event.status?.response as string;
+
+    const threadId = response.event.spec.state?.thread_id;
+    if (!threadId) {
+        return res.status(400).json({ error: "Thread ID not found" });
+    }
+
+    const thread = store.get(threadId);
     if (!thread) {
         return res.status(404).json({ error: "Thread not found" });
     }
 
-    const body: Payload = req.body;
-
-    let lastEvent = thread.events[thread.events.length - 1];
-
-    if (thread.awaitingHumanResponse() && body.type === 'response') {
-        thread.events.push({
-            type: "human_response",
-            data: body.response
-        });
-    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && !body.approved) {
-        // push feedback onto the thread
-        // 將 feedback 推送到 thread
-        thread.events.push({
-            type: "tool_response",
-            data: `user denied the operation with feedback: "${body.comment}"`
-        });
-    } else if (thread.awaitingHumanApproval() && body.type === 'approval' && body.approved) {
-        // approved, run the tool, pushing results onto the thread
-        // 已核准，執行 tool，並將結果推送到 thread
-        await handleNextStep(lastEvent.data, thread);
-    } else {
-        res.status(400).json({
-            error: "Invalid request: " + body.type,
-            awaitingHumanResponse: thread.awaitingHumanResponse(),
-            awaitingHumanApproval: thread.awaitingHumanApproval()
-        });
-        return;
+    if (!thread.awaitingHumanResponse()) {
+        return res.status(400).json({ error: "Thread is not awaiting human response" });
     }
 
-    
-    // loop until stop event
-    // 持續迴圈直到遇到停止事件
-    const result = await agentLoop(thread);
-
-    store.update(req.params.id, result);
-
-    lastEvent = result.events[result.events.length - 1];
-    lastEvent.data.response_url = `/thread/${req.params.id}/response`;
-
-    console.log("returning last event from endpoint", lastEvent);
-    
-    res.json(result);
 });
 
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/12a-server.ts src/server.ts

</details>

Start the server in another terminal

在另一個 terminal 中啟動 server

    npx tsx src/server.ts

now that the server is running, send a payload to the '/thread' endpoint

現在 server 已經啟動，請傳送一個 payload 到 `/thread` endpoint。


__ do the response step

__ 執行回應步驟

__ now handle approvals for divide

__ 現在處理 `divide` 的 approvals

__ now also handle done_for_now

__ 現在也處理 `done_for_now`

