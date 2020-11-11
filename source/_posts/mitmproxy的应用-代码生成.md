---
title: mitmproxy的应用-代码生成
date: 2019-12-28 13:48:17
tags: mitmproxy, python
---
[TOC]
## 需求
很多手机App都有做任务，得金币兑换现金的功能
手动做任务有很多限制；写代码自动化运行可以消除一些限制
但写代码时的敲键盘有点费时间，这篇文章就是要解决自动生成代码的问题

## 运行环境
背景知识: python、mitmproxy、jinja2
python: 3.8.1
python第三方库: mitmproxy, requests, Jinja2
手机代理地址: mitmproxy的地址

因为有多个文件，这里贴部分代码，完整代码地址：
https://github.com/zhoujie903/LearnPython/tree/master/mitmproxy_addons/gen_code
     
## 使用说明

### 步骤
设置: 
设置1：设置`self.api_dir`: 存放`App目录`的父目录
`代码目录`下的`gen_code_mitm.py`
```
class GenCode(object):
    def __init__(self):
        ctx.log.info('__init__')
        # 设置代码文件生成的目录\文件夹
        self.api_dir = '/Users/zhoujie/Desktop/api/'
```
设置2: 后面说明...

步骤: 
1. 电脑上运行起mitmproxy；手机设置代理地址为mitmproxy的地址
2. 打开手机App，正常操作：点击、滑动触发相应网络请求被触发，这时各个api方法代码已生成`<api-name>.text`
3. 正常结束mitmproxy，这时在done方法里会生成整体代码`code-<app-name>.py`
4. 微修改`code-<app-name>.py`并运行

步骤说明: 
步骤1: 启动mitmproxy
```
mitmdump --set session='huawei' -s "gen_code_mitm.py"
```
**session的值在后面说明...**

### 设置
`gen_code_mitm.py`并不会自动生成全部网络请求的代码，只会生成已配置好的网络请求的代码

配置如下：
```
class Api(object):
    ...
class GenCode(object):
    def __init__(self):
        ctx.log.info('__init__')
        ...
        
        # 趣头条
        # 步骤1: urls里增加用URL path代表的网络请求
        urls = [
            r'x/tree-game/',
            Api(r'/app/re/taskCenter/info/v1/get',params_as_all=True),            ,
        ]
        
        # 步骤2: 创建代表App的App对象
        self.qu_tou_tiao = App(urls, 'qu-tou-tiao')
        
        # 百度 - 好看视频
        urls = [...]      
        self.hao_kan = App(urls, 'hao-kan') 
        
        # 步骤3: 把App添加进来
        self.flowfilters = [
            self.qu_tou_tiao, 
            self.hao_kan,
            ...
        ] 
```
* 1. `urls`里增加用URL path代表的网络请求

```
urls = [
    r'x/tree-game/',
]
```

* 2. 创建代表App的App对象

```
# 'qu-tou-tiao'中给App取的名字
self.qu_tou_tiao = App(urls, 'qu-tou-tiao')
```
* 3. 把App对象加入到self.flowfilters中，代表要生成这个App的代码

```
self.flowfilters = [
    self.qu_tou_tiao, 
]
```
**urls的配置在后面说明...**

### 目录/文件夹
先约定2个概念：`代码目录`、`App目录`
`代码目录`：生成`App目录`的代码、模板等文件的目录
`App目录`：保存自动生成的App代码文件的目录

`代码目录`有2类文件：
*     *.py    - 代码文件 `gen_code_mitm.py`
*     *.j2.py - 模板文件

`App目录`有2类文件：
* *.py - 代码文件：sessions.py、code-xxx.py、session_xxx.py
* *.text - api文件，包含2个版本的代码文件、响应http_resopnse_body，不会被py文件读取

```
依赖关系[import关系]
code-xxx.py
    sessions.py
        session_xxx.py    
```
* session_xxx.py: 保存着像账号ID、cookie、token等用户登录数据 

* 每个App都有自己的目录: 比如这里有 今天头条极速版本['jin-ri-tou-tiao'], 趣头条['qu-tou-tiao'] 两个目录/文件夹

```
➜  api tree
.
├── jin-ri-tou-tiao
│   ├── api_news_feed_v47.text
│   ├── code-jin-ri-tou-tiao.py
│   ├── search_suggest_homepage_suggest.text
│   ├── session_huawei.py
│   ├── session_ios.py
│   ├── session_xiaomi.py
│   └── sessions.py
└── qu-tou-tiao
    ├── app_re_taskcenter_info_v1_get.text
    ├── code-qu-tou-tiao.py
    ├── xxx.json
    ├── session_huawei.py
    ├── session_xiaomi.py
    ├── sessions.py
    └── sign_sign.text
    
2 directories, 40 files
```
* 每个App会有多份session_xxx.py：比如session_huawei.py, session_ios.py, session_xiaomi.py; 表示有华为、iPhone、小米三台手机的运行数据



### session
这里有一个需求与实现方案的问题：

需求: 一份代码[code-xxx.py] - 维护成本低; 多份账号数据[session_xxx.py] - 批量运行, 效率高
问题: 一个网络请求的数据怎么知道写入哪个账号数据文件中[session_xxx.py]
解决: 
* 推断: 从网络请求的headers、parameters、query、body中的值来推断为某个session
* 指定: 启动时指定为某个session

先约定1个概念：`一次mitmproxy运行`
`一次mitmproxy运行`: 启动mitmproxy -> 1或多部手机操作相同App ->退出\停止mitmproxy[ctrl+c或Q]

**方案一: 指定**

在启动时指定`--set session='huawei'`, 则所有数据写入session_huawei.py文件中
```
mitmdump --set session='huawei' -s "gen_code_mitm.py"
```
**问题:** 多部手机[代表不同账号]操作相同App时，多个账号数据都写入到了同一个账号文件中[session_huawei.py]

**最佳实践: `一次mitmproxy运行`只操作一个手机[一个账号]**

**方案二: 推断**
在启动时不指定session, 代码自动推断
```
mitmdump -s "gen_code_mitm.py"
```

举例：
`今日头条极速版`有一个api: `/search/suggest/homepage_suggest/`
在parameters\query有`"device_platform": "iphone"`字段值, 那就写入session_ios.py中。

**问题：**
如果有一个api无法从headers、parameters、query、body等信息中判断出该写入哪个文件？

这时会写入session_default.py文件中，这时一个账号的数据被写到2个文件中: session_default.py、session_xxx.py, 生成的代码会被认为是2个账号，无法"正常运行"

**最佳实践: `一次mitmproxy运行`只操作一个手机[一个账号], 且启动时设置`guess_as_session`**
```
mitmdump --set guess_as_session='huawei' -s "gen_code_mitm.py"
```
guess_as_session不能像session那样设定任意值
guess_as_session的取值：ios, huawei, xiaomi [自己手头只有这些设备]
若要guess_as_session可以取其它值，需要设置：

`gen_code_mitm.py` - `class GenCode(object)` - `__init__`方法：
```
self.guess_session = collections.OrderedDict(
    ios=re.compile(r'iphone|ios', flags=re.IGNORECASE),
    xiaomi=re.compile(r'xiaomi|mi\+5|miui', flags=re.IGNORECASE),
    huawei=re.compile(r'huawei', flags=re.IGNORECASE),
    # 在这里增加其它的自己想要的值
    # meizhu=re.compile(r'meizhu', flags=re.IGNORECASE),
)
```



### urls的配置 - Api

```
urls = [
    r'x/tree-game/',
    Api(r'/app/re/taskCenter/info/v1/get',params_as_all=True),            ,
]
```
urls中可以添加2类对象: str, Api

**Api类说明**

```
class Api(object):
    def __init__(self, url, f_name='', log='', params_as_all=False, p_as_all_limit=50, body_as_all=False, f_p_enc: set=None, f_b_enc: set=None, f_p_arg: set=None, f_p_kwarg: dict=None, f_b_arg: set=None, f_b_kwarg: dict=None, content_type=''):
```
参数说明
* url - 不需要完整的URL

```
urls = [
    # 方式01：包含host
    'game-center-new.1sapp.com/x/open/game',
    
    # 方式02：只有path
    '/x/open/game',
]
```
* url - 通配: 一个URL生成多个api方法代码

```
urls = [
    # 只要写'score_task/v1/sleep/'路径前缀, 所有以它为前缀的方法都会生成相应代码
    # 'score_task/v1/sleep/status/',
    # 'score_task/v1/sleep/start/',
    # 'score_task/v1/sleep/stop/',
    # 'score_task/v1/sleep/done_task/',
    'score_task/v1/sleep/',
]
```
* url - 通配与匹配顺序

```
urls = [
    # 当触发'score_task/v1/sleep/done_task/'时，
    # 因为'score_task/v1/sleep/'先添加并且也匹配
    # 不会再寻找最佳匹配'score_task/v1/sleep/done_task/'
    # 所以会生成默认的方法名：score_task_v1_sleep_done_task
    # 而不是：sleep_done_task
    'score_task/v1/sleep/',
    Api('score_task/v1/sleep/done_task/', f_name='sleep_done_task')   
]

最佳实践：先具体[长], 后模糊[前缀]

urls = [
    Api('score_task/v1/sleep/done_task/', f_name='sleep_done_task')
    'score_task/v1/sleep/',       
]
```

* f_name - 指定方法名

```
默认方法名：
    parse_result = urlparse(request.url)
    url_path = parse_result.path
    function_name = re.sub(r'[./-]', '_', url_path).strip('_').lower()
    
比如：
    urls = ['/score_task/v1/walk/count/']
    触发的url为：https://i-hl.snssdk.com/score_task/v1/walk/count/
默认：
    def score_task_v1_walk_count(self)
    
指定后：    
    urls = [Api('/score_task/v1/walk/count/',f_name='walk_count')]
    def walk_count(self):
```
* log - 指定打印的日志信息

```
默认：
    def score_task_v1_walk_count(self):
        logging.info('score_task_v1_walk_count')
指定后：
    urls = [Api('/score_task/v1/walk/count/',log='睡觉 - 领金币')]
    def score_task_v1_walk_count(self):
        logging.info('睡觉 - 领金币')     
```
* params_as_all\body_as_all - 应对sign签名的问题

```
比如：
    趣头条的'任务'界面，会调用
    http://api.1sapp.com/app/re/taskCenter/info/v1/get
    来获取任务列表\进度信息
    params有一个字段："sign": "1ad43dd2b7dbacc62b0e2b98e325483a"
    这个字段和其它字段是一个整体，且我们不能伪造sign的值
    直接把整个params值拿来用
默认：
    def app_re_taskcenter_info_v1_get(self):
        url = self.urls['app/re/taskcenter/info/v1/get']
        params = self._params_from(url)
指定后：
    urls = [Api(r'/app/re/taskCenter/info/v1/get',params_as_all=True)]
    def app_re_taskcenter_info_v1_get(self, params_as_all):
        url = self.urls['app/re/taskcenter/info/v1/get']
        params = self._params_from(url)
        params = params_as_all

    并会生成一个全局方法
    def app_re_taskcenter_info_v1_get(user: User):
        for item in user.params_as_all['/app/re/taskCenter/info/v1/get']:
            user.app_re_taskcenter_info_v1_get(item)
```
* p_as_all_limit - 没有实现
* f_p_arg\f_b_arg - 输入参数

```
默认：
    def actcenter_piggy_videoconfirm(self):
        params = self._params_from(url)     
指定后：
    urls = [Api(r'/actcenter/piggy/videoConfirm', f_p_arg={'tag','count'})]
    def actcenter_piggy_videoconfirm(self, tag, count):
        params = self._params_from(url)
        params['tag'] = tag
        params['count'] = count
``` 
* f_p_enc\f_b_enc - params_as_all 与 f_p_arg 的结合体

```    
类中方法 与f_p_arg\f_b_arg相同
并会生成一个像params_as_all\body_as_all相似的全局方法
    def v5_user_rewar_video_callback_json(self, p):
        url = self.urls['/v5/user/rewar_video_callback.json']
        data = self._bodys_from(url)
        data['p'] = p
        
def v5_user_rewar_video_callback_json(user: User):
    for item in user.bodys_encry['/v5/user/rewar_video_callback.json']['p']:
        user.v5_user_rewar_video_callback_json(item)    
```
* f_p_kwarg\f_b_kwarg - 有默认值的输入参数


### 未实现需求
1. 类似动态变化可伪造的值，比如时间戳，没有实现自动生成
```
def api_contains_timstamp(self):
    #这句需要自己输入
    params['ts'] = time.time() 
```

1. 同品牌的2台手机，无法区分出session
