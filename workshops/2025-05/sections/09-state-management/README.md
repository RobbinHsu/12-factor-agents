# Chapter 9 - In-Memory State and Async Clarification
# 第 9 章 - In-Memory State 與非同步 Clarification

Add state management and async clarification support.

加入 state management 與非同步 clarification 支援。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

Add some simple in-memory state management for threads

為 threads 加入一些簡單的 in-memory state management。

    cp ./walkthrough/09-state.ts src/state.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/09-state.ts
import crypto from 'crypto';
import { Thread } from '../src/agent';


// you can replace this with any simple state management,
// 你可以用任何簡單的 state management 來取代這裡，
// e.g. redis, sqlite, postgres, etc
// 例如 redis、sqlite、postgres 等
export class ThreadStore {
    private threads: Map<string, Thread> = new Map();
    
    create(thread: Thread): string {
        const id = crypto.randomUUID();
        this.threads.set(id, thread);
        return id;
    }
    
    get(id: string): Thread | undefined {
        return this.threads.get(id);
    }
    
    update(id: string, thread: Thread): void {
        this.threads.set(id, thread);
    }
}
```

</details>

update the server to use the state management

更新 server 以使用 state management。

* Add thread state management using `ThreadStore`
* 使用 `ThreadStore` 加入 thread state management
* return thread IDs and response URLs from the /thread endpoint
* 從 `/thread` endpoint 回傳 thread ID 與 response URL
* implement GET /thread/:id 
* 實作 `GET /thread/:id`
* implement POST /thread/:id/response
* 實作 `POST /thread/:id/response`



```diff
src/server.ts
 import express from 'express';
 import { Thread, agentLoop } from '../src/agent';
+import { ThreadStore } from '../src/state';
 
 const app = express();
 app.set('json spaces', 2);
 
+const store = new ThreadStore();
+
 // POST /thread - Start new thread
 // POST /thread - 開始新的 thread
 app.post('/thread', async (req, res) => {
         data: req.body.message
     }]);
-    const result = await agentLoop(thread);
-    res.json(result);
+    
+    const threadId = store.create(thread);
+    const newThread = await agentLoop(thread);
+    
+    store.update(threadId, newThread);
+
+    const lastEvent = newThread.events[newThread.events.length - 1];
+    // If we exited the loop, include the response URL so the client can
+    // 如果我們跳出了迴圈，就附上 response URL，讓 client 可以
+    // push a new message onto the thread
+    // 將新的訊息推送到該 thread
+    lastEvent.data.response_url = `/thread/${threadId}/response`;
+
+    console.log("returning last event from endpoint", lastEvent);
+
+    res.json({ 
+        thread_id: threadId,
+        ...newThread 
+    });
 });
 
 app.get('/thread/:id', (req, res) => {
-    // optional - add state
-    // 可選：加入 state
-    res.status(404).json({ error: "Not implemented yet" });
+    const thread = store.get(req.params.id);
+    if (!thread) {
+        return res.status(404).json({ error: "Thread not found" });
+    }
+    res.json(thread);
 });
 
+// POST /thread/:id/response - Handle clarification response
+// POST /thread/:id/response - 處理 clarification 回應
+app.post('/thread/:id/response', async (req, res) => {
+    let thread = store.get(req.params.id);
+    if (!thread) {
+        return res.status(404).json({ error: "Thread not found" });
+    }
+    
+    thread.events.push({
+        type: "human_response",
+        data: req.body.message
+    });
+    
+    // loop until stop event
+    // 持續迴圈直到遇到停止事件
+    const newThread = await agentLoop(thread);
+    
+    store.update(req.params.id, newThread);
+
+    const lastEvent = newThread.events[newThread.events.length - 1];
+    lastEvent.data.response_url = `/thread/${req.params.id}/response`;
+
+    console.log("returning last event from endpoint", lastEvent);
+    
+    res.json(newThread);
+});
+
 const port = process.env.PORT || 3000;
 app.listen(port, () => {
```

<details>
<summary>skip this step / 略過此步驟</summary>

    cp ./walkthrough/09-server.ts src/server.ts

</details>

Start the server

啟動 server

    npx tsx src/server.ts

Test clarification flow

測試 clarification flow

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'

