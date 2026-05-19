[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)
[← 返回 README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 6. Launch/Pause/Resume with simple APIs
### 6. 以簡單 API 啟動／暫停／恢復

Agents are just programs, and we have things we expect from how to launch, query, resume, and stop them.

Agents 本質上就是程式，而我們自然會期待它們在啟動、查詢、恢復與停止方面，都具備清楚一致的操作方式。

[![pause-resume animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)](https://github.com/user-attachments/assets/feb1a425-cb96-4009-a133-8bd29480f21f)

<details>
<summary><a href="https://github.com/RobbinHsu/12-factor-agents/blob/main/img/165-pause-resume-animation.gif">GIF Version</a></summary>

![pause-resume animation](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)

</details>

It should be easy for users, apps, pipelines, and other agents to launch an agent with a simple API.

對 users、apps、pipelines 與其他 agents 而言，應該都能透過簡單的 API 輕鬆啟動一個 agent。

Agents and their orchestrating deterministic code should be able to pause an agent when a long-running operation is needed.

當需要執行長時間運作的操作時，agents 與負責 orchestration 的 deterministic code 應該能夠暫停 agent。

External triggers like webhooks should enable agents to resume from where they left off without deep integration with the agent orchestrator.

像 webhooks 這類 external triggers，應該要能讓 agents 從中斷處繼續執行，而不需要和 agent orchestrator 做很深的整合。

Closely related to [factor 5 - unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) and [factor 8 - own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md), but can be implemented independently.

這和 [factor 5 - unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) 以及 [factor 8 - own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) 關係密切，但也可以獨立實作。

**Note** - often AI orchestrators will allow for pause and resume, but not between the moment of tool selection and tool execution. See also [factor 7 - contact humans with tool calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) and [factor 11 - trigger from anywhere, meet users where they are](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md).

**注意**——很多 AI orchestrators 雖然支援 pause 與 resume，但往往無法在 tool 選擇完成與 tool 實際執行之間的那個時點暫停。另請參考 [factor 7 - contact humans with tool calls](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) 與 [factor 11 - trigger from anywhere, meet users where they are](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)。

[← Unify Execution State](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) | [Contact Humans With Tools →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
[← 統一 execution state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) | [以 tools 聯繫 humans →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
