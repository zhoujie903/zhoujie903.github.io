---
title: mitmproxy与charles的抓包应用
date: 2019-09-29 16:14:57
tags: 
- mitmproxy
- charles
---

[TOC]
## 问题
mitmproxy是命令行界面，不太方便查看，自己遇到的问题有：
* 在Flow Details界面不知道用哪个快捷键查看下一条或上一条网络请求详情
* json中的中文显示为 \ua9b4 或 ？不能看清是哪个中文
* 不知道怎么复制界面中显示的内容

Charles的编程能力弱

把Charles和mitmproxy结合起来使用，可以查看方便、编程扩展

## 环境

电脑和手机都连接到同一个Wi-Fi, 比如TP_LINK_629F
电脑: ip=192.168.1.100; 运行着Charles、mitmproxy
Charles: 192.168.1.100:8888
mitmproxy: 192.168.1.100:8080
手机: 设置代理地址为Charles的地址192.168.1.100:8888

Charles - External Proxy Settings功能开启：
* Web Proxy(HTTP) - Web Proxy Server设置为mitmproxy的地址192.168.1.100:8080
* Secure Web Proxy(HTTPS) - Secure Web Proxy Server设置为mitmproxy的地址192.168.1.100:8080

| 手机 |
| :-: |
| ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6s2bq3vglj30u01hcaak.jpg) |
| |
| Charles - External Proxy Settings|
| |
| ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6s2ks0wauj31am0qe76x.jpg) |

## 补充: mitmproxy设置上游代理

```
mitmdump --mode=upstream:http://127.0.0.1:8888 --ssl-insecure
```

## 应用
应用：自动生成接口的python代码

以`趣头条App`为例：
阅读得积分的接口为: `https://api.1sapp.com/readtimer/report`
自动生成的代码大约如下：

```
def readtimer_report(self):

    headers = {
		'User-Agent': 'qukan_android;retrofit/2.4.0 okhttp/3.11.0;os/7.0 M5 Note Meizu;device/862546036581345;version/3.9.41.000.0904.1121;channel/012',
		'Host': 'api.1sapp.com',
		'Accept-Encoding': 'gzip',
		'Connection': 'keep-alive',
	}

    params = {
		'qdata': 'NzVFNkE3Mjc0RjEzRkYwRTM5OTEzNjAyRUZBMDMzQTAuY0dGeVlXMGZPR0kyTldKalltTXRNVGczT1MwME9EZGlMVGszT0RBdE1EUmlOVEV3TWpOak9UazRIblpsY25OcGIyNGZNVEVlY0d4aGRHWnZjbTBmWVc1a2NtOXBaQjVsWXg4eC46LxvyIvgD62/T93SlANmywpigTwlOwcfCHe0iZ1D8mH1zpslt2JCRPdiHOj1M20bU0zDX0odUOBig6Kt51mheNJuQYeDvp15R8RSGTT3LR9s55nBCGvWyTLq+3pjEvkyERElR9E2I384/nHQR4iqxqv7LKQ4rBA0R6bNG8sksHqNl1izSbF87G/4/Qw5vVYcuNUfU0BM6vvIbsy2CTPWlJ51YCzadQZZLONuaYTpyhuOiUV4vnx6qkvpYDNp9XpPPjbXxJAb7fikqjWSdyx167hXDPzUkNZGndjZsv7kQANDkIk2Dm+g5YW1I49xnkOzJkmxvyrLevnsSb9S5fSEUEyfq0GlPHE0RRBeSjFxVltH1zdZraTtk13Z+MvA7HBYQONz/0OwyMnujc1Ety91uKh6YCCVEDvBO+RTOzoRDa3nlRo3FTo9OeRBsyL20qIP24977MMYXEoxinTuwNonipCjnjSYIrhOu6cyv7uxuLd6FxtmTsydawNGMPI/K+habIKXNUFsQMCUcAGhYpoKQJvkQqHEq6lPyZZzXDot1EsN6bsGj56xQdiuJZLMFyZaGNR6E9FmVlI0LAVT8ttVpOvs+5f08T0iMxMNb0VQk6DOySpYHp7EVjc9YFpPcVxj8aXvuTjoPkaNGhSKQ0fgBd8HVRLslcnzX0QLJkTuU7NQ9aili6m2M2hWvh/q8ghWrvtLT+izCiFNKHE+4GTC9J6jqgyjHsXkAjcOSBAaIXMQKnOd664hdoR2GqV+GAy95fc5zZCJ7EFvzmTbJQrpMOwW+Y2NvYPZtgjw1uJEyU7AR7nVw7VqMjPpCDYeWBWoQ1W4OjlXTqgBR4MIu1sTag6a+my/0hItf91SNa58zCN3YmE2NnsWwwiCC+ZP91moV/KqPwX3vMLKW4/3Vsziqe8gl',
	}

    data = {
	}

    url = 'http://api.1sapp.com/readtimer/report'
    result = self._get(url, headers=headers, params=params, data=data)
    return result
     
```

应用步骤：
1. 手机按上面环境设置好代理地址
2. 在电脑上启动Charles，Charles按上面环境设置
3. 在电脑上启动mitimproxy, 启动命令如下：`mitmproxy -s gen_code_mitm.py`
4. 文件gen_code_mitm.py内容参考下面
5. 在手机`趣头条App`阅读文章或看视频一定时间
6. 会在`~/Desktop/api/`目录下生成一个叫`readtimer_report.text`
7. 把`readtimer_report.text`的相应代码复制粘帖到`文件qu_tou_tiao.py`中
8. 按需要修改自动生成的代码为希望的样子
9. 运行`qu_tou_tiao.py`

注意：
* `readtimer_report.text`的名字和所在目录是由代码`gen_code_mitm.py`决定的，请自行阅读修改
* `readtimer_report`方法名由代码`gen_code_mitm.py`决定的，请自行阅读修改
* 对body为复杂json格式的代码自动生成会有错误，有能力的自行修改
* `readtimer/report`这个接口自动生成的代码数据是不能重复获取积分的，这里只是演示

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6s4idpz9gj31fa0u045o.jpg)

## 文件
文件qu_tou_tiao.py: 
```
'''
代码模板
'''
import requests
import json
import logging

logging.basicConfig(format='%(asctime)s:%(message)s', datefmt='%m-%d %H:%M:%S', level=logging.INFO)

class User(object):
    def __init__(self):
        pass
        
    def api_need_implement(self):
        pass

    def _header(self):
        return {
            'User-Agent': '',
            'Cookie':self.cookie
        }

    @staticmethod
    def _post(url, data=None, json=None, p=logging.warning, **kwargs):
        res = requests.post(url, data=data, **kwargs)
        result = res.text
        p(res.json())
        logging.info('')
        return result

    @staticmethod
    def _get(url, params=None, p=logging.warning, **kwargs):
        res = requests.get(url, params=params, **kwargs)
        result = res.text
        p(json.loads(result))
        logging.info('')
        return result

def genUsers():
    yield User()

if __name__ == "__main__":
    for user in genUsers():
        user.api_need_implement()  
```


文件gen_code_mitm.py
```
import json
import re
from urllib.parse import urlparse

from mitmproxy import ctx
from mitmproxy import flowfilter
from mitmproxy import http

'''
生成接口python代码
'''

class GenCode(object):
    def __init__(self):
        ctx.log.info('__init__')

        # 趣头条
        urls = [
            r'taskcenter/getListV2',#tab页：任务
            r'readtimer/report',
        ]
        self.qu_tou_tiao = flowfilter.parse('|'.join(urls)) 

        # 百度 - 全民小视频 
        urls = [
            r'mvideo/api', # 每日签到
        ]
        self.quan_ming = flowfilter.parse('|'.join(urls)) 

        self.flowfilters = [
            self.qu_tou_tiao, 
            self.quan_ming,
        ]      

    def load(self, loader):
        ctx.log.info('event: load')

    def configure(self, updated):
        ctx.log.info('event: configure')

    def running(self):
        ctx.log.info('event: running')

    def done(self):
        ctx.log.info('event: done')



    def response(self, flow: http.HTTPFlow):
        if any( [ filter(flow) for filter in self.flowfilters ] ):

            request: http.HTTPRequest = flow.request

            parse_result = urlparse(request.url)
            url_path = parse_result.path

            function_name = re.sub(r'[/-]','_', url_path).strip('_')
            headers_code = self.headers_string(flow)
            params_code = self.params_string(flow)
            data_code = self.data_string(flow) 

            path = f'''/Users/zhoujie/Desktop/api/{function_name}.text'''  
            with open(path, 'a') as f:
                print(f'''# ---------------------''',file=f)

                code = f'''
def {function_name}(self):

    {headers_code}

    {params_code}

    {data_code}

    url = '{request.scheme}://{request.pretty_host}{url_path}'
    result = self._{request.method.lower()}(url, headers=headers, params=params, data=data)
    return result
                
'''
                f.write(code)

                print(f'''Response:''',file=f)
                print(f'''{flow.response.text}''',file=f)
                print(f'''# ---------------------\n\n''',file=f)

    def headers_string(self, flow: http.HTTPFlow):
        lines = ''
        for key,value in flow.request.headers.items():
            lines += f"\n\t\t'{key}': '{value}',"
        s = f'''headers = {{{lines}\n\t}}'''        
        return s


    def params_string(self, flow: http.HTTPFlow):
        lines = ''
        for key,value in flow.request.query.items():
            lines += f"\n\t\t'{key}': '{value}',"
        s = f'''params = {{{lines}\n\t}}'''        
        return s

    def data_string(self, flow: http.HTTPFlow):
        '''
        Content-Type: application/x-www-form-urlencoded
        Content-Type: application/json; charset=utf-8
        Content-Type: text/plain;charset=utf-8
        '''
        lines = ''

        # [urlencoded_form, multipart_form, plan, json]取其一
        for key,value in flow.request.urlencoded_form.items():
            lines += f"\n\t\t'{key}': '{value}',"

        for key,value in flow.request.multipart_form.items():
            key = key.decode(encoding='utf-8')
            value = value.decode(encoding='utf-8') 
            lines += f"\n\t\t'{key}': '{value}',"

        # Todo:复杂json数据还不能代码化
        if 'application/json' in flow.request.headers.get('content-type',''):
            d = json.loads(flow.request.text)
            for key,value in d.items():
                lines += f"\n\t\t'{key}': {value},"
        
        s = f'''data = {{{lines}\n\t}}'''        
        return s


    

addons = [
    GenCode()
]

```
