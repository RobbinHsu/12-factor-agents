### Factor 13 - pre-fetch all the context you might need
### 要素 13 - 預先抓取你可能需要的所有 context

If there's a high chance that your model will call tool X, don't waste token round trips telling the model to fetch it, that is, instead of a pseudo-prompt like:

如果你的 model 很有可能會呼叫 tool X，就不要把 token round trips 浪費在告訴 model 自己去抓它這件事上；也就是說，與其寫出像下面這樣的 pseudo-prompt：

```jinja
When looking at deployments, you will likely want to fetch the list of published git tags,
so you can use it to deploy to prod.

Here's what happened so far:

{{ thread.events }}

What's the next step?

Answer in JSON format with one of the following intents:

{
  intent: 'deploy_backend_to_prod',
  tag: string
} OR {
  intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}
```

and your code looks like

而你的程式碼長這樣：

```python
thread = {"events": [initial_message]}
next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'list_git_tags':
      tags = await fetch_git_tags()
      thread["events"].append({
        type: 'list_git_tags',
        data: tags,
      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

You might as well just fetch the tags and include them in the context window, like:

你其實不如直接把 tags 抓回來，然後放進 context window，像這樣：

```diff
- When looking at deployments, you will likely want to fetch the list of published git tags,
- so you can use it to deploy to prod.

+ The current git tags are:

+ {{ git_tags }}


Here's what happened so far:

{{ thread.events }}

What's the next step?

Answer in JSON format with one of the following intents:

{
  intent: 'deploy_backend_to_prod',
  tag: string
- } OR {
-   intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}

```

and your code looks like

而你的程式碼會變成這樣：

```diff
thread = {"events": [initial_message]}
+ git_tags = await fetch_git_tags()

- next_step = await determine_next_step(thread)
+ next_step = await determine_next_step(thread, git_tags)

while True:
  switch next_step.intent:
-    case 'list_git_tags':
-      tags = await fetch_git_tags()
-      thread["events"].append({
-        type: 'list_git_tags',
-        data: tags,
-      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

or even just include the tags in the thread and remove the specific parameter from your prompt template:

或者甚至可以直接把 tags 放進 thread 裡，並從 prompt template 中移除那個特定參數：

```diff
thread = {"events": [initial_message]}
+ # add the request
+ # 加入請求
+ thread["events"].append({
+  "type": 'list_git_tags',
+ })

git_tags = await fetch_git_tags()

+ # add the result
+ # 加入結果
+ thread["events"].append({
+  "type": 'list_git_tags_result',
+  "data": git_tags,
+ })

- next_step = await determine_next_step(thread, git_tags)
+ next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append(deploy_result)
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

Overall:

總結：

> #### If you already know what tools you'll want the model to call, just call them DETERMINISTICALLY and let the model do the hard part of figuring out how to use their outputs
>
> #### 如果你已經知道 model 會想呼叫哪些 tools，那就直接以 DETERMINISTIC 的方式先呼叫它們，然後把真正困難的部分——也就是如何使用這些輸出——交給 model 處理

Again, AI engineering is all about [Context Engineering](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md).

再次強調，AI engineering 的核心就是 [Context Engineering](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)。

[← Stateless Reducer](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md) | [Further Reading →](https://github.com/humanlayer/12-factor-agents/blob/main/README.md#related-resources)
