# DeepSeek V4 对话页面设计

## 概述

创建一个独立的 DeepSeek V4 对话调试页面，用于测试 API。

## 文件结构

```
cnblog/source/deepseek-chat/
├── index.html      # 主页面
└── api-key.js      # Base64 混淆的 API Key
```

## 页面功能

| 功能 | 描述 |
|------|------|
| 模型选择 | deepseek-v4-flash / deepseek-v4-pro 下拉选择 |
| 消息输入 | textarea，支持 Enter 发送，Shift+Enter 换行 |
| 流式输出 | 打字机效果，实时显示响应 |
| 对话历史 | localStorage 持久保存 |
| 清空按钮 | 页面提供"清空历史"按钮 |

## API Key 安全方案

```javascript
// api-key.js
const apiKey = atob("BASE64_ENCODED_KEY");
```

Key 以 Base64 编码存储，运行时解码使用。Hexo 构建不会将 `source/` 目录暴露到 `public/`。

## 对话历史存储

| 项目 | 描述 |
|------|------|
| 存储方式 | localStorage |
| Key 名称 | `deepseek-chat-history` |
| 数据格式 | JSON 数组，每条记录含 role/content/timestamp |
| 最大条数 | 50 条（超出则删除最早的） |
| 清除按钮 | 页面提供"清空历史"按钮 |

## API 调用

- **端点**: `https://api.deepseek.com/chat/completions`
- **方法**: POST
- **认证**: Bearer Token
- **模型**: `deepseek-v4-flash` 或 `deepseek-v4-pro`

## 技术实现

- 纯原生 HTML + CSS + JavaScript
- 无外部依赖
- fetch API 调用 DeepSeek
- ReadableStream 处理流式响应

## 验收标准

1. 页面可独立访问
2. API Key 不以明文出现在页面源码中
3. 对话历史在刷新后保留
4. 流式输出正常显示
5. 清空历史后页面重置
