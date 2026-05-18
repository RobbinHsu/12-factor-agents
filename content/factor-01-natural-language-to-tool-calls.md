[← Back to README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)
[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 1. Natural Language to Tool Calls 
### 1. 自然語言轉 tool calls

One of the most common patterns in agent building is to convert natural language to structured tool calls. This is a powerful pattern that allows you to build agents that can reason about tasks and execute them.

在建構 agent 時，最常見的模式之一，就是把自然語言轉成結構化的 tool calls。這是一種強大的模式，能讓你打造出可以對任務進行推理並加以執行的 agents。

![110-natural-language-tool-calls](https://github.com/humanlayer/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)

This pattern, when applied atomically, is the simple translation of a phrase like

當這個模式以原子化方式套用時，它就是把像這樣的一句話，簡單地轉換成另一種形式：

> can you create a payment link for $750 to Terri for sponsoring the february AI tinkerers meetup? 

> 你可以為 Terri 建立一個 $750 的 payment link，用來贊助 2 月的 AI tinkerers meetup 嗎？

to a structured object that describes a Stripe API call like

轉成一個描述 Stripe API 呼叫的結構化物件，例如：

```json
{
  "function": {
    "name": "create_payment_link",
    "parameters": {
      "amount": 750,
      "customer": "cust_128934ddasf9",
      "product": "prod_8675309",
      "price": "prc_09874329fds",
      "quantity": 1,
      "memo": "Hey Jeff - see below for the payment link for the february ai tinkerers meetup"
    }
  }
}
```

**Note**: in reality the stripe API is a bit more complex, a [real agent that does this](https://github.com/dexhorthy/mailcrew) ([video](https://www.youtube.com/watch?v=f_cKnoPC_Oo)) would list customers, list products, list prices, etc to build this payload with the proper ids, or include those ids in the prompt/context window (we'll see below how those are kinda the same thing though!)

**注意**：實際上，stripe API 會再複雜一些；一個[真正會這樣做的 agent](https://github.com/dexhorthy/mailcrew)（[影片](https://www.youtube.com/watch?v=f_cKnoPC_Oo)）會先列出 customers、products、prices 等資訊，才能用正確的 ids 建立這個 payload，或者把那些 ids 直接放進 prompt/context window 中（稍後你會看到，這兩者某種程度上其實差不多！）

From there, deterministic code can pick up the payload and do something with it. (More on this in [factor 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md))

從這裡開始，deterministic code 就可以接手這個 payload，並據此執行某些操作。（更多內容請見 [factor 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)）

```python
# The LLM takes natural language and returns a structured object
# LLM 接收自然語言並回傳結構化物件
nextStep = await llm.determineNextStep(
  """
  create a payment link for $750 to Jeff 
  for sponsoring the february AI tinkerers meetup
  """
  )

# Handle the structured output based on its function
# 根據其 function 處理結構化輸出
if nextStep.function == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return  # or whatever you want, see below
    # 或其他你想要的處理，見下文
elif nextStep.function == 'something_else':
    # ... more cases
    # ……更多情況
    pass
else:  # the model didn't call a tool we know about
    # model 沒有呼叫我們已知的 tool
    # do something else
    # 做其他處理
    pass
```

**NOTE**: While a full agent would then receive the API call result and loop with it, eventually returning something like

**注意**：完整的 agent 接著會收到 API 呼叫結果，並據此持續迴圈，最後回傳類似這樣的內容：

> I've successfully created a payment link for $750 to Terri for sponsoring the february AI tinkerers meetup. Here's the link: https://buy.stripe.com/test_1234567890

> 我已成功為 Terri 建立一個 $750 的 payment link，用於贊助 2 月的 AI tinkerers meetup。連結如下：https://buy.stripe.com/test_1234567890

**Instead**, We're actually going to skip that step here, and save it for another factor, which you may or may not want to also incorporate (up to you!)

**不過**，這裡我們會先跳過那一步，留到另一個 factor 再談；至於你是否也想把它納入，則由你決定！

[← How We Got Here](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software.md) | [Own Your Prompts →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
[← 我們如何走到這裡](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software.md) | [掌握你的 prompts →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
