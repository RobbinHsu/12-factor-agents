[← Back to README](https://github.com/RobbinHsu/12-factor-agents/blob/main/README.md)

### 9. Compact Errors into Context Window
### 9. 將錯誤壓縮進 Context Window

This one is a little short but is worth mentioning. One of these benefits of agents is "self-healing" - for short tasks, an LLM might call a tool that fails. Good LLMs have a fairly good chance of reading an error message or stack trace and figuring out what to change in a subsequent tool call.

這一節稍微短一些，但值得一提。agents 的其中一個好處是「self-healing」——對於短任務來說，LLM 可能會呼叫一個失敗的 tool。好的 LLM 有相當高的機會讀懂錯誤訊息或 stack trace，並推斷出下一次 tool call 應該改變什麼。


Most frameworks implement this, but you can do JUST THIS without doing any of the other 11 factors. Here's an example: 

大多數 frameworks 都有實作這一點，但就算不做其他 11 個 factors，你也可以單獨只做這件事。範例如下：


```python
thread = {"events": [initial_message]}

while True:
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread["events"].append({
    "type": next_step.intent,
    "data": next_step,
  })
  try:
    result = await handle_next_step(thread, next_step) # our switch statement
    # 我們的 switch statement
  except Exception as e:
    # if we get an error, we can add it to the context window and try again
    # 如果發生錯誤，我們可以把它加入 context window，然後再試一次
    thread["events"].append({
      "type": 'error',
      "data": format_error(e),
    })
    # loop, or do whatever else here to try to recover
    # 繼續迴圈，或在這裡做任何有助於恢復的處理
```

You may want to implement an errorCounter for a specific tool call, to limit to ~3 attempts of a single tool, or whatever other logic makes sense for your use case. 

你可能會想為特定 tool call 實作一個 errorCounter，把單一 tool 的嘗試次數限制在大約 3 次，或加入任何更適合你 use case 的邏輯。

```python
consecutive_errors = 0

while True:

  # ... existing code ...
  # ... 既有程式碼 ...

  try:
    result = await handle_next_step(thread, next_step)
    thread["events"].append({
      "type": next_step.intent + '_result',
      data: result,
    })
    # success! reset the error counter
    # 成功了！重設錯誤計數器
    consecutive_errors = 0
  except Exception as e:
    consecutive_errors += 1
    if consecutive_errors < 3:
      # do the loop and try again
      # 繼續迴圈並再試一次
      thread["events"].append({
        "type": 'error',
        "data": format_error(e),
      })
    else:
      # break the loop, reset parts of the context window, escalate to a human, or whatever else you want to do
      # 跳出迴圈、重設部分 context window、升級給人類處理，或做任何你想做的事
      break
  }
}
```
Hitting some consecutive-error-threshold might be a great place to [escalate to a human](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md), whether by model decision or via deterministic takeover of the control flow.

達到某個 consecutive-error-threshold，可能就是一個很適合[升級給人類處理](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)的時機，無論是由 model 決定，還是由 deterministic 的 control flow 接手。

[![195-factor-09-errors](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/195-factor-09-errors.gif)](https://github.com/user-attachments/assets/cd7ed814-8309-4baf-81a5-9502f91d4043)


<details>
<summary>[GIF Version](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/195-factor-09-errors.gif)</summary>

![195-factor-09-errors](https://github.com/RobbinHsu/12-factor-agents/blob/main/img/195-factor-09-errors.gif)

</details>

Benefits:

優點：

1. **Self-Healing**: The LLM can read the error message and figure out what to change in a subsequent tool call
1. **Self-Healing**：LLM 可以讀取錯誤訊息，並推斷下一次 tool call 應該改變什麼
2. **Durable**: The agent can continue to run even if one tool call fails
2. **Durable**：即使某一次 tool call 失敗，agent 仍然可以繼續執行


I'm sure you will find that if you do this TOO much, your agent will start to spin out and might repeat the same error over and over again. 

我很確定你會發現，如果你把這件事做得 TOO much，你的 agent 會開始失控打轉，並且一再重複同樣的錯誤。

That's where [factor 8 - own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) and [factor 3 - own your context building](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) come in - you don't need to just put the raw error back on, you can completely restructure how it's represented, remove previous events from the context window, or whatever deterministic thing you find works to get an agent back on track. 

這就是 [factor 8 - own your control flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) 與 [factor 3 - own your context building](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) 發揮作用的地方——你不必只是把原始錯誤直接放回去；你可以徹底重構它的呈現方式、從 context window 中移除先前事件，或做任何你發現能夠把 agent 拉回正軌的 deterministic 處理。

But the number one way to prevent error spin-outs is to embrace [factor 10 - small, focused agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md).

但要防止 error spin-out，排名第一的方法還是擁抱 [factor 10 - small, focused agents](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)。

[← Own Your Control Flow](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) | [Small Focused Agents →](https://github.com/RobbinHsu/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
