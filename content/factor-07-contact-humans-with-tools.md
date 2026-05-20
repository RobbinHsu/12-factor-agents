[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 7. Contact humans with tool calls
### 7. 透過 tool calls 聯繫人類

By default, LLM APIs rely on a fundamental HIGH-STAKES token choice: Are we returning plaintext content, or are we returning structured data?

預設情況下，LLM APIs 依賴一個根本而且 HIGH-STAKES 的 token 選擇：我們要回傳 plaintext content，還是要回傳 structured data？

![170-contact-humans-with-tools](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)

You're putting a lot of weight on that choice of first token, which, in the `the weather in tokyo` case, is

你把非常多的成敗都壓在第一個 token 的選擇上，而在 `the weather in tokyo` 這個例子裡，那個 token 是：

> "the"
>
> 「the」

but in the `fetch_weather` case, it's some special token to denote the start of a JSON object.

但在 `fetch_weather` 這個情況裡，它會是某個特殊 token，用來表示 JSON 物件的起始。

> |JSON>
>
> |JSON>

You might get better results by having the LLM *always* output json, and then declare it's intent with some natural language tokens like `request_human_input` or `done_for_now` (as opposed to a "proper" tool like `check_weather_in_city`). 

更好的做法可能是讓 LLM *永遠*輸出 json，然後再用一些自然語言 token 來宣告它的 intent，例如 `request_human_input` 或 `done_for_now`（而不是像 `check_weather_in_city` 那樣的「正式」tool）。

Again, you might not get any performance boost from this, but you should experiment, and ensure you're free to try weird stuff to get the best results.

再次提醒，這麼做不一定會帶來任何效能提升，但你應該親自實驗，並確保自己可以自由嘗試各種看似奇怪的方法，以找出最佳結果。

```python

class Options:
  urgency: Literal["low", "medium", "high"]
  format: Literal["free_text", "yes_no", "multiple_choice"]
  choices: List[str]

# Tool definition for human interaction
# 人機互動的工具定義
class RequestHumanInput:
  intent: "request_human_input"
  question: str
  context: str
  options: Options

# Example usage in the agent loop
# agent 迴圈中的使用範例
if nextStep.intent == 'request_human_input':
  thread.events.append({
    type: 'human_input_requested',
    data: nextStep
  })
  thread_id = await save_state(thread)
  await notify_human(nextStep, thread_id)
  return # Break loop and wait for response to come back with thread ID
  # 中斷迴圈並等待帶有 thread ID 的回應返回
else:
  # ... other cases
  # ... 其他情況
```

Later, you might receive a webhook from a system that handles slack, email, sms, or other events.

之後，你可能會從某個處理 slack、email、sms 或其他事件的系統收到 webhook。

```python

@app.post('/webhook')
def webhook(req: Request):
  thread_id = req.body.threadId
  thread = await load_state(thread_id)
  thread.events.push({
    type: 'response_from_human',
    data: req.body
  })
  # ... simplified for brevity, you likely don't want to block the web worker here
  # ... 為了簡潔起見而省略細節，你大概不會想在這裡阻塞 web worker
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread.events.append(next_step)
  result = await handle_next_step(thread, next_step)
  # todo - loop or break or whatever you want
  # todo - 要繼續迴圈、跳出，或做任何你想做的控制都可以

  return {"status": "ok"}
```

The above includes patterns from [factor 5 - unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md), [factor 8 - own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md), [factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md), and [factor 4 - tools are just structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md), and several others.

以上內容包含了 [factor 5 - unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)、[factor 8 - own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)、[factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)、[factor 4 - tools are just structured outputs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) 以及其他幾個要素中的模式。

If we were using the XML-y formatted from [factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md), our context window after a few turns might look like this:

如果我們使用 [factor 3 - own your context window](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) 裡那種偏 XML 風格的格式化方式，那麼在幾個 turns 之後，我們的 context window 可能會長這樣：

```xml

(snipped for brevity)

<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy backend v1.2.3 to production?
    Thread: []
</slack_message>

<request_human_input>
    intent: "request_human_input"
    question: "Would you like to proceed with deploying v1.2.3 to production?"
    context: "This is a production deployment that will affect live users."
    options: {
        urgency: "high"
        format: "yes_no"
    }
</request_human_input>

<human_response>
    response: "yes please proceed"
    approved: true
    timestamp: "2024-03-15T10:30:00Z"
    user: "alex@company.com"
</human_response>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<deploy_backend_result>
    status: "success"
    message: "Deployment v1.2.3 to production completed successfully."
    timestamp: "2024-03-15T10:30:00Z"
</deploy_backend_result>
```


Benefits:

優點：

1. **Clear Instructions**: Tools for different types of human contact allow for more specificity from the LLM
1. **Clear Instructions**：針對不同類型的人類互動建立 tools，可以讓 LLM 給出更明確的指示
2. **Inner vs Outer Loop**: Enables agents workflows **outside** of the traditional chatGPT-style interface, where the control flow and context initialization may be `Agent->Human` rather than `Human->Agent` (think, agents kicked off by a cron or an event)
2. **Inner vs Outer Loop**：讓 agent workflows 能夠發生在傳統 chatGPT 風格介面**之外**，此時 control flow 與 context 初始化可能是 `Agent->Human`，而不是 `Human->Agent`（想像一下由 cron 或事件啟動的 agents）
3. **Multiple Human Access**: Can easily track and coordinate input from different humans through structured events
3. **Multiple Human Access**：可以透過 structured events，輕鬆追蹤並協調來自不同人類的輸入
4. **Multi-Agent**: Simple abstraction can be easily extended to support `Agent->Agent` requests and responses
4. **Multi-Agent**：這個簡單抽象可以很容易延伸，支援 `Agent->Agent` 的請求與回應
5. **Durable**: Combined with [factor 6 - launch/pause/resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md), this makes for durable, reliable, and introspectable multiplayer workflows
5. **Durable**：搭配 [factor 6 - launch/pause/resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) 使用後，就能建立 durable、reliable、且可 introspect 的多人 workflow


[More on Outer Loop Agents over here](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)

更多關於 Outer Loop Agents 的內容請看[這裡](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)。

![175-outer-loop-agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/175-outer-loop-agents.png)

Works great with [factor 11 - trigger from anywhere, meet users where they are](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)

與 [factor 11 - trigger from anywhere, meet users where they are](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md) 搭配效果極佳。

[← Launch/Pause/Resume](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) | [Own Your Control Flow →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
