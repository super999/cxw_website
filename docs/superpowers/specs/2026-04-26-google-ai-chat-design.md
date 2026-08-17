# Google AI Chat 页面设计

## 概述

仿照 DeepSeek Chat 页面，实现一个 Google AI 对话页面。使用 Google Gemini API，支持文字对话和图片上传分析。

## 技术栈

- **博客框架**：Hexo 7.3.0
- **页面类型**：静态 HTML（`layout: false`）
- **前端**：Vanilla JS + CSS，无外部依赖
- **API**：Google Gemini API (AI Studio)

## 目录结构

```
cnblog/source/google-ai-chat/
├── index.html      # 主页面（HTML + CSS + JS）
└── api-key.js      # Base64 编码的 API Key
```

## 页面结构

### Header
- 标题："Google AI 对话"
- 模型选择下拉框：
  - `gemini-2.0-flash`（默认）
  - `gemini-3-flash-preview`
- 清空历史按钮

### Chat Container
- 消息列表，垂直滚动
- 每条消息包含：
  - Role（user/assistant）
  - Content（支持 Markdown/代码块渲染）
  - Timestamp

### Input Area
- 图片上传按钮（点击选择文件）
- 文本输入框（支持自动 resize）
- 发送按钮
- 支持拖拽图片上传
- 支持 Enter 发送，Shift+Enter 换行

## 配色方案

| 元素 | 颜色 |
|------|------|
| 主色/渐变起点 | #4285F4（Google Blue） |
| 渐变终点 | #34A853（Google Green） |
| 用户消息气泡 | 渐变（#4285F4 → #34A853） |
| AI 消息气泡 | #F1F3F4 |
| 背景 | #FFFFFF |
| 文字 | #202124 |
| 边框/分隔线 | #DADCE0 |
| 错误提示 | #EA4335（Google Red） |

## API 集成

### 端点
```
POST https://generativelanguage.googleapis.com/v1beta/models/{model}:streamGenerateContent?key={API_KEY}&alt=sse
```

### 请求格式
```json
{
  "contents": [
    {
      "parts": [
        { "text": "用户输入文字" },
        { "inline_data": { "mime_type": "image/jpeg", "data": "base64编码图片" } }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 1.0,
    "topP": 0.95,
    "topK": 40,
    "maxOutputTokens": 8192
  }
}
```

### 响应处理
- 使用 SSE（Server-Sent Events）流式接收
- 实时更新 UI，带 typewriter 效果
- 处理 `[DONE]` 信号

## 数据存储

### localStorage Schema
- Key：`google-ai-chat-history`
- Value：JSON 数组，格式同 DeepSeek Chat
- 最大条数：50条

### 消息格式
```json
{
  "role": "user|assistant",
  "content": "消息内容",
  "timestamp": 1714137600000
}
```

## 实现步骤（分步）

### Step 1: 基础对话功能
- 创建页面基础结构
- 实现纯文字对话
- 支持 SSE 流式响应
- 完成基础 UI 样式

### Step 2: 图片上传功能
- 实现点击上传按钮选择图片
- 实现拖拽上传
- 图片 Base64 编码
- 支持图文混合输入

### Step 3: 完善和优化
- 添加 Markdown 渲染优化
- 添加错误处理优化
- 添加加载状态优化

## 验收标准

1. 纯文字对话正常工作
2. 图片上传和识别正常
3. 支持拖拽上传图片
4. 流式响应正常显示
5. 历史记录正确保存和加载
6. 页面样式符合 Google 主题
7. 无 console error
