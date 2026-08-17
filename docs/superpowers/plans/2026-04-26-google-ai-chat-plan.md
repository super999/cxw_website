# Google AI Chat Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 实现 Google AI 对话页面，支持文字对话和图片上传分析

**Architecture:** 基于 DeepSeek Chat 的实现模式，创建独立的静态 HTML 页面，使用 Google Gemini API，通过 SSE 流式接收响应

**Tech Stack:** Vanilla JS + CSS, Google Gemini API, localStorage

---

## File Structure

```
cnblog/source/google-ai-chat/
├── index.html      # 主页面（HTML + CSS + JS）
└── api-key.js      # Base64 编码的 API Key
```

---

## Task 1: 创建基础页面结构和样式

**Files:**
- Create: `cnblog/source/google-ai-chat/index.html`
- Create: `cnblog/source/google-ai-chat/api-key.js`

- [ ] **Step 1: 创建 google-ai-chat 目录**

Run: `mkdir -p cnblog/source/google-ai-chat`

- [ ] **Step 2: 创建 api-key.js 文件**

```javascript
// cnblog/source/google-ai-chat/api-key.js
// Base64 编码的 Google AI Studio API Key
const apiKey = atob("YOUR_BASE64_ENCODED_API_KEY");
```

- [ ] **Step 3: 创建 index.html 基础结构**

```html
<!-- cnblog/source/google-ai-chat/index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Google AI 对话</title>
  <style>
    /* CSS 样式 - Google 主题 */
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      font-family: 'Google Sans', 'Segoe UI', sans-serif;
      background: #FFFFFF;
      color: #202124;
      height: 100vh;
      display: flex;
      flex-direction: column;
    }
    /* Header */
    .header {
      background: linear-gradient(135deg, #4285F4 0%, #34A853 100%);
      padding: 16px 24px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      color: white;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    }
    .header h1 {
      font-size: 20px;
      font-weight: 500;
    }
    .header-controls {
      display: flex;
      gap: 12px;
      align-items: center;
    }
    select, button {
      padding: 8px 16px;
      border-radius: 4px;
      border: none;
      cursor: pointer;
      font-size: 14px;
    }
    select {
      background: white;
      color: #202124;
    }
    .btn-clear {
      background: rgba(255,255,255,0.2);
      color: white;
    }
    .btn-clear:hover {
      background: rgba(255,255,255,0.3);
    }
    /* Chat Container */
    .chat-container {
      flex: 1;
      overflow-y: auto;
      padding: 24px;
    }
    /* Messages */
    .message {
      max-width: 70%;
      margin-bottom: 16px;
      padding: 12px 16px;
      border-radius: 16px;
      line-height: 1.5;
      animation: fadeIn 0.3s ease;
    }
    .message.user {
      background: linear-gradient(135deg, #4285F4 0%, #34A853 100%);
      color: white;
      margin-left: auto;
      border-bottom-right-radius: 4px;
    }
    .message.assistant {
      background: #F1F3F4;
      color: #202124;
      border-bottom-left-radius: 4px;
    }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: translateY(0); }
    }
    /* Input Area */
    .input-area {
      border-top: 1px solid #DADCE0;
      padding: 16px 24px;
      display: flex;
      gap: 12px;
      align-items: flex-end;
    }
    .input-area textarea {
      flex: 1;
      padding: 12px 16px;
      border: 1px solid #DADCE0;
      border-radius: 24px;
      resize: none;
      font-size: 14px;
      min-height: 48px;
      max-height: 120px;
      font-family: inherit;
    }
    .input-area textarea:focus {
      outline: none;
      border-color: #4285F4;
    }
    .btn-upload {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      background: #F1F3F4;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
    }
    .btn-upload:hover {
      background: #E8EAED;
    }
    .btn-send {
      width: 48px;
      height: 48px;
      border-radius: 50%;
      background: linear-gradient(135deg, #4285F4 0%, #34A853 100%);
      color: white;
      font-size: 20px;
    }
    .btn-send:hover {
      opacity: 0.9;
    }
    .btn-send:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
    /* Empty State */
    .empty-state {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100%;
      color: #5F6368;
    }
    .empty-state .icon {
      font-size: 64px;
      margin-bottom: 16px;
    }
    /* Image Preview */
    .image-preview {
      position: relative;
      display: inline-block;
      margin: 8px 0;
    }
    .image-preview img {
      max-width: 200px;
      max-height: 200px;
      border-radius: 8px;
    }
    .image-preview .remove-btn {
      position: absolute;
      top: -8px;
      right: -8px;
      width: 24px;
      height: 24px;
      border-radius: 50%;
      background: #EA4335;
      color: white;
      border: none;
      cursor: pointer;
      font-size: 14px;
    }
    /* Hidden file input */
    #fileInput {
      display: none;
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>Google AI 对话</h1>
    <div class="header-controls">
      <select id="modelSelect">
        <option value="gemini-2.0-flash">gemini-2.0-flash</option>
        <option value="gemini-3-flash-preview">gemini-3-flash-preview</option>
      </select>
      <button class="btn-clear" onclick="clearHistory()">清空历史</button>
    </div>
  </div>

  <div class="chat-container" id="chatContainer">
    <div class="empty-state" id="emptyState">
      <div class="icon">🤖</div>
      <p>开始和 Google AI 对话吧</p>
    </div>
  </div>

  <div class="input-area">
    <input type="file" id="fileInput" accept="image/*">
    <button class="btn-upload" onclick="document.getElementById('fileInput').click()">📷</button>
    <textarea id="messageInput" placeholder="输入消息..." rows="1"></textarea>
    <button class="btn-send" id="sendBtn" onclick="sendMessage()">➤</button>
  </div>

  <script src="api-key.js"></script>
  <script>
    // JavaScript 实现 - 待完成
  </script>
</body>
</html>
```

- [ ] **Step 4: Commit 基础结构**

```bash
git add cnblog/source/google-ai-chat/
git commit -m "feat: scaffold google-ai-chat page structure"
```

---

## Task 2: 实现核心 JavaScript 逻辑（文字对话）

**Files:**
- Modify: `cnblog/source/google-ai-chat/index.html`（添加 JS 逻辑）

- [ ] **Step 1: 添加核心 JavaScript 变量和初始化函数**

在 `<script>` 标签中添加：

```javascript
const chatContainer = document.getElementById('chatContainer');
const messageInput = document.getElementById('messageInput');
const sendBtn = document.getElementById('sendBtn');
const modelSelect = document.getElementById('modelSelect');
const fileInput = document.getElementById('fileInput');
const emptyState = document.getElementById('emptyState');
let attachedImage = null;
let isStreaming = false;

function init() {
  loadHistory();
  messageInput.addEventListener('input', autoResize);
  messageInput.addEventListener('keydown', handleKeyDown);
  fileInput.addEventListener('change', handleFileSelect);
  initDragDrop();
}

function autoResize() {
  messageInput.style.height = 'auto';
  messageInput.style.height = Math.min(messageInput.scrollHeight, 120) + 'px';
}

function handleKeyDown(e) {
  if (e.key === 'Enter' && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
}
```

- [ ] **Step 2: 添加历史记录管理函数**

```javascript
function loadHistory() {
  const history = JSON.parse(localStorage.getItem('google-ai-chat-history') || '[]');
  if (history.length === 0) return;
  emptyState.style.display = 'none';
  history.forEach(msg => renderMessage(msg.role, msg.content, msg.timestamp, false));
  scrollToBottom();
}

function saveHistory(role, content) {
  let history = JSON.parse(localStorage.getItem('google-ai-chat-history') || '[]');
  history.push({ role, content, timestamp: Date.now() });
  if (history.length > 50) history = history.slice(-50);
  localStorage.setItem('google-ai-chat-history', JSON.stringify(history));
}

function clearHistory() {
  localStorage.removeItem('google-ai-chat-history');
  chatContainer.querySelectorAll('.message').forEach(el => el.remove());
  emptyState.style.display = 'flex';
}
```

- [ ] **Step 3: 添加消息渲染函数**

```javascript
function renderMessage(role, content, timestamp, animate = true) {
  const div = document.createElement('div');
  div.className = `message ${role}`;

  if (role === 'user') {
    div.innerHTML = formatContent(content);
    if (attachedImage) {
      const imgPreview = document.createElement('div');
      imgPreview.className = 'image-preview';
      imgPreview.innerHTML = `<img src="${attachedImage.data}" alt="attached">`;
      div.appendChild(imgPreview);
    }
  } else {
    div.innerHTML = formatContent(content);
  }

  if (!animate) div.style.animation = 'none';
  chatContainer.appendChild(div);
  return div;
}

function formatContent(content) {
  // HTML 转义
  let html = content
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');

  // 代码块
  html = html.replace(/```(\w*)\n?([\s\S]*?)```/g, (match, lang, code) => {
    return `<pre><code class="language-${lang}">${code.trim()}</code></pre>`;
  });

  // 行内代码
  html = html.replace(/`([^`]+)`/g, '<code>$1</code>');

  // 换行
  html = html.replace(/\n/g, '<br>');

  return html;
}

function scrollToBottom() {
  chatContainer.scrollTop = chatContainer.scrollHeight;
}
```

- [ ] **Step 4: 添加发送消息和 API 调用函数**

```javascript
async function sendMessage() {
  const content = messageInput.value.trim();
  if (!content && !attachedImage) return;
  if (isStreaming) return;

  emptyState.style.display = 'none';
  const userContent = content;
  const imageData = attachedImage;

  // 渲染用户消息
  renderMessage('user', userContent);
  saveHistory('user', userContent);

  // 清空输入
  messageInput.value = '';
  messageInput.style.height = 'auto';
  if (attachedImage) {
    attachedImage = null;
    fileInput.value = '';
  }

  // 显示加载状态
  const loadingDiv = document.createElement('div');
  loadingDiv.className = 'message assistant loading';
  loadingDiv.innerHTML = '<span>思考中...</span>';
  chatContainer.appendChild(loadingDiv);
  scrollToBottom();

  try {
    await callGeminiAPI(userContent, imageData, loadingDiv);
  } catch (error) {
    loadingDiv.innerHTML = `<span style="color: #EA4335;">错误: ${error.message}</span>`;
  }
}

async function callGeminiAPI(text, imageData, loadingDiv) {
  isStreaming = true;
  sendBtn.disabled = true;

  const model = modelSelect.value;
  const parts = [{ text }];
  if (imageData) {
    parts.push({
      inline_data: {
        mime_type: imageData.mimeType,
        data: imageData.data
      }
    });
  }

  const response = await fetch(
    `https://generativelanguage.googleapis.com/v1beta/models/${model}:streamGenerateContent?key=${apiKey}&alt=sse`,
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        contents: [{ parts }],
        generationConfig: {
          temperature: 1.0,
          topP: 0.95,
          topK: 40,
          maxOutputTokens: 8192
        }
      })
    }
  );

  if (!response.ok) {
    throw new Error(`API 请求失败: ${response.status}`);
  }

  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  let fullContent = '';
  loadingDiv.innerHTML = '<span class="typing-indicator"></span>';

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n');

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') continue;

        try {
          const json = JSON.parse(data);
          const text = json.candidates?.[0]?.content?.parts?.[0]?.text;
          if (text) {
            fullContent += text;
            loadingDiv.innerHTML = formatContent(fullContent);
            scrollToBottom();
          }
        } catch (e) {
          // 忽略解析错误
        }
      }
    }
  }

  saveHistory('assistant', fullContent);
  isStreaming = false;
  sendBtn.disabled = false;
}
```

- [ ] **Step 5: 调用 init 函数**

在 script 末尾添加：

```javascript
init();
```

- [ ] **Step 6: 测试页面功能**

由于这是静态页面，需要先运行 Hexo 本地服务器测试。

Run: `cd cnblog && npm run server`（或 `hexo server`）

在浏览器中打开页面，测试：
1. 发送文字消息
2. 检查是否有 SSE 流式响应
3. 验证历史记录保存

- [ ] **Step 7: Commit**

```bash
git add cnblog/source/google-ai-chat/index.html
git commit -m "feat: implement basic chat functionality with Gemini API"
```

---

## Task 3: 实现图片上传功能

**Files:**
- Modify: `cnblog/source/google-ai-chat/index.html`

- [ ] **Step 1: 添加图片选择处理函数**

在 JavaScript 中添加：

```javascript
function handleFileSelect(e) {
  const file = e.target.files[0];
  if (!file) return;

  if (!file.type.startsWith('image/')) {
    alert('请选择图片文件');
    return;
  }

  const reader = new FileReader();
  reader.onload = (event) => {
    attachedImage = {
      data: event.target.result.split(',')[1], // 移除 data:image/...;base64, 前缀
      mimeType: file.type
    };
    // 可以选择显示预览或提示用户
    messageInput.focus();
  };
  reader.readAsDataURL(file);
}
```

- [ ] **Step 2: 添加拖拽上传功能**

```javascript
function initDragDrop() {
  const container = chatContainer;

  container.addEventListener('dragover', (e) => {
    e.preventDefault();
    container.style.background = '#F8F9FA';
  });

  container.addEventListener('dragleave', (e) => {
    e.preventDefault();
    container.style.background = '';
  });

  container.addEventListener('drop', (e) => {
    e.preventDefault();
    container.style.background = '';

    const files = e.dataTransfer.files;
    if (files.length > 0 && files[0].type.startsWith('image/')) {
      fileInput.files = files;
      handleFileSelect({ target: fileInput });
    }
  });
}
```

- [ ] **Step 3: 测试图片上传功能**

1. 点击上传按钮选择图片
2. 拖拽图片到聊天区域
3. 发送带图片的消息
4. 验证 API 返回的图片分析结果

- [ ] **Step 4: Commit**

```bash
git add cnblog/source/google-ai-chat/index.html
git commit -m "feat: add image upload support"
```

---

## Task 4: 添加 Typing Indicator 样式和最终优化

**Files:**
- Modify: `cnblog/source/google-ai-chat/index.html`

- [ ] **Step 1: 添加 Typing Indicator CSS**

在 `<style>` 中添加：

```css
/* Typing Indicator */
.typing-indicator {
  display: inline-block;
}
.typing-indicator::after {
  content: '';
  animation: typing 1.4s infinite;
}
.typing-indicator::after {
  content: '...';
}
@keyframes typing {
  0%, 20% { content: '.'; }
  40% { content: '..'; }
  60%, 100% { content: '...'; }
}

.message pre {
  background: #F8F9FA;
  padding: 12px;
  border-radius: 8px;
  overflow-x: auto;
  margin: 8px 0;
}
.message code {
  background: #F1F3F4;
  padding: 2px 6px;
  border-radius: 4px;
  font-family: 'Consolas', monospace;
  font-size: 13px;
}
.message pre code {
  background: none;
  padding: 0;
}
```

- [ ] **Step 2: 测试并修复任何问题**

- [ ] **Step 3: Commit**

```bash
git add cnblog/source/google-ai-chat/index.html
git commit -m "feat: add typing indicator and code styling"
```

---

## Task 5: 创建博客文章入口

**Files:**
- Create: `cnblog/source/_posts/google-ai-chat.md`

- [ ] **Step 1: 创建博客文章**

```markdown
---
title: Google AI 对话
date: 2026-04-26
layout: false
---

# Google AI 对话

体验 Google Gemini AI 的强大能力，支持文字对话和图片分析。

[打开 Google AI 对话页面](../google-ai-chat/)
```

- [ ] **Step 2: Commit**

```bash
git add cnblog/source/_posts/google-ai-chat.md
git commit -m "docs: add google-ai-chat blog post"
```

---

## 验收清单

每个 Task 完成后验证：

- [ ] Task 1: 页面可正常加载，无 console error
- [ ] Task 2: 文字消息可发送并收到流式响应
- [ ] Task 3: 图片上传和拖拽功能正常
- [ ] Task 4: 样式和动画正常
- [ ] Task 5: 博客文章可访问

---

## 执行选项

**1. Subagent-Driven (推荐)** - 每个 Task 由独立的 subagent 执行，任务间有检查点

**2. Inline Execution** - 在当前 session 中按批次执行任务

选择哪个方式？
