---
title: 简单聊聊 Figma MCP
date: 2025-09-12
draft: false
descritpion: '聊聊 Figma MCP 和对 AI 时代前端开发的看法'
categories:
  - other
---

(在 2025-07-07 文档上修改)
## Figma quick start guide
https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server

## 为什么用
Figma 从很早的时候，就推出了 Design System，Design to Code 这些功能，从原子的 Design Token 到组件，都极大的提高了研发的效率。然而，要用好这套系统，得对设计、研发都有所了解，和配置大量的内容（token映射、组件管理等）；并在设计中按照规范使用。相当于将前端研发的逻辑套入了设计阶段。虽然能够给后续开发带来极大的提升，但是构建和修改这个系统的门槛不低。

如果使用 Figma MCP，这个流程就能解耦了。研发不必知道设计是否运用了组件（当然最好保留 Design Token 的使用），只需要告诉 Agent 使用了那个组件代码（定制、 开源），获取到设计结构再做判断。

## 怎么用
跟随官方的 quick start，往对应平台做类似以下配置。这里我使用的是 Cursor
```json
{
  "mcpServers": {
    "Figma": {
      "url": "http://127.0.0.1:3845/sse"
    },
    "chrome-devtools": {
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest"]
    }
  }
}
```
然后对图层右键，Copy link to selection.

在以前版本，需要明确的给 Agent 指出：使用 XXX MCP 这种操作，添加比如检测到 figma.com/design 地址 等等提示。截至 09-29 最新版本。如下图，可以看到已经将支持的方法都列举出来了，并且在调用的时候，展示 MCP 获取返回的数据。

![cursor mcp settings.png](https://s2.loli.net/2025/09/29/iocYlad9H37pnOW.png)

figma mcp 返回的是已经 Design 2 code 的前端代码，而不是 export .fig 的设计树，一个是前端代码更通用，设计树数据无效 token（文本）比较多，但由于图层可能有很多的 svg path，很多的元素。也很容易导致 Token 太长超量。所以，在官方文档中也是推荐的在小范围内使用！基于 Cursor claude-4-sonnet Max Mode 去同步 Design Token 的 Color 和 Text，生成对应的前端 CSS 变量，效果非常一致，但是大概花费超 30$。

这个方式虽然很便利，也很看 Agent 心情，并且它觉得对，那就不改了。生成的代码有时候会有很多 figma D2C 的无用代码，比如一些绝对定位等等信息。

## Chrome Devtool MCP
https://developer.chrome.com/blog/chrome-devtools-mcp 

以上步骤做到了转换生成，最终效果无法验证。可用性就大打折扣，需要反复的和 Agent 沟通。期间我也试用了很多 Agent 浏览器，都没有找到很合适的。09-23, 官方发布了 chrome mcp，可以通过它提供浏览器的填充和快照等操作。虽然很多人机验证等等都没法通过，但是为什么在这里介绍。

agent 结合调用 figma mcp、devtool mcp 的快照方式，就能完成，输入设计，输出生成，验证生成的这个闭环。