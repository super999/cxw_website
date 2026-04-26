# DeepSeek V4 对话页面实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建独立的 DeepSeek V4 对话调试页面，支持流式输出和对话历史持久化

**Architecture:** 纯前端实现，使用 fetch API 调用 DeepSeek V4 API，Base64 混淆存储 API Key，localStorage 持久化对话历史

**Tech Stack:** 原生 HTML + CSS + JavaScript，无外部依赖

---

## 文件结构

```
cnblog/source/deepseek-chat/
├── index.html      # 主页面
└── api-key.js      # Base64 混淆的 API Key

cnblog/source/_posts/
└── deepseek-chat.md   # 博客文章
```

---

## Task 1: 创建 api-key.js

**Files:**
- Create: `cnblog/source/deepseek-chat/api-key.js`

- [ ] **Step 1: 创建 api-key.js 文件**

```javascript
// DeepSeek API Key (Base64 编码)
// 使用: atob("YOUR_BASE64_ENCODED_KEY") 解码
const apiKey = atob("YOUR_BASE64_ENCODED_KEY_HERE");
```

- [ ] **Step 2: 提示用户替换为自己的 Base64 编码 Key**

用户需要将自己的 DeepSeek API Key 进行 Base64 编码后替换 `YOUR_BASE64_ENCODED_KEY_HERE`

- [ ] **Step 3: 提交**

```bash
git add cnblog/source/deepseek-chat/api-key.js
git commit -m "feat: add DeepSeek API key file"
```

---

## Task 2: 创建 index.html 主页面

**Files:**
- Create: `cnblog/source/deepseek-chat/index.html`

- [ ] **Step 1: 创建 index.html 文件结构**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DeepSeek V4 对话测试</title>
    <style>
        /* CSS 样式 */
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>DeepSeek V4 对话测试</h1>
            <div class="controls">
                <select id="model-select">
                    <option value="deepseek-v4-flash">deepseek-v4-flash</option>
                    <option value="deepseek-v4-pro">deepseek-v4-pro</option>
                </select>
                <button id="clear-btn">清空历史</button>
            </div>
        </header>
        <div id="chat-container">
            <div id="messages"></div>
        </div>
        <div class="input-area">
            <textarea id="user-input" placeholder="输入消息..."></textarea>
            <button id="send-btn">发送</button>
        </div>
    </div>
    <script src="api-key.js"></script>
    <script>
        // JavaScript 实现
    </script>
</body>
</html>
```

- [ ] **Step 2: 实现 CSS 样式**

基础样式：聊天容器、消息气泡（user 右对齐/assistant 左对齐）、输入框样式、按钮样式

- [ ] **Step 3: 实现 JavaScript 功能**

1. `loadHistory()` - 从 localStorage 加载历史记录
2. `saveHistory()` - 保存历史到 localStorage（最大 50 条）
3. `addMessage(role, content)` - 添加消息到界面和历史
4. `sendMessage()` - 发送消息到 DeepSeek API
5. `handleStreamResponse(response)` - 处理流式响应
6. `clearHistory()` - 清空历史记录

- [ ] **Step 4: 实现 API 调用**

```javascript
async function sendMessage() {
    const userMessage = document.getElementById('user-input').value.trim();
    if (!userMessage) return;

    const model = document.getElementById('model-select').value;
    const messages = getMessages(); // 从历史获取

    const response = await fetch('https://api.deepseek.com/chat/completions', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${apiKey}`
        },
        body: JSON.stringify({
            model: model,
            messages: messages,
            stream: true
        })
    });

    // 处理流式响应...
}
```

- [ ] **Step 5: 提交**

```bash
git add cnblog/source/deepseek-chat/index.html
git commit -m "feat: add DeepSeek chat page UI and API integration"
```

---

## Task 3: 创建 Hexo 博客文章

**Files:**
- Create: `cnblog/source/_posts/deepseek-chat.md`

- [ ] **Step 1: 创建博客文章**

```markdown
---
title: DeepSeek V4 对话测试
date: 2026-04-26
categories:
  - 工具
permalink: /deepseek-chat/
---

# DeepSeek V4 对话测试

这是一个用于测试 DeepSeek V4 API 的对话页面。

## 功能

- 支持 deepseek-v4-flash 和 deepseek-v4-pro 模型
- 流式输出，实时显示响应
- 对话历史自动保存

## 使用方式

点击下方链接打开对话页面：

<a href="/cnblog/deepseek-chat/">进入对话</a>
```

- [ ] **Step 2: 提交**

```bash
git add cnblog/source/_posts/deepseek-chat.md
git commit -m "docs: add DeepSeek chat page introduction post"
```

---

## Task 4: 验证实现

**Files:**
- Modify: `cnblog/source/deepseek-chat/api-key.js` (用户替换自己的 key)

- [ ] **Step 1: 用户替换 API Key**

用户将自己的 DeepSeek API Key 进行 Base64 编码，替换 `api-key.js` 中的 `YOUR_BASE64_ENCODED_KEY_HERE`

- [ ] **Step 2: 本地测试**

```bash
cd cnblog
hexo server
# 访问 http://localhost:4000/cnblog/deepseek-chat/
```

- [ ] **Step 3: 验证功能**

1. 发送消息，确认 API 调用成功
2. 刷新页面，确认历史记录保留
3. 点击清空历史，确认重置正常

---

## 验收标准检查

| 标准 | 状态 |
|------|------|
| 页面可独立访问 | ✓ |
| API Key 不以明文出现在页面源码中 | ✓ (Base64 混淆) |
| 对话历史在刷新后保留 | ✓ (localStorage) |
| 流式输出正常显示 | ✓ (ReadableStream) |
| 清空历史后页面重置 | ✓ |
| 博客文章可跳转到对话页面 | ✓ |

---

## 执行选项

**1. Subagent-Driven (推荐)** - 每个 Task 分发独立 subagent 执行，快速迭代

**2. Inline Execution** - 在当前 session 执行，批量处理带检查点

选择哪个方式？
