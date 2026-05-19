[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)
[← 返回 README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 5. Unify execution state and business state
### 5. 統一 execution state 與 business state

Even outside the AI world, many infrastructure systems try to separate "execution state" from "business state". For AI apps, this might involve complex abstractions to track things like current step, next step, waiting status, retry counts, etc. This separation creates complexity that may be worthwhile, but may be overkill for your use case. 

即使在 AI 之外的世界，許多 infrastructure systems 也會嘗試把「execution state」與「business state」分開。對 AI apps 而言，這通常意味著要用複雜的 abstractions 來追蹤 current step、next step、waiting status、retry counts 等資訊。這種分離確實可能有價值，但對你的 use case 來說，也可能是過度設計。

As always, it's up to you to decide what's right for your application. But don't think you *have* to manage them separately.

一如既往，什麼做法最適合你的 application，還是得由你自己決定。但不要以為你*一定*得把它們分開管理。

More clearly:

說得更清楚一些：

- **Execution state**: current step, next step, waiting status, retry counts, etc. 
- **Business state**: What's happened in the agent workflow so far (e.g. list of OpenAI messages, list of tool calls and results, etc.)

- **Execution state**：current step、next step、waiting status、retry counts 等資訊
- **Business state**：到目前為止 agent workflow 中發生了什麼（例如 OpenAI messages 清單、tool calls 與 results 清單等）

If possible, SIMPLIFY - unify these as much as possible. 

如果可以，請盡量 SIMPLIFY——能統一多少就統一多少。

[![155-unify-state](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/155-unify-state-animation.gif)](https://github.com/user-attachments/assets/e5a851db-f58f-43d8-8b0c-1926c99fc68d)

<details>
<summary><a href="https://github.com/RobbinHsu/12-factor-agents/blob/main/img/155-unify-state-animation.gif">GIF Version</a></summary>

![155-unify-state](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/155-unify-state-animation.gif)

</details>

In reality, you can engineer your application so that you can infer all execution state from the context window. In many cases, execution state (current step, waiting status, etc.) is just metadata about what has happened so far.

實際上，你可以把 application 設計成：所有 execution state 都能從 context window 中推導出來。在許多情況下，execution state（current step、waiting status 等）其實只是「到目前為止發生了什麼」的 metadata。

You may have things that can't go in the context window, like session ids, password contexts, etc, but your goal should be to minimize those things. By embracing [factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) you can control what actually goes into the LLM 

你可能仍然會有一些無法放進 context window 的東西，例如 session ids、password contexts 等，但你的目標應該是把這類資訊降到最低。透過採用 [factor 3](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)，你就能掌控真正進入 LLM 的內容。

This approach has several benefits:

這種做法有幾個好處：

1. **Simplicity**: One source of truth for all state
2. **Serialization**: The thread is trivially serializable/deserializable
3. **Debugging**: The entire history is visible in one place
4. **Flexibility**: Easy to add new state by just adding new event types
5. **Recovery**: Can resume from any point by just loading the thread
6. **Forking**: Can fork the thread at any point by copying some subset of the thread into a new context / state ID
7. **Human Interfaces and Observability**: Trivial to convert a thread into a human-readable markdown or a rich Web app UI

1. **Simplicity**：所有 state 都有單一可信來源
2. **Serialization**：thread 可以非常輕鬆地序列化／反序列化
3. **Debugging**：完整歷史都集中在同一個地方，方便檢查
4. **Flexibility**：只要新增新的 event types，就能輕鬆擴充 state
5. **Recovery**：只要重新載入 thread，就能從任何節點恢復
6. **Forking**：只要複製 thread 的某個子集合到新的 context／state ID，就能在任意節點分叉
7. **Human Interfaces and Observability**：可以很輕鬆地把 thread 轉成人類可讀的 markdown，或更豐富的 Web app UI

[← Tools Are Structured Outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) | [Launch/Pause/Resume →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
[← tools 是結構化輸出](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) | [啟動／暫停／恢復 →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
