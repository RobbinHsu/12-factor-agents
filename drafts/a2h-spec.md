# A2H - The Agent-to-Human Protocol
# A2H - Agent-to-Human 通訊協定

## Overview
## 概述

A2H is a service that allows an agent to request human interaction

A2H 是一項服務，可讓 agent 請求人類互動。

## Why another protocol?
## 為什麼需要另一個 protocol？

MCP and A2A are not enough

MCP 和 A2A 還不夠。

## Shoulds
## 建議事項

- Clients should respect A2H_BASE_URL and A2H_API_KEY environment variables if set, to allow for simple oauth2-based authentication to REST services.
- 如果已設定 `A2H_BASE_URL` 與 `A2H_API_KEY` 環境變數，client 應予以遵循，以支援對 REST 服務進行簡單的 oauth2 驗證。


## Core Protocol 
## 核心協定

### Scopes 
### 範圍

The A2H protocol supports two scopes:

A2H protocol 支援兩個範圍：

- The agent side, APIs consumed by an agent to request human interaction
- agent 端：由 agent 使用、用於請求人類互動的 API
- The (Optional) admin side, APIs consumed by an admin or web application to manage humans and their contact channels
- （可選）admin 端：由 admin 或 web application 使用、用於管理人員及其聯絡管道的 API


This separation allows for agents to query and find humans to contact, without exposing the human's contact details to the agent. It is the responsibility of the A2H provider to relay agent requests to the appropriate human via that human's preferred contact channel(s).

這種分離可讓 agent 查詢並尋找可聯絡的人員，同時不會將該人員的聯絡資訊暴露給 agent。A2H provider 有責任透過該人員偏好的聯絡管道，將 agent 的請求轉送給適當的人。

### Objects
### 物件

```
apiVersion: proto.a2h.dev/v1alpha1
kind: Message
metatdata:
  uid: "123"
spec: # spec sent by agent
      # agent 傳送的 spec
  message: "" # message from the agent
              # 來自 agent 的訊息
  response_schema:
   # optional, json schema for the response,
   # 可選，用於回應的 JSON schema
  channel_id: 
status: # status resolved by a2h server
        # 由 a2h server 判定的狀態
  humanMessage: "" # message from the human
                   # 來自人類的訊息
  response:
    # optional, matches spec schema
    # 可選，需符合 spec schema
```

```
apiVersion: proto.a2h.dev/v1alpha1
kind: NewConversation
metadata:
  uid: "abc"
spec: # spec sent by a2h server
      # 由 a2h server 傳送的 spec
  message: "" # message from the agent
              # 來自 agent 的訊息
  channel_id: "123" # channel id to use for future conversations
                    # 未來對話要使用的 channel id
  response_schema:
   # optional, json schema for the response,
   # 可選，用於回應的 JSON schema
```

#### HumanContact
#### HumanContact

```json
{
  "run_id": "run_123",
  "call_id": "call_456",
  "spec": {
    "msg": "I've tried using the tool to refund the customer but its returning a 500 error. Can you help?",
    "channel": {
      "slack": {
        "channel_or_user_id": "U1234567890",
        "context_about_channel_or_user": "Support team lead"
      }
    },
  },
}
```

A HumanContact represents a request for human interaction. It contains:

HumanContact 代表一個請求人類互動的請求。它包含：

- `run_id` (string): Unique identifier for the run
- `run_id`（string）：此次 run 的唯一識別碼
- `call_id` (string): Unique identifier for the contact request
- `call_id`（string）：此次聯絡請求的唯一識別碼
- `spec` (HumanContactSpec): The specification for the contact request
- `spec`（HumanContactSpec）：聯絡請求的規格
- `status` (HumanContactStatus, optional): The current status of the contact request
- `status`（HumanContactStatus，可選）：聯絡請求的目前狀態


The HumanContactSpec contains:

HumanContactSpec 包含：

- `msg` (string): The message to send to the human
- `msg`（string）：要傳送給人類的訊息
- `subject` (string, optional): Subject of the contact request
- `subject`（string，可選）：聯絡請求的主旨
- `channel` (ContactChannel, optional): The channel to use for contact
- `channel`（ContactChannel，可選）：要使用的聯絡管道
- `response_options` (ResponseOption[], optional): Available response options
- `response_options`（ResponseOption[]，可選）：可用的回應選項
- `state` (object, optional): Additional state information
- `state`（object，可選）：額外的狀態資訊


The HumanContactStatus contains:

HumanContactStatus 包含：

- `requested_at` (datetime, optional): When the contact was requested
- `requested_at`（datetime，可選）：提出聯絡請求的時間
- `responded_at` (datetime, optional): When the human responded
- `responded_at`（datetime，可選）：人類回覆的時間
- `response` (string, optional): The human's response
- `response`（string，可選）：人類的回應
- `response_option_name` (string, optional): Name of the selected response option
- `response_option_name`（string，可選）：所選回應選項的名稱
- `slack_message_ts` (string, optional): Slack message timestamp if applicable
- `slack_message_ts`（string，可選）：若適用，Slack 訊息的時間戳
- `failed_validation_details` (object, optional): Details if validation failed
- `failed_validation_details`（object，可選）：驗證失敗時的詳細資訊


#### FunctionCall
#### FunctionCall

Example:

範例：

```json
{
  "run_id": "run_789",
  "call_id": "call_101",
  "spec": {
    "fn": "process_payment",
    "kwargs": {
      "amount": 100.00,
      "currency": "USD",
      "recipient": "merchant_123"
    },
    "channel": {
      "email": {
        "address": "ap@example.com",
      }
    },
  },
  "status": {
    "requested_at": "2024-03-20T11:00:00Z",
    "responded_at": "2024-03-20T11:02:00Z",
    "approved": true,
    "comment": "Payment looks good, approved",
    "user_info": {
      "name": "John Doe",
      "role": "Finance Manager"
    },
    "slack_message_ts": "1234567890.123457"
  }
}
```

A FunctionCall represents a request for human approval of a function execution. It contains:

FunctionCall 代表對函式執行請求人類核准的請求。它包含：

- `run_id` (string): Unique identifier for the run
- `run_id`（string）：此次 run 的唯一識別碼
- `call_id` (string): Unique identifier for the function call
- `call_id`（string）：此次函式呼叫的唯一識別碼
- `spec` (FunctionCallSpec): The specification for the function call
- `spec`（FunctionCallSpec）：函式呼叫的規格
- `status` (FunctionCallStatus, optional): The current status of the function call
- `status`（FunctionCallStatus，可選）：函式呼叫的目前狀態


The FunctionCallSpec contains:

FunctionCallSpec 包含：

- `fn` (string): The function to be called
- `fn`（string）：要呼叫的函式
- `kwargs` (object): The keyword arguments for the function
- `kwargs`（object）：該函式的 keyword arguments
- `channel` (ContactChannel, optional): The channel to use for contact
- `channel`（ContactChannel，可選）：要使用的聯絡管道
- `reject_options` (ResponseOption[], optional): Available rejection options
- `reject_options`（ResponseOption[]，可選）：可用的拒絕選項
- `state` (object, optional): Additional state information
- `state`（object，可選）：額外的狀態資訊


The FunctionCallStatus contains:

FunctionCallStatus 包含：

- `requested_at` (datetime, optional): When the approval was requested
- `requested_at`（datetime，可選）：提出核准請求的時間
- `responded_at` (datetime, optional): When the human responded
- `responded_at`（datetime，可選）：人類回覆的時間
- `approved` (boolean, optional): Whether the function call was approved
- `approved`（boolean，可選）：函式呼叫是否已核准
- `comment` (string, optional): Any comment from the human
- `comment`（string，可選）：人類提供的任何評論
- `user_info` (object, optional): Information about the responding user
- `user_info`（object，可選）：回覆者的資訊
- `slack_context` (object, optional): Slack-specific context
- `slack_context`（object，可選）：Slack 專屬的 context
- `reject_option_name` (string, optional): Name of the selected rejection option
- `reject_option_name`（string，可選）：所選拒絕選項的名稱
- `slack_message_ts` (string, optional): Slack message timestamp if applicable
- `slack_message_ts`（string，可選）：若適用，Slack 訊息的時間戳
- `failed_validation_details` (object, optional): Details if validation failed
- `failed_validation_details`（object，可選）：驗證失敗時的詳細資訊


#### ContactChannel
#### ContactChannel

Example:

範例：

```json
{
  "slack": {
    "channel_or_user_id": "U1234567890",
    "context_about_channel_or_user": "Support team lead",
    "allowed_responder_ids": ["U1234567890", "U2345678901"],
    "experimental_slack_blocks": true,
    "thread_ts": "1234567890.123456"
  }
}
```

or

或

```json
{
    "email": {
        "address": "ap@example.com",
        "context_about_user": "Accounts Payable",
        "in_reply_to_message_id": "1234567890",
        "references_message_id": "1234567890",
        "template": "<html><body>...</body></html>"
    }
}
```

A ContactChannel represents a channel through which a human can be contacted. The protocol supports several channel types:

ContactChannel 代表可用來聯絡人類的管道。此 protocol 支援數種管道類型：

1. SlackContactChannel:
   - `channel_or_user_id` (string): The Slack channel or user ID
   - `context_about_channel_or_user` (string, optional): Additional context
   - `bot_token` (string, optional): Bot token for authentication
   - `allowed_responder_ids` (string[], optional): IDs of allowed responders
   - `experimental_slack_blocks` (boolean, optional): Enable experimental blocks
   - `thread_ts` (string, optional): Thread timestamp for threaded messages

2. SMSContactChannel:
   - `phone_number` (string): The phone number to contact
   - `context_about_user` (string, optional): Additional context about the user

3. WhatsAppContactChannel:
   - `phone_number` (string): The phone number to contact
   - `context_about_user` (string, optional): Additional context about the user

1. SlackContactChannel：
   - `channel_or_user_id`（string）：Slack channel 或 user ID
   - `context_about_channel_or_user`（string，可選）：額外的 context
   - `bot_token`（string，可選）：用於驗證的 Bot token
   - `allowed_responder_ids`（string[]，可選）：允許回覆者的 ID 清單
   - `experimental_slack_blocks`（boolean，可選）：啟用實驗性 blocks
   - `thread_ts`（string，可選）：串接訊息的 thread 時間戳

2. SMSContactChannel：
   - `phone_number`（string）：要聯絡的電話號碼
   - `context_about_user`（string，可選）：關於使用者的額外 context

3. WhatsAppContactChannel：
   - `phone_number`（string）：要聯絡的電話號碼
   - `context_about_user`（string，可選）：關於使用者的額外 context

#### Human (Agent Side)
#### Human（Agent 端）

From the agent's perspective, a human is an object that has a name and description.

從 agent 的角度來看，human 是一個具有名稱與描述的物件。

#### Human (Admin Side)
#### Human（Admin 端）

From the admin's perspective, a human is an object that has a name, description, and a list of prioritized contact channels, with details 

從 admin 的角度來看，human 是一個具有名稱、描述，以及依優先順序排列之聯絡管道明細清單的物件。

### Agent Endpoints
### Agent 端點

#### POST /human_contacts
#### POST /human_contacts（建立 HumanContact）

#### GET /human_contacts/:call_id
#### GET /human_contacts/:call_id（取得 HumanContact）

#### POST /function_calls
#### POST /function_calls（建立 FunctionCall）

#### GET /function_calls/:call_id
#### GET /function_calls/:call_id（取得 FunctionCall）

## Extended Protocol
## 延伸協定

- Admin Humans
- Agent Humans Get
- Agent Humans Search
- Agent Channels List
- Agent Channels validate

- Admin Humans
- Agent Humans Get
- Agent Humans Search
- Agent Channels List
- Agent Channels validate

### Objects
### 物件

#### Human (Agent Side)
#### Human（Agent 端）

From the agent's perspective, a human is an object that has a name and description.

從 agent 的角度來看，human 是一個具有名稱與描述的物件。

#### Human (Admin Side)
#### Human（Admin 端）

From the admin's perspective, a human is an object that has a name, description, and a list of prioritized contact channels, with details 

從 admin 的角度來看，human 是一個具有名稱、描述，以及依優先順序排列之聯絡管道明細清單的物件。

### Agent Endpoints
### Agent 端點

#### GET /channels 
#### GET /channels（取得可用聯絡管道）

return what contact channels are available and their supported fields

回傳可用的聯絡管道以及各自支援的欄位。

example response:

回應範例：

```json
{
    "channels": {
        "slack": {
            "channelOrUserId": {
                "type": "string",
                "description": "The Slack channel or user ID to send messages to"
            },
            "contextAboutChannelOrUser": {
                "type": "string", 
                "description": "Additional context about the Slack channel or user"
            }
        },
        "email": {
            "address": {
                "type": "string",
                "description": "Email address to send messages to"
            },
            "contextAboutUser": {
                "type": "string",
                "description": "Additional context about the email recipient"
            },
            "inReplyToMessageId": {
                "type": "string",
                "description": "The message ID of the email to reply to"
            },
            "referencesMessageId": {
                "type": "string",
                "description": "The message ID of the email to reference"
            }
        }
    }
}
```

#### GET /humans
#### GET /humans（取得可互動的人員清單）

return a list of humans that are available to interact with

回傳目前可互動的人員清單。

example response:

回應範例：

```json
{
    "humans": [
        {
            "id": "654",
            "name": "Jane Doe",
            "description": "Jane Doe is a human who knows about technology and entrepreneurship",
        },
        {
            "id": "123",
            "name": "John Doe",
            "description": "John Doe is a human who knows about sales and marketing"
        }
    ]
}
```
#### GET /humans/search?q=
#### GET /humans/search?q=（依名稱或描述搜尋人員）

search for humans by name or description

依名稱或描述搜尋人員。

example response:

回應範例：

```json
{
    "humans": [
        {
            "id": "654",
            "name": "Jane Doe",
            "description": "Jane Doe is a human who knows about technology and entrepreneurship",
        },
    ]
}
```

### Administrative Endpoints
### 管理端點

#### POST /humans
#### POST /humans（註冊新的人員）

Enroll a new human for agent contact

註冊一位可供 agent 聯絡的新成員。

example request:

請求範例：

```json
{
    "name": "John Doe",
    "description": "John Doe is a human who knows about sales and marketing",
    "prioritizedContactChannels": [
        {
            "slack": {
                "channelOrUserId": "U1234567890",
            }
        },
        {
            "email": {
                "address": "john.doe@example.com",
            }
        }
    ]
}
```

#### GET /humans/:id
#### GET /humans/:id（依 ID 取得人員）

Get a human by id

依 ID 取得人員。

example response:

回應範例：

```json
