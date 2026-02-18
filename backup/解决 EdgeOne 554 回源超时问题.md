看了一下**站里**和**搜索引擎**，都没有找到相关解决方案，但是有相关案例

> 我看到佬你套了一层 EdgeOne，EdgeOne 发生 554 是因为 EdgeOne 请求目标服务器，默认 16 秒超时就会触发

然后花了**一个小时**，好吧其实故事挺曲折，解决方法是提工单找的 _（主要是在等工单）_

# 前情提要

> 2025-9-1-2:00- 尝试使用 edgeone 优选域名以解决移动无法访问的问题，实用性有待测试

优选方法的话其实 l 站有，而且很简单，但是都很不清晰，导致我第一次弄花了 2 个小时，有人问的话我就讲一下，没人的话我就当人人都会了

# 今天

看到这个贴子  
[[还是觉得千问和豆包生图能力大于香蕉 - 搞七捻三 / 搞七捻三，Lv1 - LINUX DO](https://linux.do/t/topic/922501)](https://linux.do/t/topic/922501)

于是使用贴子提示词去 cherry 生成，使用的 [https://linux.do/t/topic/442610](https://linux.do/t/topic/442610)  
之前可以用的，但是今天不行，我以为失效了，一直返回  
**554 status code (no body)**  
但是试了试官网可以，于是闹出笑话

> 佬，今天试了试，生图是不是又不行了？

# 发现问题

> 突然想起一件事，昨天给 new-api 套了 edgeone 的 cdn 优选，可能是这个问题，我排查一下

**原因是因为生成图片时间太长导致超时了**

# 解决方案

### 依据

[[回源超时时间](https://www.tencentcloud.com/zh/document/product/1145/63618?has_map=2)](https://www.tencentcloud.com/zh/document/product/1145/63618?has_map=2)

### 实际步骤

<img width="1704" height="699" alt="Image" src="https://github.com/user-attachments/assets/f78eaaac-11dc-4f96-8271-ba7d84c900b4" />

<img width="274" height="84" alt="Image" src="https://github.com/user-attachments/assets/0987a72f-daf9-49cf-8a1e-83b57a8baa32" />

<img width="1403" height="548" alt="Image" src="https://github.com/user-attachments/assets/b753c310-2456-413d-9ec5-15e847d37798" />