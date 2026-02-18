RT, 今天排查了一下为什么使用我的主域名会出现这种问题，使用一些没配置的域名就不会出现这个问题，我就怀疑是 cloudflare 的配置问题，全网搜索了一下，没找到解决办法，自己排查出来了，给大家提个醒，当然也有可能是我配置错误，不太明白为什么会出现这个原因

项目是这个，一些开源项目部署也会导致这个问题: [Grok2API - 修复视频生成、支持动态头部 - 开发调优 - LINUX DO](https://linux.do/t/topic/1101201)

**没开启前**

---

<img width="1099" height="589" alt="Image" src="https://github.com/user-attachments/assets/e3e36f9d-d071-48d9-b336-5e2a969a2309" />

---

**开启后**

---

<img width="690" height="270" alt="Image" src="https://github.com/user-attachments/assets/554b2193-bb99-464b-b424-c9932592c1bb" />

---

帖子写完了，发现其实有类似的，不过没搜索到: [如果 Cloudflare 的 Rocket Loader 让你的 Fuclaude 网站在 Chrome 上挂了（Safari 却能用） - 开发调优 - LINUX DO](https://linux.do/t/topic/457981)

可以在 Dashboard 里面写规则让其只在特定页面开启

---

<img width="1358" height="720" alt="Image" src="https://github.com/user-attachments/assets/b7f644b2-953b-4814-98b4-382cbf75ac8d" />

---
