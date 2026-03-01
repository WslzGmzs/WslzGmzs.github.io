今天机缘巧合发现 Cloudflare AI Gateway 支持自定义供应商了

<img width="1504" height="680" alt="Image" src="https://github.com/user-attachments/assets/16a562ec-14c2-4cf2-b812-8a44a40a065f" />

就简单测试一下看看 Cloudflare 把什么发给了我们的上游

先搭一个靶子先

```python
from flask import Flask, request
import json
import sys

app = Flask(__name__)

@app.route('/', defaults={'path': ''}, methods=['GET', 'POST', 'PUT', 'DELETE'])
@app.route('/<path:path>', methods=['GET', 'POST', 'PUT', 'DELETE'])
def catch_all(path):
    request_info = {
        "method": request.method,
        "path": path,
        "ip": request.remote_addr,
        "headers": dict(request.headers),
        "body": request.get_data(as_text=True)
    }

    print("\n--- New Request Received ---", flush=True)
    print(json.dumps(request_info, indent=4, ensure_ascii=False), flush=True)
    
    return {"status": "success", "received_path": path}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

```
pip install flask
```

<img width="555" height="431" alt="Image" src="https://github.com/user-attachments/assets/1e924b42-0e8a-4a6d-bcd3-26f79adfad6b" />

Post 请求得到下面

```
{
    "method": "POST",
    "path": "v1/chat/completions",
    "ip": "172.18.0.1",
    "headers": {
        "Host": "xxx.xxxx.xyz",
        "X-Forwarded-For": "13.173.100.50,2a06:98c0:0000::13, 162.18.128.41",
        "X-Forwarded-Host": "xxx.xxxx.xyz",
        "X-Real-Ip": "162.18.128.41",
        "X-Forwarded-Proto": "https",
        "Content-Length": "146",
        "Cf-Ray": "9d56fb73640d888d-SIN",
        "Cf-Iplongitude": "103.85007",
        "Cf-Iplatitude": "1.28967",
        "Accept-Encoding": "gzip, br",
        "Cf-Ipcountry": "SG",
        "Cf-Ipcontinent": "AS",
        "Cf-Visitor": "{\"scheme\":\"https\"}",
        "Cf-Ipcity": "Singapore",
        "Cf-Worker": "gateway.ai.cloudflare.com",
        "Cf-Timezone": "Asia/Singapore",
        "Cf-Pseudo-Ipv4": "253.102.157.193",
        "X-Title": "Cherry Studio",
        "Cf-Postal-Code": "018989",
        "Content-Type": "application/json",
        "Cf-Ew-Via": "15",
        "Cdn-Loop": "cloudflare; loops=1; subreqs=1",
        "Accept-Language": "zh-CN",
        "Accept": "*/*",
        "Authorization": "Bearer sk-xxx",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) CherryStudio/1.7.19 Chrome/140.0.7339.249 Electron/38.7.0 Safari/537.36",
        "Cf-Connecting-Ip": "2a06:98c0:3600::163",
        "Http-Referer": "https://cherry-ai.com",
        "Priority": "u=1, i",
        "Sec-Ch-Ua": "\"Not=A?Brand\";v=\"24\", \"Chromium\";v=\"140\"",
        "Sec-Ch-Ua-Mobile": "?0",
        "Sec-Ch-Ua-Platform": "\"Windows\"",
        "Sec-Fetch-Dest": "empty",
        "Sec-Fetch-Mode": "cors",
        "Sec-Fetch-Site": "cross-site",
        "Cookie": "__cf_bm=BlU.H71jVXbGn.KlFpriO7FZuEHdENh_Mut6.VVgTqA-1772233459-1.0.1.1-qwxSdI3avXTLxkiFUHrRbhmU2oGMNbT69W4c4P5V37r1vVFqaEUJ_oIsnh332SosE4fpxLRf9qeIDNTCG_rD4yfet7GaMBBc1_UJwCtKty0"
    },
    "body": "{\"model\":\"1\",\"messages\":[{\"role\":\"system\",\"content\":\"test\"},{\"role\":\"user\",\"content\":\"hi\"}],\"stream\":true,\"stream_options\":{\"include_usage\":true}}"
}
```

这里可以看到 `"X-Forwarded-For": "13.173.100.50` 精准的泄露了我的真实 ip，而我目前没有什么办法，除非不用 Cloudflare AI Gateway

