<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<!-- 不向上游页面泄露 referrer，避免目标页根据来源注入广告 -->
<meta name="referrer" content="no-referrer">
<title>正在进入…</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  html, body { height: 100%; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
    background: #f5f6fa;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .box { text-align: center; padding: 24px; max-width: 420px; width: 100%; }
  .spinner {
    width: 42px; height: 42px;
    margin: 0 auto 18px;
    border: 4px solid #e3e6f0;
    border-top-color: #4a6cf7;
    border-radius: 50%;
    animation: spin .8s linear infinite;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
  .title { font-size: 17px; color: #333; margin-bottom: 8px; }
  .status { font-size: 13px; color: #999; line-height: 1.6; word-break: break-all; }
  .btn {
    display: none;
    margin: 18px auto 0;
    padding: 11px 34px;
    font-size: 15px;
    color: #fff;
    background: #4a6cf7;
    border: none;
    border-radius: 22px;
    cursor: pointer;
  }
  .btn.show { display: inline-block; }
</style>
</head>
<body>
<div class="box">
  <div class="spinner" id="spinner"></div>
  <div class="title">正在进入…</div>
  <div class="status" id="status">正在解析跳转地址…</div>
  <button class="btn" id="btn">手动继续</button>
</div>

<script>
(function () {
  // 短链 code，可通过 ?code=xxx 覆盖，默认 smyz
  var params = new URLSearchParams(location.search);
  var code = (params.get('code') || 'smyz').replace(/[^A-Za-z0-9_-]/g, '');
  var SHORT_URL = 'https://s.c1ns.cn/' + code;

  var statusEl = document.getElementById('status');
  var btnEl = document.getElementById('btn');
  var spinnerEl = document.getElementById('spinner');

  function setStatus(text) { statusEl.textContent = text; }

  // 展示错误并提供手动兜底（走原短链，保证永不卡死）
  function fail(msg) {
    statusEl.innerHTML = (msg || '解析失败') + '，即将通过原始链接进入<br>或点击下方按钮手动继续';
    btnEl.textContent = '手动继续（原始链接）';
    btnEl.onclick = function () { location.replace(SHORT_URL); };
    btnEl.className = 'btn show';
    spinnerEl.style.display = 'none';
    setTimeout(function () { location.replace(SHORT_URL); }, 3000);
  }

  // 从短链返回的 HTML 中提取 wx.c1n.cn/wx/app/* 真实地址
  function extractTarget(html) {
    var m = html.match(/https?:\/\/wx\.c1n\.cn\/wx\/app\/[A-Za-z0-9_-]+/);
    return m ? m[0] : null;
  }

  // 带超时的 fetch
  function fetchText(url, timeout) {
    var ctrl = new AbortController();
    var timer = setTimeout(function () { ctrl.abort(); }, timeout);
    return fetch(url, { credentials: 'omit', signal: ctrl.signal })
      .then(function (res) {
        if (!res.ok) throw new Error('HTTP ' + res.status);
        return res.text();
      })
      .finally(function () { clearTimeout(timer); });
  }

  // CORS 代理兜底列表（万一官方关闭跨域时使用）
  function proxyText(url) {
    var proxies = [
      'https://api.allorigins.win/raw?url=' + encodeURIComponent(url),
      'https://api.codetabs.com/v1/proxy?quest=' + encodeURIComponent(url)
    ];
    return proxies.reduce(function (p, proxyUrl) {
      return p.catch(function () { return fetchText(proxyUrl, 6000); });
    }, Promise.reject());
  }

  function go(target) {
    setStatus('即将进入：' + target);
    location.replace(target);
  }

  function resolve() {
    // 直接抓取短链页面（该接口 CORS 开放），提取真实目标
    fetchText(SHORT_URL, 5000)
      .catch(function () { return proxyText(SHORT_URL); })   // 跨域被禁时走代理兜底
      .then(function (html) {
        var target = extractTarget(html);
        if (target) {
          go(target);
        } else {
          fail('未解析到目标地址');
        }
      })
      .catch(function () { fail('网络异常'); });
  }

  resolve();
})();
</script>
</body>
</html>
