# Chapter 8 - Adding API Endpoints
# 第 8 章 - 加入 API Endpoints

Add an Express server to expose the agent via HTTP.

加入一個 Express server，透過 HTTP 對外提供 agent。

for this section, we'll disable the baml logs. You can optionally enable them if you want to see more details.

在本章節中，我們會停用 baml logs。若你想查看更多細節，也可以自行啟用。

    export BAML_LOG=off

Install Express and types

安裝 Express 與型別套件

    npm install express && npm install --save-dev @types/express supertest

Add the server implementation

加入 server 實作

    cp ./walkthrough/08-server.ts src/server.ts

<details>
<summary>show file / 顯示檔案</summary>

```ts
// ./walkthrough/08-server.ts
import express from 'express';
import { Thread, agentLoop } from '../src/agent';

const app = express();
app.use(express.json());
app.set('json spaces', 2);

// POST /thread - Start new thread
// POST /thread - 開始新的 thread
app.post('/thread', async (req, res) => {
    const thread = new Thread([{
        type: "user_input",
        data: req.body.message
    }]);
    const result = await agentLoop(thread);
    res.json(result);
});

// GET /thread/:id - Get thread status 
// GET /thread/:id - 取得 thread 狀態
app.get('/thread/:id', (req, res) => {
    // optional - add state
    // 可選：加入 state
    res.status(404).json({ error: "Not implemented yet" });
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});

export { app };
```

</details>

Start the server

啟動 server

    npx tsx src/server.ts

Test with curl (in another terminal)

使用 curl 測試（在另一個 terminal 中）

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

You should get an answer from the agent which includes the
agentic trace, ending in a message like: 

你應該會從 agent 收到一個回答，其中包含
agentic trace，最後會出現像這樣的訊息：


    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}

