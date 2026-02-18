------------------------------------------------------------------------

# 前情提要

> 优选方法的话其实 L 站有，而且很简单，但是**都很不清晰**，导致我第一次弄花了 2 个小时，有人问我就讲一下

# 前言
- 因为优选不知是否明确允许，并且**可能有封号风险，请自行斟酌**
- 我是小白，不懂原理，**只讲方法**，作为记录，可能具有时效性（2025年9月后请自觉判断是否可用）
- 这里不讲 Cloudflare 的 SaaS 回源来优选，因为我觉得复杂化并且我没有成功使用过这个方法

# 需要

（这里不教怎么获取和托管，默认已经获得和会，站里的教程很详细，找不到可以问我）

- Cloudflare 账号
- 一个域名（需要托管在 Cloudflare，支持免费的三级域名）
- EdgeOne 账号（这里使用国际站演示，不确保国内网站是否可以使用，可以通过谷歌来进行登录，可以不用实名和绑卡能够直接使用的）
- EdgeOne 的一个免费套餐（获取方法有很多，目前一个账号可以获得 5 个，绰绰有余）

# 教程

**从头开始，优选关键在第 6 步**

1. 打开控制台 [[服务总览 - EdgeOne - 控制台](https://console.tencentcloud.com/edgeone/zones)] ，注意，这是国际站，和腾讯云有区别
2. 添加站点

<img width="1794" height="655" alt="Image" src="https://github.com/user-attachments/assets/6a190731-63cb-4a43-a488-668d19c0a612" />

 3. EdgeOne 是支持免费的三级域名的，不知道站里有没有人讲过（下面分两种情况）  
  - 先讲购买的二级域名，例如我的 `243344.xyz`

<img width="925" height="491" alt="Image" src="https://github.com/user-attachments/assets/4f51f574-9397-4dfd-9eb8-bcb57828c65b" />

<img width="1341" height="733" alt="Image" src="https://github.com/user-attachments/assets/a3bcb07a-77f9-407c-b36d-0c890c641a32" />

- 这个教程基本上是面向不备案的，备案的也不太需要优选？**必须选择 CNAME！**

<img width="886" height="506" alt="Image" src="https://github.com/user-attachments/assets/c9643830-1723-4dc0-b10c-5506f0f77285" />

- 下面这里分成两种情况，一是使用的二级域名，就照着教程下去验证就行，二如果使用的是三级域名例如 `xxx.dpdns.org`，那就跳过验证，因为我已经添加过了，图片仅供演示

<img width="934" height="645" alt="Image" src="https://github.com/user-attachments/assets/aaf79f19-9f3e-47a1-9ccb-22e633d75f53" />

<img width="1206" height="333" alt="Image" src="https://github.com/user-attachments/assets/73e2746f-321f-4adc-8ffc-4bd0e0098975" />

- 不知为何，同为三级域名，`xxx.ggff.net` 可以过验证，`xxx.dpdns.org` 不行，其实，实践发现，这一步验不验证好像没什么用（还是有用的，有些情况要求验证）

<img width="919" height="529" alt="Image" src="https://github.com/user-attachments/assets/6294bfbd-0837-4e3a-b4ca-94e1f0b38931" />

<img width="814" height="469" alt="Image" src="https://github.com/user-attachments/assets/ce9ec4a5-9855-449b-a663-738527cca9b2" />

- 使用 `xxx.dpdns.org` 的话，接入 `dpdns.org` 即可，然后到验证那一步直接跳过

<img width="759" height="485" alt="Image" src="https://github.com/user-attachments/assets/aa5dd7ec-4fc4-45e4-a123-4766318ab639" />

<img width="940" height="608" alt="Image" src="https://github.com/user-attachments/assets/318ab217-4db7-4321-9c2c-eb94ca4d1a37" />

<img width="681" height="221" alt="Image" src="https://github.com/user-attachments/assets/d36b3bd3-2cb0-4e0d-88c0-88c93cc1f2fa" />

 4. **添加域名**，模板可以不选择，看自己需求，直接下一步

<img width="1731" height="776" alt="Image" src="https://github.com/user-attachments/assets/435d05d6-070c-43e6-86a7-b5c84148c604" />

- 我这里讲一下我的用法（怎么添加源站）

> 我是参考 [[[超详细教学] 教你从零开始，入坑域名、云服务器并部署 New API + Open WebUI！ - 文档共建 - LINUX DO](https://linux.do/t/topic/188813)](https://linux.do/t/topic/188813) 这一篇帖子服务器套 Cloudflare 的 CDN 了，所以不想动了并且 EdgeOne 是xxx不敢这么放心
> <img width="764" height="106" alt="Image" src="https://github.com/user-attachments/assets/5b347203-1ea1-420c-bc7b-a1a4473d05fd" />
> 网站是使用 OpenResty 反代，原本是 `api.wslz.dpdns.org` 就改成别名 `api2.wslz.dpdns.org`, 然后加速域名就可以使用 `api.wslz.dpdns.org` 了（其实我是想配置两个 api 地址，但是发现 new-api 好像不支持并且如果换了 L 站登录也会失效），不想这么麻烦可以直接使用 ip，参考 [[leaflow 部署 donehub 并使用自定义域名接入 edgeone cdn 加速 - 开发调优 - LINUX DO](https://linux.do/t/topic/876681)]，不想讲太多感觉越讲越乱，看自己悟吧，实在不懂也可以给出具体的来具体分析，但是不要把公网 ip 发出来
> _这里简单说明白了一件事情就是中间再套一层 cf 是完全可行的，双层 cdn，不过好像没什么用，说不定减速_

<img width="1350" height="641" alt="Image" src="https://github.com/user-attachments/assets/b63dab76-1e4f-4a9f-a0bd-43d36b4d5106" />

 5. 验证

<img width="1409" height="670" alt="Image" src="https://github.com/user-attachments/assets/80cbf62d-6db3-4ceb-999f-9b2f6e0950dc" />

<img width="1334" height="540" alt="Image" src="https://github.com/user-attachments/assets/93a2c1d8-e8bb-4d51-a808-1fb2ab8457d0" />

- 这时候如果使用 `xxx.dpdns.org` 的话，大概率要你验证域名所有权，直接验证即可，参考图片即可，不要验证 `dpdns.org` 的所有权，是验证 `xxx.dpdns.org` 的所有权，方法同上

<img width="1075" height="699" alt="Image" src="https://github.com/user-attachments/assets/5bed6093-4034-430a-86ed-127e40a1016b" />

<img width="1341" height="678" alt="Image" src="https://github.com/user-attachments/assets/112e1bcd-3343-47fb-b8bb-e94b0cdc0607" />

- 出现不安全考虑开启 HTTPS 配置，EdgeOne 的部署普遍偏慢，要 5 分钟左右，当然可能更多

<img width="1481" height="431" alt="Image" src="https://github.com/user-attachments/assets/48d84b65-2fd0-4b0f-8048-a5a516f605de" />

<img width="1014" height="734" alt="Image" src="https://github.com/user-attachments/assets/2853bd46-6976-4725-a09c-f0627ba92f5d" />

 6. **加速优选（关键，但是是最简单的一步，我也不知道为什么前面会写这么多，越写越多，想到什么就写什么）**  
 找到上一步添加在 Cloudflare 添加的 CNAME 记录，修改值为优选域名，保存即可，优选域名参考 
https://www.wetest.vip

<img width="1351" height="450" alt="Image" src="https://github.com/user-attachments/assets/7733bb5c-e123-48de-98b9-f73a424fb0b3" />

- 出现下面这个不用管

<img width="543" height="263" alt="Image" src="https://github.com/user-attachments/assets/a0fa8dff-551d-422f-9c1b-fd0530788a3a" />

7. **加速前后对比**（参考，因为这个项目 worker 我优选过一次域名了，所以区别不是特别明显，建议自己尝试，通过这个我主要是解决了我服务器的域名优选，加速并且解决移动阻断的问题，而 cf worker 请看下面另外一种方法）

- **加速前**

<img width="764" height="690" alt="Image" src="https://github.com/user-attachments/assets/8ea356cb-f689-4e3e-b140-12b4adc8d7f7" />

<img width="759" height="680" alt="Image" src="https://github.com/user-attachments/assets/51e31451-4f4e-4796-bf3c-548848f74c0d" />

- **加速后**（好像不是特别明显，建议自己试试，因为这个项目 worker 我优选过一次域名了）

<img width="768" height="614" alt="Image" src="https://github.com/user-attachments/assets/07c11b18-9c16-46d0-9888-aa062dcaf615" />

<img width="761" height="631" alt="Image" src="https://github.com/user-attachments/assets/4c64e3db-414e-4350-b7bb-5fb6571ab276" />

8. Cloudflare Worker 域名优选（这个方法仅对 Worker 有效，page 无效，不过 page 好像没被墙？）
> qaz741wsd856:
> page 也能优选，只要不把 page 的域名托管到 cf 就行，方法跟 edgeone 的 cname 接入一样。

- 先部署一个 Cloudflare Worker 项目
- 可以在自定义域名里添加域名路由，这里推荐到域名设置里进行
- 先在 dns 添加记录，优选域名这里看 [[微测网 - 全球云服务监测平台](https://www.wetest.vip/)]

<img width="1354" height="518" alt="Image" src="https://github.com/user-attachments/assets/b0be3400-1743-4c50-b1c7-a0ffc8ff177b" />

- **然后**

<img width="1771" height="600" alt="Image" src="https://github.com/user-attachments/assets/53171e0a-dfc3-4747-ae0e-cd8951038505" />

- **记得末尾要加**`/*`

# 最后

感谢感谢感谢所有对我有帮助的帖子和大佬  

我也不知道为什么这个帖子写了 2 个半小时，越写越详细，我部署也才研究 1 个钟

# 配置补充
- [[解决 EdgeOne 554 回源超时问题 - 开发调优 - LINUX DO](https://linux.do/t/topic/923049)]
- [[腾讯云 EdgeOne 免费加速服务使用经验：从缓存到 Web 防护的配置总结 - 开发调优 - LINUX DO](https://linux.do/t/topic/794089)]
- [[分享一个 edgeone 加速 openwebui 的缓存规则 - 开发调优 - LINUX DO](https://linux.do/t/topic/822160)]
- [[[最终完成] 配置 edgeone 边缘函数及 nginx 反代增加 etag 机制，加快 openwebui 的访问速度 - 开发调优 - LINUX DO](https://linux.do/t/topic/892763)]