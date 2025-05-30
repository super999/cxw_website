---
title: 服务器最新IP地址
date: 2025-05-29 14:39:04
tags:
categories:
    - 服务器
permalink: /posts/server-ip.html
---

## 当前服务器列表（实时更新，每分钟刷新一次）

<!-- ① 在这里插入占位元素 -->
<div id="server-ip">加载中……</div>
<div id="server-ip-others">加载中……</div>

<!-- ② 然后再放你的脚本 -->

<script>
// 【1】你的接口 URL，注意如果跨域需要服务器允许 CORS
const API_URL = 'https://chenxiawen.cn/api/latest_ip';
// 【2】将接口返回的 JSON 转为 HTML 表格
function renderTable(data) {
  const now = Date.now();
  const threshold = 2 * 60 * 1000; // 2 分钟（毫秒）
  let html = '<table border="1" cellpadding="6" cellspacing="0">'
           + '<tr><th>服务器</th><th>IP</th><th>最后更新时间</th></tr>';
  for (const name in data) {
    const item = data[name];
    const ts = new Date(item.timestamp).getTime();
    if (now - ts <= threshold) {
      html += `<tr>
                 <td>${name}</td>
                 <td>${item.ip}</td>
                 <td>${item.timestamp}</td>
               </tr>`;
    }
  }
  html += '</table>';
  console.log('渲染的 HTML:', html); // 调试输出
  // 返回生成的 HTML 字符串
  return html;
}
// 新增：渲染 2 分钟前更新的那些条目
function renderOthers(data) {
  const now = Date.now();
  const threshold = 2 * 60 * 1000;
  let html = '<h3>2 分钟前更新的服务器</h3>'
           + '<table border="1" cellpadding="6" cellspacing="0">'
           + '<tr><th>服务器</th><th>IP</th><th>最后更新时间</th></tr>';
  for (const name in data) {
    const item = data[name];
    const ts = new Date(item.timestamp).getTime();
    if (now - ts > threshold) {
      html += `<tr>
                 <td>${name}</td>
                 <td>${item.ip}</td>
                 <td>${item.timestamp}</td>
               </tr>`;
    }
  }
  html += '</table>';
  return html;
}
// 【3】拉取接口并更新页面
async function fetchAndUpdate() {
  try {
    const resp = await fetch(API_URL, { cache: 'no-store' });
    if (!resp.ok) throw new Error(resp.statusText);
    const json = await resp.json();
    const htmlIn = renderTable(json);
    const htmlOut = renderOthers(json);
    document.getElementById('server-ip').innerHTML = htmlIn;
    document.getElementById('server-ip-others').innerHTML = htmlOut;
  } catch (err) {
    document.getElementById('server-ip').innerHTML = '获取数据失败：' + err.message;
    document.getElementById('server-ip-others').innerHTML = '';
  }
}
// 页面加载完成后，立即拉取一次并启动定时器
document.addEventListener('DOMContentLoaded', () => {
  fetchAndUpdate();
  // 每 60 秒刷新一次，按需修改间隔（毫秒）
  setInterval(fetchAndUpdate, 60 * 1000);
});
</script>