> 一开始天天添加 api 地址，加 key,cherry studio 添加的渠道都 2，30 个了，而且使用起来不方便，有时候对接其它工具换来换去很麻烦，就想着部署一个 new-api 之类的中转聚合

# 手把手来（但是不包括部署 new-api 这种事情，站里有很多很多很多）
## 1. 添加渠道（模型重定向教程），以 T 佬的为例子

---

<img width="294" height="746" alt="Image" src="https://github.com/user-attachments/assets/b61c6d6d-f03c-44a0-85ef-2e27c1a88662" />

---

<img width="1513" height="604" alt="Image" src="https://github.com/user-attachments/assets/5b65c960-8a7e-4b2b-80d8-4a57215db319" />

---

 默认是 OpenAI 格式，不要改，除非你知道你在干什么！
 名字随意，选择你喜欢的，比如像我一样，叫【公益】T 佬
 然后密钥和 API 地址要填对应

 **然后这里有两个选择:**
- 1）像我一样，先在 GPT-load 套一层，然后用 GPT-load 的代理地址和代理密钥，但是完全没有必要，因为我当时还没有部署 New-API 所以才这么使用，因为是聚合公益往往是一个 key 的，所以没必要这么使用，当然这样可能逻辑一套一套的，可能好用？（麻烦）
- 2）直接使用例如 T 佬的 API 地址和自己创建 / 领取的密钥，组织不用管

---

<img width="701" height="553" alt="Image" src="https://github.com/user-attachments/assets/66006742-7f1a-4c43-8d69-8eeee7129e5c" />

---

<img width="713" height="263" alt="Image" src="https://github.com/user-attachments/assets/10cef1ba-87e2-4000-a5db-942148ee4f09" />

---

 **使用默认 OpenAI 格式，API 地址不要带 /v1/chat/completions, 它会自动补全，或者你使用 #号放在最后面来禁止自动补全**
- 正常：
```
https://tbai.xin
```
- 或者（对于非标准v1格式有需要）：
```
https://tbai.xin/v1/chat/completions#
```

 然后点击获取模型列表，获取不了的前面配置肯定有错误

---

<img width="721" height="271" alt="Image" src="https://github.com/user-attachments/assets/462fa81e-376e-4984-8996-2bccff8b6a2a" />

---

### 添加模型

---

<img width="1130" height="529" alt="Image" src="https://github.com/user-attachments/assets/847d22e8-6b71-4747-a3e8-9960136affd5" />

---

### 然后点击复制所有模型

---

<img width="635" height="110" alt="Image" src="https://github.com/user-attachments/assets/5f4ecabe-df65-4807-b1f3-c33802389560" />

---

然后打开你的 AI 软件，使用下面提示词，删除中文中括号本身和里面的全部内容，粘贴刚才复制的内容，我是使用 Gemini-2.5-pro 生成的，没有出错过，添加前缀tbphp/ 可以修改为你想要的内容，例如：`添加前缀Openrouter/ 和去除:free`

```
给这些模型
【删除中文中括号本身和里面的全部内容】
按照下面格式重定向名称，左边为重定向名称，右边为原来名称，统一为添加前缀tbphp/
{
  "OpenAI/gpt-3.5-turbo": "gpt-3.5-turbo",
  "field_1": "",
  "field_2": "",
  "field_3": ""
}
最后再把重定向后的模型名称以,分割输出给我，用代码块包裹
```

**或者naruto jiang佬的提示词**

```
输入模型列表：
【删除中文中括号本身和里面的全部内容】
重定向规则：

按照“厂商/标准模型名”格式重定向名称，左边为重定向后的名称，右边为原来名称。
厂商列表包括但不限于：
Anthropic
OpenAI
Gemini
Moonshot
智谱（包括 GLM/BigModel 前缀的全部归为“智谱”）
通义千问
DeepSeek
腾讯混元
MistralAI
xAI
Llama
penAl
识别模型名的厂商归属，去掉模型名里的日期或多余后缀，只保留主模型名，如 claude-3-5-haiku，不要 20241022。
按照“厂商/模型名”格式输出，如：Anthropic/claude-3.5-haiku、Gemini/Gemini-xxx、智谱/GLM-4.5、MistralAI/MistralAI-xxx 等。
输出格式：

以对象形式输出，每个条目为 "重定向名称": "原来名称"
再将所有重定向后的模型名称（即左边部分）以英文逗号,分割，放在代码块里输出
示例：

JSON
{
  "Anthropic/claude-3.5-haiku": "claude-3-5-haiku-20241022",
  "Anthropic/claude-opus-4.1": "claude-opus-4-1-20250805",
  "Anthropic/claude-sonnet-4": "claude-sonnet-4-20250514",
  "智谱/GLM-4.5": "BigModel/GLM-4.5",
  "智谱/GLM-4.5-air": "BigModel/GLM-4.5-air",
  "penAl/penAl-xxx": "penAl-xxx",
  "Gemini/Gemini-xxx": "Gemini-xxx",
  "Moonshot/Moonshot-xxx": "Moonshot-xxx",
  "智谱/智谱-xxx": "智谱-xxx",
  "通义千问/通义千问-xxx": "通义千问-xxx",
  "DeepSeek/DeepSeek-xxx": "DeepSeek-xxx",
  "腾讯混元/腾讯混元-xxx": "腾讯混元-xxx",
  "MistralAI/MistralAI-xxx": "MistralAI-xxx",
  "xAI/xAI-xxx": "xAI-xxx",
  "Llama/Llama-xxx": "Llama-xxx"
}
然后只输出重定向后的模型名称（左边部分）：

Text
Anthropic/claude-3.5-haiku,Anthropic/claude-opus-4.1,Anthropic/claude-sonnet-4,智谱/GLM-4.5,智谱/GLM-4.5-air,penAl/penAl-xxx,Gemini/Gemini-xxx,Moonshot/Moonshot-xxx,智谱/智谱-xxx,通义千问/通义千问-xxx,DeepSeek/DeepSeek-xxx,腾讯混元/腾讯混元-xxx,MistralAI/MistralAI-xxx,xAI/xAI-xxx,Llama/Llama-xxx
```

### 点击清除所有模型

---

<img width="633" height="90" alt="Image" src="https://github.com/user-attachments/assets/dc5d38ac-67bd-4064-9ff2-af922bfd981b" />

---

**复制粘贴代码块里面的重定向模型到自定义名称里，并且点击填入，测试模型可以不用管，也可以复制一个快速便宜的模型，例如：thphp/gpt-4.1-nano，不过要注意，测试模型要填写重定向后的模型名称**

---

<img width="1228" height="468" alt="Image" src="https://github.com/user-attachments/assets/3b958ecf-2b4c-41c5-a2fa-28ace5889215" />

---

<img width="718" height="119" alt="Image" src="https://github.com/user-attachments/assets/14c89f9d-3fcf-4a47-b101-6125e80b70aa" />

---

**然后到模型重定向，点击手动编辑，复制 ai 生成的 json 文本粘贴进去**

---

<img width="1191" height="696" alt="Image" src="https://github.com/user-attachments/assets/acb52d67-b365-457b-8303-8e7f3c40e5d7" />

---

<img width="680" height="623" alt="Image" src="https://github.com/user-attachments/assets/2e76d3fc-9426-4094-8ca3-cc98709a33e0" />

---

**分组自己用的话随便添加，标签随意，可以像我一样填写：【公益】，这样子开启标签聚合可以更加直观，这个标签只在渠道上自己可以看到，别人看不到的，可以不填写**

---

<img width="1528" height="711" alt="Image" src="https://github.com/user-attachments/assets/883e165f-db0c-414e-89e8-150ba9b9ec3c" />

---

下面可以把自动禁用关闭，避免误判，我是用 GPT-Load 来轮询和负载，所以不用这个
然后提交即可，渠道就添加完成了

## 2. 模型管理，先添加一个提供商，例如：T 佬

---

<img width="1816" height="751" alt="Image" src="https://github.com/user-attachments/assets/8bb74c3e-c0e0-44b5-b9a7-048ea7cfd868" />

---

**然后点击 未配置模型 ，找到刚刚添加的任意一个模型，点击配置**

---

<img width="825" height="725" alt="Image" src="https://github.com/user-attachments/assets/8594503b-b2f3-42ee-8a4d-1e013a5a0d8a" />

---

然后把 / 后面的内容删除了，仅保留重定向模型添加的部分，下面选择`前缀名称匹配`

---

<img width="738" height="388" alt="Image" src="https://github.com/user-attachments/assets/52c6feb6-ed7c-4d23-86e8-bdb704e5c133" />

---

 图形和描述我没配置，喜欢的可以去配置一下
 标签可以添加，**例如：公益**
 然后点击提交即可

 理论上可以精确配置每个模型，但是理论是理论，实际是实际

---

<img width="449" height="53" alt="Image" src="https://github.com/user-attachments/assets/8b403a48-8675-4b2c-86fc-e4c240423dc7" />

---

## 配置倍率
 开自用模式的不用看，有强迫症可以看，让你的数据更加直观，按照图片找到未配置的模型

---

<img width="1714" height="780" alt="Image" src="https://github.com/user-attachments/assets/756add24-fba9-4caa-8037-5fccfb066889" />

---

全部固定价格为按次即可，可以全部设置为 1

---

<img width="1350" height="578" alt="Image" src="https://github.com/user-attachments/assets/edbf5010-1aee-4e95-971c-ad038aaf5e0f" />

---

**使用批量设置更加方便**

---

<img width="1188" height="586" alt="Image" src="https://github.com/user-attachments/assets/6ca9b315-ab49-403f-aa06-615fa7cadd4b" />

---

好了，终于打完了，打一半网站还维护丢失了一段内容重新打，害我又不能 2 点前睡觉

## 成果：

---

<img width="1809" height="744" alt="Image" src="https://github.com/user-attachments/assets/e0a223ba-a18c-41bd-b5c1-b9e966eebb95" />

---

## 好啦，该你来了
