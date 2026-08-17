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

---

## ⚙️ 后台服务与更新机制说明

为了防止以后忘记维护路径，将整个 IP 上报与展示系统的架构及维护方式记录如下：

### 1. 源码项目信息
* **项目名称**：`IpNotifier`
* **本地源码路径**：`D:\python_workspace\IpNotifier`
* **GitHub 仓库**：[https://github.com/super999/IP-Notifier](https://github.com/super999/IP-Notifier)

---

### 2. 更新机制说明

1. **客户端（各路由器）**：
   * 后台运行 Docker 容器 `ip-notifier-client`。
   * 每 60 秒向 `http://ip.3322.net` 查询当前宽带拨号的公网 IP。
   * 携带统一的 `X-API-Key` 密钥向服务端 `https://chenxiawen.cn/api/update_ip` 发送 POST 上报请求。
2. **服务端（腾讯云 `106.****.251`）**：
   * 运行 Nginx (OpenResty) 与 Docker 容器 `ip-update-server`（Flask 框架，监听 5000 端口）。
   * Nginx 配置 `/nginx_data/nginx/conf.d/chenxiawen.cn.conf` 中将 `/api/` 反向代理至本机 5000 端口（`http://172.21.0.1:5000`）。
   * 服务端校验 API Key 后更新内存并在当前目录的 `latest_ip.txt` 中持久化保存。
   * 日志采用 `RotatingFileHandler` 自动滚动覆盖，单文件上限 20MB，保留 5 个备份（总占用最大 100MB）。
3. **前端渲染（本博客页面）**：
   * 页面加载后自动通过 `GET https://chenxiawen.cn/api/latest_ip` 获取所有服务器最新记录。
   * **在线判定**：最后更新时间在 **2 分钟以内**的标记为在线状态，超过 2 分钟的归入历史记录表格。

---

### 3. 常见维护与排查命令

若页面 IP 停止更新或需要重启维护，可按以下命令快速定位：

#### 检查/重启服务端 (腾讯云 `106.****.251`)
```bash
# 查看服务端实时接收日志
ssh super***@106.****.251 "sudo docker logs -f ip-update-server"

# 重启服务端
ssh super***@106.****.251 "sudo docker restart ip-update-server"

# 检查 Nginx 转发配置并重载
ssh super***@106.****.251 "sudo docker exec openresty nginx -t && sudo docker exec openresty nginx -s reload"
```

#### 检查/重启车库路由器客户端 (`192.168.9.1`)
```bash
# 目录: /docker_app/ip_notifier
ssh root@192.168.9.1 "docker logs -f ip-notifier-client"
ssh root@192.168.9.1 "docker restart ip-notifier-client"
```

#### 检查/重启家里路由器客户端 (`cxwl999.eicp.net`)
```bash
# 目录: /docker_app/ip-notifier/client
ssh root@cxwl999.eicp.net "docker logs -f ip-notifier-client"
ssh root@cxwl999.eicp.net "docker restart ip-notifier-client"
```