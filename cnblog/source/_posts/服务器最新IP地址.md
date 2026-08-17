---
title: 服务器最新IP地址
date: 2025-05-29 14:39:04
tags:
    - IP
    - 服务器
    - 运维
categories:
    - 服务器
permalink: /posts/server-ip.html
---

## 当前服务器列表（实时更新，每分钟刷新一次）

<div id="server-ip">正在加载最新 IP 数据……</div>
<div id="server-ip-others"></div>

<script>
(function() {
  const API_URL = 'https://chenxiawen.cn/api/latest_ip';

  function renderTable(data) {
    const now = Date.now();
    const threshold = 2 * 60 * 1000;
    let html = '<table border="1" cellpadding="6" cellspacing="0" style="width:100%; margin-bottom: 20px;">'
             + '<thead><tr><th>服务器设备</th><th>当前公网 IP</th><th>最后更新时间</th><th>状态</th></tr></thead><tbody>';
    let count = 0;
    for (const name in data) {
      const item = data[name];
      const timeStr = (item.timestamp || '').replace(/-/g, '/');
      const ts = new Date(timeStr).getTime();
      if (now - ts <= threshold) {
        count++;
        html += '<tr>'
              + '<td><strong>' + name + '</strong></td>'
              + '<td><code style="font-size:1.1em; color:#0366d6;">' + item.ip + '</code></td>'
              + '<td>' + item.timestamp + '</td>'
              + '<td><span style="color:#28a745; font-weight:bold;">● 在线</span></td>'
              + '</tr>';
      }
    }
    if (count === 0) {
      html += '<tr><td colspan="4" style="text-align:center; color:#999;">暂无 2 分钟内活跃的服务器</td></tr>';
    }
    html += '</tbody></table>';
    return html;
  }

  function renderOthers(data) {
    const now = Date.now();
    const threshold = 2 * 60 * 1000;
    let html = '<h3 style="margin-top:20px;">2 分钟前更新的历史记录</h3>'
             + '<table border="1" cellpadding="6" cellspacing="0" style="width:100%;">'
             + '<thead><tr><th>服务器设备</th><th>历史记录 IP</th><th>最后更新时间</th><th>状态</th></tr></thead><tbody>';
    let count = 0;
    for (const name in data) {
      const item = data[name];
      const timeStr = (item.timestamp || '').replace(/-/g, '/');
      const ts = new Date(timeStr).getTime();
      if (now - ts > threshold) {
        count++;
        html += '<tr>'
              + '<td>' + name + '</td>'
              + '<td><code>' + item.ip + '</code></td>'
              + '<td>' + item.timestamp + '</td>'
              + '<td><span style="color:#6c757d;">离线 / 历史</span></td>'
              + '</tr>';
      }
    }
    if (count === 0) {
      html += '<tr><td colspan="4" style="text-align:center; color:#999;">无历史记录</td></tr>';
    }
    html += '</tbody></table>';
    return html;
  }

  async function fetchAndUpdate() {
    const elIn = document.getElementById('server-ip');
    const elOut = document.getElementById('server-ip-others');
    if (!elIn) return;
    try {
      const resp = await fetch(API_URL, { cache: 'no-store' });
      if (!resp.ok) throw new Error('HTTP ' + resp.status + ' ' + resp.statusText);
      const json = await resp.json();
      elIn.innerHTML = renderTable(json);
      if (elOut) elOut.innerHTML = renderOthers(json);
    } catch (err) {
      if (elIn) elIn.innerHTML = '<div style="color:red; padding:10px;">获取 IP 失败：' + err.message + '</div>';
    }
  }

  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', fetchAndUpdate);
  } else {
    fetchAndUpdate();
  }

  document.addEventListener('pjax:complete', fetchAndUpdate);

  if (window._serverIpTimer) {
    clearInterval(window._serverIpTimer);
  }
  window._serverIpTimer = setInterval(fetchAndUpdate, 60 * 1000);
})();
</script>