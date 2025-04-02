# MCP、FunctionCall科普


{{< admonition note >}}
> 以下将 MCP (主要指服务端) 和 FunctionCall (主要指 OpenAI chat 规范的函数调用) 都笼统称为“插件”，方便快速理解，
> - 模型本身不具备调用 function 或 mcp Server 的能力
> - 两者都是告诉远程模型，客户端具备那些额外能力(如天气插件、数据库插件、画图插件)，模型依据当前上下文 (当前语句)，判断当前场景是否应该执行某几个插件, 以及参数是什么,并返回一个特殊的响应，将结果返回到客户端
> - 模型本身只是返回各种"文本"
> - 当客户端(如 CherryStudio、NextChat、openwebui) 观察到模型返回了某种特殊的响应文本，客户端就执行模型响应文本中告知的插件，当然，客户端可以完全无视模型的特殊响应
{{< /admonition >}}


一个简易的流程如下
![image.png](https://uchat.cn-bj.ufileos.com/rw_ebfde1fd-40d2-441f-b02b-be335fe7e185_image.png)
## FunctionCall

以获取本地天气为例子

请求结构如下：
通过 `functions` 或 `tools` (最新的 `functions` 的别名) 告知了模型，当前客户端具备获取天气的能力
```json
{
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "今天上海天气怎么样？，湿度怎么样？"
        }
    ],
    "functions": [
        {
            "name": "get_localtion_weather",
            "description": "get_localtion_weather，获取某个地点的当前天气情况",
            "parameters": {
                "type": "object",
                "properties": {
                    "localtion": {
                        "type": "string",
                        "description": "localtion,地点"
                    },
                    "need_humidity":{
                        "type":"boolean",
                        "description":"是否返回湿度，默认为false"
                    }
                },
                "required": [
                    "localtion"
                ]
            }
        }
    ],
    "function_call": "auto",
    "temperature": 0.5,
    "stream":false,
    "model": "gpt-4o"
}
```

```shell
curl --location 'https://api.xxxxx.com/v1/chat/completions' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer sk-xxxxx' \
--data '{
    "messages": [
        {
            "role": "system",
            "content": "You are a helpful assistant."
        },
        {
            "role": "user",
            "content": "今天上海天气怎么样？，湿度怎么样？"
        }
    ],
    "functions": [
        {
            "name": "get_localtion_weather",
            "description": "get_localtion_weather，获取某个地点的当前天气情况",
            "parameters": {
                "type": "object",
                "properties": {
                    "localtion": {
                        "type": "string",
                        "description": "localtion,地点"
                    },
                    "need_humidity":{
                        "type":"boolean",
                        "description":"是否返回湿度，默认为false"
                    }
                },
                "required": [
                    "localtion"
                ]
            }
        }
    ],
    "function_call": "auto",
    "temperature": 0.5,
    "stream":false,
    "model": "gpt-4o"
}'
```

响应结构如下：
![image.png](https://uchat.cn-bj.ufileos.com/rw_de152be6-9ea9-43de-98b1-019f9ed5e9c5_image.png)

客户端执行本地的 `get_localtion_weather` 函数

整体流程如图所示 (下图截取自 OpenAI 的文档，翻译来自沉浸式翻译)

![image.png](https://uchat.cn-bj.ufileos.com/rw_d41e55c5-7988-4fb7-a09a-fb51eaf44a5d_image.png)

## MCP

简单理解：functionCall 是大家自己写自己的，没有一种统一的约定，而 MCP 是一种君子约定，mcpServer (查询天气插件) 遵循 MCP 协议开发，由 mcpClient (Cursor) 调用，原本需要自己写函数，现在变成了**调用别人的库**

![image.png](https://uchat.cn-bj.ufileos.com/rw_dea11dcc-83e9-4614-b73e-0c7e98c95e8b_image.png)

MCP 重点在于 MCP 客户端 (`Cherry Studio`) 如何发现 MCP 服务端 ("插件")、理解 “插件”具备的能力、调用“插件”、获取“插件”的响应结果

至于
- 将当前具备的插件功能发送到模型 (functions (别名 tools) 描述方式、提示词方式)
- MCP 客户端 (`Cherry Studio`) 如何根据模型的响应判断执行是否执行“插件”，执行那几个插件，则是客户端自己定义
- MCP 客户端将插件执行结果返回给模型，也是可以自己定义

以下部分只是在应用领域领域，举例描述了结合MCP 对话流程，并没有对 MCP 这个“君子约定”做具体的详解
### `Cherry Studio` 举例

配置 mcpServers 如下，其中只有一个获取 `和风天气` 的 server
源自：[HeFeng Weather MCP Server \| Glama](https://glama.ai/mcp/servers/@shanggqm/hefeng-mcp-weather)
```json
{
  "mcpServers": {
    "hefeng-weather": {
      "isActive": true,
      "command": "npx",
      "args": [
        "hefeng-mcp-weather@latest",
        "--apiKey=key-123132131321"
      ]
    }
  }
}
```

对话如下
![image.png](https://uchat.cn-bj.ufileos.com/rw_178d8754-cf4b-4c16-ade6-dea68e105997_image.png)

1. 查看本地拦截的第一次请求，可以发现是通过 tools 描述的方式，告诉模型，本地具备那些“插件”以及具备什么能力
```json
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": "今天上海天气怎么样？"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 2000,
  "stream": true,
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "fd71edc4f65924613b9fd8330e78eb243",
        "description": "获取中国国内的天气预报",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "逗号分隔的经纬度信息 (e.g., 116.40,39.90)"
            },
            "days": {
              "type": "string",
              "enum": [
                "now",
                "24h",
                "72h",
                "168h",
                "3d",
                "7d",
                "10d",
                "15d",
                "30d"
              ],
              "description": "预报天数，now为实时天气，24h为24小时预报，72h为72小时预报，168h为168小时预报，3d为3天预报，以此类推"
            }
          }
        }
      }
    }
  ]
}
```

2. 查看第一次响应，模型判断是否执行触发调用插件。
模型判断执行需要用到"天气插件"，模型本身只是做了判断

![image.png](https://uchat.cn-bj.ufileos.com/rw_cf3d1625-72fd-4e08-a46c-2d2edef1c333_image.png)

3. MCP 客户端 (`Cherry Studio`) 执行“天气插件”，可以简单看作调用了一个别人写好的包或API
4. 将执行结果返回给模型，也就是加入对话上下文
```json
{
  "model": "gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": "今天上海天气怎么样？"
    },
    {
      "role": "assistant",
      "tool_calls": [
        {
          "id": "call_M4eYNv6oPaLunpZLe1iWdfiK",
          "function": {
            "name": "fd71edc4f65924613b9fd8330e78eb243",
            "arguments": "{\"days\":\"now\",\"location\":\"121,31\"}"
          },
          "type": "function"
        }
      ]
    },
    {
      "role": "tool",
      "content": "[{\"type\":\"text\",\"text\":\"地点: 121,31\\n观测时间: 2025-03-27T16:51+08:00\\n天气: 阴\\n温度: 13°C\\n体感温度: 12°C\\n风向: 东北风\\n风力: 1级\"}]",
      "tool_call_id": "call_M4eYNv6oPaLunpZLe1iWdfiK"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 2000,
  "stream": true,
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "fd71edc4f65924613b9fd8330e78eb243",
        "description": "获取中国国内的天气预报",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "逗号分隔的经纬度信息 (e.g., 116.40,39.90)"
            },
            "days": {
              "type": "string",
              "enum": [
                "now",
                "24h",
                "72h",
                "168h",
                "3d",
                "7d",
                "10d",
                "15d",
                "30d"
              ],
              "description": "预报天数，now为实时天气，24h为24小时预报，72h为72小时预报，168h为168小时预报，3d为3天预报，以此类推"
            }
          }
        }
      }
    }
  ]
}
```

5. 模型结合上下文 (发送的对话历史) 进行响应
截取其中一部分
![image.png](https://uchat.cn-bj.ufileos.com/rw_f275337f-744d-4b43-b630-0d4b9c30da5a_image.png)

`Cherry Studio` 是通过 functionCall 的方式，判断应该执行那些“插件”的功能

### 自定义协议 (使用系统提示词的方式)

```text
你是一个智能助手，你具备MCP功能，MCP可以理解为一种本地功能插件，当用户的语句需要调用插件时，你应该严格按照以下规定返回，规定如下:
<plug>
	<pn>插件名称</pn>
	<fn>函数名称(功能名称)</fn>
	<arg1>参数</arg1>
	<arg2>参数2<arg2>
	<arg3>参数3<arg3>
<plug/>。
当本地执行了某插件后，结果回按照以下格式返回
<tool_res>
	<plug>
		<pn>插件名称</pn>
		<fn>函数名称(功能名称)</fn>
		<res>执行结果<res/>
	<plug/>
<tool_res/>

当前本地插件:
1.插件名称：get_localtion_weather
	描述：查询某个地点天气插件
	 功能：
	 - query_now_weather:
	 	args:
	 		location:string,必需的,查询地点
	 		need_humidity：boolean,非必需的，默认为false，是否返回湿度
```

图中所示为 `ChatBox`，其他的客户端会对 `<xx><xx/>` 进行处理，显示不出来

![image.png](https://uchat.cn-bj.ufileos.com/rw_548a672e-7876-4451-a8da-5d0b2d3b6375_image.png)

![image.png](https://uchat.cn-bj.ufileos.com/rw_604fce9a-f8fe-4487-a6da-6411af2c5b8c_image.png)

## 参考视频和链接🔗

- [MCP是怎么对接大模型的？抓取AI提示词，拆解MCP的底层原理\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1P3XTYPEJm/)
- [【大模型函数调用】function calling及自动发邮件案例\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1eH4y1c7KQ)
- [🫴手把手教你QwQ-32B如何玩转function call\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1EAQ3YwEFC)
- [7分钟讲清楚MCP是什么](https://www.bilibili.com/video/BV1uXQzYaEpJ)
