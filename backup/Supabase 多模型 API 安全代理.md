## 前情提要：

询问了 T 佬关于部署 GPT-Load 设置动态代理之类，然后陷入了误区，一直想着给容器配置代理，现在终于找到方法：给 api 代理不就行了，代码质量不保证，有大佬更好的话可以发出来迭代一下

> 经过我深入的研究和体验，某些场景 Supabase 比 Deno 还要好用。  
> 特别是针对上游会根据 ip 指纹等进行速率控制的服务。  
> Supabase 在脚本里面屏蔽相关 header 后，出口能够完全隐藏 ip 身份，每次请求都生成不一样的 ipv6 地址。让上游无法判断频次，从而实现高 RPM。

## 本贴子得到以下贴子帮助 (按代码时间迭代排序)：

> [deno 多模型 API 安全代理](https://linux.do/t/topic/278306) 
> import { serve } from "https://deno.land/std/http/server.ts"; const apiMapping = { '/discord': 'https://discord.com/api', '/telegram': 'https://api.telegram.org', '/openai': 'https://api.openai…

> [deno 安全反代网站 && 解决 CORS 限制 - 开发调优 / 开发调优，Lv1 - LINUX DO](https://linux.do/t/topic/320012)

> [Deno 大模型 API 代理修改版【增加 gemini-nothink】](https://linux.do/t/topic/685206)
> 结合两个大佬的贴放到 cursor 优化了一下发出来： [https://linux.do/t/topic/685038](https://linux.do/t/topic/685038) 增加了 gemini-nothink 代理 修复 cluade 请求头报错 import {serve} from "https://deno.land/std/http/server.ts"; const apiMapping = { "/discord": "https…

> [Deno 多模型 API 代理修改 2.0 版 - 开发调优 / 开发调优，Lv1 - LINUX DO](https://linux.do/t/topic/685216)

[【T 佬】Deno 多代理防封脚本！不过还需要时间验证。](https://linux.do/t/topic/726100/41) 
> // Supabase Edge Functions API 代理 - 独立版本 // 一个简单、安全的 HTTPS API 反向代理，基于 Supabase Edge Functions 构建 /// <reference types="https://esm.sh/@supabase/functions-js/src/edge-runtime.d.ts" /> // CORS 配置 con…

> [Deno Deploy 迁移到 Supabase Edge Function 全指北 - 文档共建 / 文档共建，Lv1 - LINUX DO](https://linux.do/t/topic/726205)

```ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
// 1. API 代理映射表
const proxies = {
  discord: "discord.com/api",
  telegram: "api.telegram.org",
  httpbin: "httpbin.org",
  openai: "api.openai.com",
  claude: "api.anthropic.com",
  gemini: "generativelanguage.googleapis.com",
  gemininothink: "generativelanguage.googleapis.com",
  meta: "www.meta.ai/api",
  groq: "api.groq.com/openai",
  xai: "api.x.ai",
  cohere: "api.cohere.ai",
  huggingface: "api-inference.huggingface.co",
  together: "api.together.xyz",
  novita: "api.novita.ai",
  portkey: "api.portkey.ai",
  fireworks: "api.fireworks.ai",
  targon: "api.targon.com",
  openrouter: "openrouter.ai/api"
};
// 2. 增强的 Header 黑名单
const BlacklistedHeaders = new Set([
  "cf-connecting-ip",
  "cf-ipcountry",
  "cf-ray",
  "cf-visitor",
  "cf-worker",
  "cdn-loop",
  "cf-ew-via",
  "baggage",
  "sb-request-id",
  "x-amzn-trace-id",
  "x-forwarded-for",
  "x-forwarded-host",
  "x-forwarded-proto",
  "x-forwarded-server",
  "x-real-ip",
  "x-original-host",
  "forwarded",
  "via",
  "referer",
  "x-request-id",
  "x-correlation-id",
  "x-trace-id"
]);
// 3. 日志记录辅助函数
function logRequest(method, pathname, targetUrl, status) {
  const timestamp = new Date().toISOString();
  const statusInfo = status ? ` [${status}]` : '';
  const target = targetUrl ? ` -> ${targetUrl}` : '';
  console.log(`[${timestamp}] ${method} ${pathname}${target}${statusInfo}`);
}
// 4. 错误类型识别函数
function categorizeError(error) {
  const errorMessage = error.message.toLowerCase();
  if (error.name === 'AbortError' || errorMessage.includes('timeout')) {
    return {
      type: 'TIMEOUT',
      message: 'Request timeout - the target service took too long to respond',
      status: 504
    };
  }
  if (errorMessage.includes('network') || errorMessage.includes('fetch')) {
    return {
      type: 'NETWORK',
      message: 'Network error - unable to reach the target service',
      status: 502
    };
  }
  if (errorMessage.includes('dns') || errorMessage.includes('name resolution')) {
    return {
      type: 'DNS',
      message: 'DNS resolution failed - unable to resolve target hostname',
      status: 502
    };
  }
  if (errorMessage.includes('connection refused') || errorMessage.includes('connect')) {
    return {
      type: 'CONNECTION',
      message: 'Connection refused - target service is not accepting connections',
      status: 503
    };
  }
  if (errorMessage.includes('ssl') || errorMessage.includes('tls') || errorMessage.includes('certificate')) {
    return {
      type: 'SSL',
      message: 'SSL/TLS error - certificate or encryption issue',
      status: 502
    };
  }
  return {
    type: 'UNKNOWN',
    message: `Unexpected error: ${error.message}`,
    status: 500
  };
}
// 5. 创建标准错误响应
function createErrorResponse(message, status, details) {
  const errorBody = JSON.stringify({
    error: message,
    status,
    timestamp: new Date().toISOString(),
    ...details && {
      details
    }
  });
  return new Response(errorBody, {
    status,
    headers: {
      "Content-Type": "application/json; charset=utf-8",
      "Access-Control-Allow-Origin": "*"
    }
  });
}
// 6. 主服务函数
serve(async (req)=>{
  const url = new URL(req.url);
  const { pathname, search } = url;
  logRequest(req.method, pathname);
  // OPTIONS 请求处理
  if (req.method === "OPTIONS") {
    return new Response(null, {
      status: 204,
      headers: {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS, HEAD, PATCH",
        "Access-Control-Allow-Headers": "Content-Type, Authorization, X-Requested-With, anthropic-version, x-api-key"
      }
    });
  }
  // 恢复到第一版的路径解析逻辑
  const pathParts = pathname.split("/");
  // 验证路径格式（适配第一版逻辑）
  // 假设格式为 /<base_path>/<service_alias>/...  (e.g., /api/openai/...)
  // pathParts will be ['', <base_path>, <service_alias>, ...]
  if (pathParts.length < 3) {
    return createErrorResponse("Invalid path format. Expected format like: /api/{service}/{path}", 400, `Available services: ${Object.keys(proxies).join(', ')}`);
  }
  const targetAlias = pathParts[2]; // 获取服务别名
  const targetHost = proxies[targetAlias];
  console.log(`服务别名: ${targetAlias}, 目标主机: ${targetHost}`);
  // 验证服务映射
  if (!targetHost) {
    console.error(`服务映射未找到: 客户端尝试访问别名 '${targetAlias}'`);
    return createErrorResponse(`Service alias '${targetAlias}' not found`, 404, `Available services: ${Object.keys(proxies).join(', ')}`);
  }
  // 构建目标路径和URL
  const targetPath = pathParts.slice(3).join("/");
  const targetUrl = `https://${targetHost}/${targetPath}${search}`;
  console.log(`目标路径: ${targetPath}`);
  console.log(`最终目标URL: ${targetUrl}`);
  try {
    // 处理请求头
    const forwardedHeaders = new Headers();
    for (const [key, value] of req.headers.entries()){
      const lowerKey = key.toLowerCase();
      if (!BlacklistedHeaders.has(lowerKey) && !lowerKey.startsWith("sec-ch-ua")) {
        forwardedHeaders.set(key, value);
      }
    }
    // 设置必要的headers
    forwardedHeaders.set("Host", targetHost);
    forwardedHeaders.set("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36");
    // Claude API 特殊处理
    if (targetAlias === "claude" && !forwardedHeaders.has("anthropic-version")) {
      forwardedHeaders.set("anthropic-version", "2023-06-01");
    }
    // 处理请求体
    let requestBody = null;
    if (req.method !== "GET" && req.method !== "HEAD") {
      // Gemini NoThink 特殊处理
      if (targetAlias === "gemininothink" && req.method === "POST" && req.headers.get("content-type")?.includes("application/json")) {
        try {
          const bodyText = await req.text();
          const bodyJson = JSON.parse(bodyText);
          bodyJson.generationConfig = {
            ...bodyJson.generationConfig || {},
            thinkingConfig: {
              thinkingBudget: 0
            }
          };
          requestBody = JSON.stringify(bodyJson);
          forwardedHeaders.set("content-type", "application/json");
        } catch (e) {
          console.error(`修改gemininothink请求体失败: ${e.message}`);
          return createErrorResponse("Invalid JSON format in request body", 400);
        }
      } else {
        requestBody = req.body;
      }
    }
    console.log(`发起请求到: ${targetUrl}`);
    // 发起代理请求
    const apiResponse = await fetch(targetUrl, {
      method: req.method,
      headers: forwardedHeaders,
      body: requestBody,
      redirect: "manual"
    });
    console.log(`API响应状态: ${apiResponse.status}`);
    logRequest(req.method, pathname, targetUrl, apiResponse.status);
    // 设置CORS响应头
    const responseHeaders = new Headers(apiResponse.headers);
    responseHeaders.set("Access-Control-Allow-Origin", "*");
    responseHeaders.set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS, HEAD, PATCH");
    responseHeaders.set("Access-Control-Allow-Headers", "Content-Type, Authorization, X-Requested-With, anthropic-version, x-api-key");
    return new Response(apiResponse.body, {
      status: apiResponse.status,
      headers: responseHeaders
    });
  } catch (error) {
    // 详细的错误分类和日志记录
    const errorInfo = categorizeError(error);
    console.error(`[${errorInfo.type}] API代理请求失败:`);
    console.error(`  目标URL: ${targetUrl}`);
    console.error(`  错误详情: ${error.message}`);
    console.error(`  错误堆栈: ${error.stack}`);
    logRequest(req.method, pathname, targetUrl, errorInfo.status);
    return createErrorResponse(errorInfo.message, errorInfo.status, `Error type: ${errorInfo.type}`);
  }
});
console.log("🚀 API代理服务器已启动");
console.log(`📋 支持的服务: ${Object.keys(proxies).join(', ')}`);
console.log("📖 使用方法: /<base_path>/{service}/{path} (例如: /api/openai/v1/chat/completions)");
```

## 最新版本的功能和优点：

<img width="1060" height="414" alt="Image" src="https://github.com/user-attachments/assets/b0ab1a72-eb45-4dd4-bedd-a376d8e84ccd" />

## 最后感谢：

[@F-droid](/u/f-droid) [@fangyuan99](/u/fangyuan99) [@minij](/u/minij) [@hanka](/u/hanka) [@dext7r](/u/dext7r) [@Neuroplexus](/u/neuroplexus) [@tbphp](/u/tbphp) [@dabuliu](/u/tbphp)   `@Gemini2.5pro` `@claude-4-sonnet`  
_不用感谢我，我全靠 AI_ 
---
好吧，**还是感谢我吧（点赞！认可！）**，发现 bug 又又又迭代了好几个版本，又过了两个钟头了，现在凌晨 4 点了，我该睡觉了，刚说好的早睡

部署教程：N 佬的帖子有 
使用方法：https://<项目 ID>.supabase.co/functions/v1/< 函数名 >/< 自定义路径 > 
体验测试地址： [Supabase 多模型 API 安全代理体验和测试 - 开发调优 / 开发调优，Lv1 - LINUX DO](https://linux.do/t/topic/858423)

## 更新日志：

_2025.8.12-1：30 - 添加更多渠道，修正一些渠道名称，完善错误类型和日志，增强了对敏感 HTTP 头的过滤，完整支持跨域请求，修复当后缀错误还是显示检测正确的 bug, 以及少量 bug 和添加不知名的 bug_

_2025.8.11 - 凌晨四点发布_

---
## 保活：

**请求：**
类型: HTTP (S)
URL: https://[您的项目].supabase.co/rest/v1/
请求方法: GET

**请求头配置：**
```
{
"apikey": "Supabase-Anon-Key"
}
```

**Supabase-Anon-Key 获取方法：**
<img width="268" height="698" alt="Image" src="https://github.com/user-attachments/assets/a05452eb-ec9f-4b28-b398-639453d95c94" />

<img width="1039" height="549" alt="Image" src="https://github.com/user-attachments/assets/7c9cc271-17fd-4548-a95b-adfa101a89f8" />

---
## [地址](https://chdwzmwaeodkunrexnko.supabase.co/functions/v1/test)：
我保活了一个，不出意外会一直保活下去，每个月55w请求，单次请求最长400s
```
https://chdwzmwaeodkunrexnko.supabase.co/functions/v1/test
```