[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 8. Own your control flow
### 8. 掌握你自己的 control flow

If you own your control flow, you can do lots of fun things.

如果你掌握了自己的 control flow，就能做很多很有意思的事。

![180-control-flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/180-control-flow.png)


Build your own control structures that make sense for your specific use case. Specifically, certain types of tool calls may be reason to break out of the loop and wait for a response from a human or another long-running task like a training pipeline. You may also want to incorporate custom implementation of:

建立真正適合你特定 use case 的 control structures。具體來說，某些類型的 tool calls 可能就是你跳出迴圈、等待人類回覆或等待像 training pipeline 這種長時間任務完成的理由。你可能也會想納入以下自訂實作：

- summarization or caching of tool call results
- tool call 結果的 summarization 或 caching
- LLM-as-judge on structured output
- 針對 structured output 的 LLM-as-judge
- context window compaction or other [memory management](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- context window compaction 或其他 [memory management](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- logging, tracing, and metrics
- logging、tracing 與 metrics
- client-side rate limiting
- client-side rate limiting
- durable sleep / pause / "wait for event"
- durable sleep / pause / 「wait for event」



The below example shows three possible control flow patterns:

下面的範例展示了三種可能的 control flow pattern：


- request_clarification: model asked for more info, break the loop and wait for a response from a human
- request_clarification：model 要求更多資訊，因此跳出迴圈並等待人類回覆
- fetch_git_tags: model asked for a list of git tags, fetch the tags, append to context window, and pass straight back to the model
- fetch_git_tags：model 要求取得 git tags 清單，因此抓取 tags、附加到 context window，然後直接回傳給 model
- deploy_backend: model asked to deploy a backend, this is a high-stakes thing, so break the loop and wait for human approval
- deploy_backend：model 要求部署 backend，這是高風險操作，因此跳出迴圈並等待人類核准


```python
def handle_next_step(thread: Thread):

  while True:
    next_step = await determine_next_step(thread_to_prompt(thread))
    
    # inlined for clarity - in reality you could put 
    # 為了清楚起見，這裡直接內嵌——實務上你可以把這段放到
    # this in a method, use exceptions for control flow, or whatever you want
    # 一個方法裡、用例外來控制流程，或採用任何你想要的做法
    if next_step.intent == 'request_clarification':
      thread.events.append({
        type: 'request_clarification',
          data: nextStep,
        })

      await send_message_to_human(next_step)
      await db.save_thread(thread)
      # async step - break the loop, we'll get a webhook later
      # async step - 跳出迴圈，之後我們會再收到一個 webhook
      break
    elif next_step.intent == 'fetch_open_issues':
      thread.events.append({
        type: 'fetch_open_issues',
        data: next_step,
      })

      issues = await linear_client.issues()

      thread.events.append({
        type: 'fetch_open_issues_result',
        data: issues,
      })
      # sync step - pass the new context to the LLM to determine the NEXT next step
      # sync step - 把新的 context 傳給 LLM，決定再下一個步驟是什麼
      continue
    elif next_step.intent == 'create_issue':
      thread.events.append({
        type: 'create_issue',
        data: next_step,
      })

      await request_human_approval(next_step)
      await db.save_thread(thread)
      # async step - break the loop, we'll get a webhook later
      # async step - 跳出迴圈，之後我們會再收到一個 webhook
      break
```

This pattern allows you to interrupt and resume your agent's flow as needed, creating more natural conversations and workflows.

這種模式可以讓你在需要時中斷並恢復 agent 的流程，進而建立更自然的對話與 workflows。

**Example** - the number one feature request I have for every AI framework out there is we need to be able to interrupt 
a working agent and resume later, ESPECIALLY between the moment of tool **selection** and the moment of tool **invocation**.

**範例** - 我對所有 AI framework 排名第一的功能請求，就是我們必須能夠中斷一個正在運作的 agent，之後再恢復它，尤其是在 tool **selection** 與 tool **invocation** 之間的那一刻。

Without this level of resumability/granularity, there's no way to review/approve the tool call before it runs, which means

沒有這種程度的 resumability / granularity，你就不可能在 tool call 執行之前先完成審查／核准，這意味著：

1. Pause the task in memory while waiting for the long-running thing to complete (think `while...sleep`) and restart it from the beginning if the process is interrupted
1. 在記憶體中暫停任務，等待長時間執行的工作完成（想像 `while...sleep`），而一旦程序中斷就必須從頭開始
2. Restrict the agent to only low-stakes, low-risk calls like research and summarization
2. 把 agent 限制在 research、summarization 這類低風險、低利害關係的 calls
3. Give the agent access to do bigger, more useful things, and just yolo hope it doesn't screw up
3. 給 agent 更大、更有用的權限，然後只能 yolo 地希望它不要搞砸



You may notice this is closely related to [factor 5 - unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) and [factor 6 - launch/pause/resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md), but can be implemented independently.

你可能會注意到，這和 [factor 5 - unify execution state and business state](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) 以及 [factor 6 - launch/pause/resume with simple APIs](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) 關係密切，但它也可以獨立實作。

[← Contact Humans With Tools](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) | [Compact Errors →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
