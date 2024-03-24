# TuLink - 一个生成重定向链接的api，可用于短域名

起因就是qqbot只能发送备案域名，有人向我借二级域名，不放心就写了一个短域名程序便于监控，同时温习长久未动的flask

<!--more-->

下面是程序：

```python
from flask import Flask, request, redirect, url_for
import json
import random
import string
import time
from collections import OrderedDict
import logging
import re

app = Flask(__name__)
logging.basicConfig(filename='app.log', level=logging.ERROR)

# 获取当前时间戳（秒）
def nowtimestamp():
    timestamp = time.time()
    print(timestamp)

    # 获取当前时间戳（毫秒，需要稍作转换）
    millisecond_timestamp = int(round(time.time() * 1000))
    return millisecond_timestamp

def generate_token(length=10):
    letters_and_digits = string.ascii_lowercase + string.digits
    return ''.join(random.choice(letters_and_digits) for i in range(length))

def is_valid_url(string):  
    regex = re.compile(  
        r'^(?:http|ftp)s?://'  # http:// or https:// or ftp:// or ftps://  
        r'(?:(?:[A-Z0-9](?:[A-Z0-9-]{0,61}[A-Z0-9])?\.)+(?:[A-Z]{2,6}|[A-Z0-9-]{2,})(?!\d)|'  # domain name  
        r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})'  # ...or IP address  
        r'(?::\d+)?'  # optional port  
        r'(?:/?|[/?]\S*)$', re.IGNORECASE)  # optional / or query parameters  
  
    return regex.match(string) is not None 

@app.route('/creat_redirect')
def creat_redirect():
    try:
        if request.headers.getlist("X-Forwarded-For"):
            ip = request.headers.getlist("X-Forwarded-For")[0]
        else:
            ip = request.remote_addr
        users_data_file = "data/users.json"
        redirect_data_file = "data/redirect_datas.json"
        address = request.args.get('address')
        token = request.args.get('token')
        with open(users_data_file, 'r') as file:
            users_data = json.load(file)
        if token in users_data:
            if is_valid_url(address) == True:
                user = users_data[token]
                address_token = generate_token(12)
                data = {address_token: [{"address":address, "timestamp":nowtimestamp(), "user_token":token, "ip":ip}]}
                with open(redirect_data_file, 'r') as file:
                    old_redirect_data = json.load(file)
                # 将新数据插入到原数据的开始位置
                # 由于字典是无序的，这里我们使用OrderedDict来保持插入顺序
                combined_data = OrderedDict(data)
                combined_data.update(old_redirect_data)

                # 将修改后的数据写回文件
                with open(redirect_data_file, 'w') as f:
                    json.dump(combined_data, f, indent=4)  # indent参数用于格式化输出，使JSON更易读
                state = {"code":0, "address":f"https://127.0.0.1/redirect?address_token={address_token}", "timestamp":nowtimestamp()}
                return json.dumps(state)
            else:
                state = {"code":1, "error_message":"address error", "timestamp":nowtimestamp()}
                return json.dumps(state)
        else:
            state = {"code":1, "error_message":"token error", "timestamp":nowtimestamp()}
            return json.dumps(state)
    except Exception as e:
        # 捕获异常，并将异常信息打印出来
        error_code = generate_token(10)
        logging.error(f"{error_code},{nowtimestamp()}An error occurred: %s", e)
        state = {"code":1, "error":error_code, "timestamp":nowtimestamp()}
        return json.dumps(state)
 

@app.route('/redirect')
def redirecturl():
    redirect_data_file = "data/redirect_datas.json"
    address_token = request.args.get('address_token')
    with open(redirect_data_file, 'r') as file:
        redirect_data = json.load(file)
    if address_token in redirect_data:
        address = redirect_data[address_token][0]["address"]
        return redirect(str(address))
    else:
        return "该重定向不存在"

 
 
if __name__ == '__main__':
    app.run(debug=True, port=8823)
```

完整程序请移步仓库：

{% link TuLink::https://github.com/ye-tutu/TuLink::https://q1.qlogo.cn/g?b=qq&nk=2093142951&s=640 %}

接下来介绍使用方法
向/creat_redirect发送请求，参数如下

| token                                     | address        |
| ----------------------------------------- | -------------- |
| token1                                    | 要重定向的域名 |

（token可在../data/users.json查看和设置）

返回如下消息

| code | address                                              | timestamp     |
| ---- | ---------------------------------------------------- | ------------- |
| 0    | http://127.0.0.1/redirect?address_token=ttdux3w7yt61 | 1711293984967 |

/redirect的参数address_token随机生成，timestamp是时间戳

访问返回的address就能重定向到指定的网址

自己部署的就暂时不放出来了（怕被打）
