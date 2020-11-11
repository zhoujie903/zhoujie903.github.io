---
title: mitmproxy的使用
date: 2019-09-28 09:48:46
tags:
- mitmproxy
- charles
---
[TOC]
# mitmproxy的使用
**mitmproxy** is a free and open source interactive HTTPS proxy.
官网：https://mitmproxy.org/
安装：`pip3 install mitmproxy` 或 `brew install mitmproxy`
安装后有3个命令行工具：mitmproxy, mitmdump, mitmweb

这里不介绍`mitmweb`的使用，`mitmproxy`与`mitmdump`的功能重点：
`mitmproxy`：交互式；查看流量数据(请求与响应)；执行自定义脚本
`mitmdump`：执行自定义脚本，脚本在Mitmproxy中叫做`Addon`

使用`mitmproxy`：因为是命令行界面，所以需要记住一些快捷键
使用`mitmdump`：偏向编写python代码

通过一个典型的调用，来认识下`Mitmproxy`下的核心概念：
```
➜  ~ mitmproxy --set scripts=ad_short_mitm.py '~u baidu\.com'
```

|  | 在Mitmproxy的叫法 |
| --- | --- |
| set | `Command` |
| scripts | `Options` |
| ad_short_mitm.py | `Addon` |
| '~u baidu\\.com' | `Filter expressions` |




## mitmproxy
```
➜  ~ mitmproxy
```
输入上面命令，启动mitmproxy并显示`Flows界面`：

| Flows界面 |
| --- |
| ![Flows界面](https://upload-images.jianshu.io/upload_images/281540-af5b56b103232a27.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) |


### 快捷键
* 第1个也是最重要的快捷键：`?`: 进入`Help界面`：

| 进入Help界面 |
| --- |
| ![Help界面](http://ww4.sinaimg.cn/large/006tNc79ly1g5a4bm8fymj30xg0oqdjl.jpg) |


* 第2个重要的快捷键：`:`: Command prompt,进入命令输入模式

| 进入命令输入模式 |
| --- |
| ![命令输入模式](http://ww3.sinaimg.cn/large/006tNc79ly1g5aqk3tgiej31r40kuwhs.jpg) |
| 可以输入的命令：可以在`Command Reference界面`查看 |
| 可以按`tab`来命令补全：比如输入flow.m；再按tab; 补全为flow.mark |
| 可以按`tab`来路径补全 |
| 按`enter`执行命令 |
| 常用的命令可以用快捷键，不用进入命令输入模式，省去输入的时间 |

* **界面间跳转快捷键**

| 快捷键 | 界面 | 截图 |
| --- | --- | --- |
| `?` | Help界面 | ![Help界面](http://ww4.sinaimg.cn/large/006tNc79ly1g5a4bm8fymj30xg0oqdjl.jpg) | 
| `K` | Key Bindings界面 | ![Key Bindings界面](http://ww2.sinaimg.cn/large/006tNc79ly1g5a4hkbne2j30xg0oqjv3.jpg) |
| `P` | Flow Details界面 | ![Flow Details界面](http://ww1.sinaimg.cn/large/006tNc79ly1g5a4l0oi7lj30xg0oqwi2.jpg) |
| `E` | Events界面 | ![Events界面](http://ww2.sinaimg.cn/large/006tNc79ly1g5a4mkekpvj30xg0oqaec.jpg) |
| `C` | Command Reference界面 | ![Command Reference界面](http://ww1.sinaimg.cn/large/006tNc79ly1g5a4nxo0cgj30xg0oqjv6.jpg) |
| `O` | Options界面 | ![Options界面](http://ww2.sinaimg.cn/large/006tNc79ly1g5a4pl36xyj30xg0oqadr.jpg) |
**注意：上面的快捷键，都是大写字母，`mitmproxy`的快捷键是区分大小写的**
**Flows界面居然没有快捷键？**


* **导航快捷键**

| 快捷键 | command | 说明 |
| --- | --- | --- |
| q | console.view.pop | 返回：界面间的返回 |
| g | console.nav.start | 跳到第一行 |
| G | console.nav.end | 跳到最后一行 |
| h | console.nav.left |  |
| j | console.nav.down | 跳到下一行 |
| k | console.nav.up | 跳到上一行 |
| l | console.nav.right |  |
| space | console.nav.pagedown |  |
| ctrl b | console.nav.pageup |  |
| ctrl f | console.nav.pagedown |  |
| tab | console.nav.next |  |
**g\G\j\k等这样的导航键是通用的：在Flows、Events、Command、Options等界面都能用**

刚开始学命令行界面时，有这么命令、快捷键要记，没记住怎么办？
这里介绍下mitmproxy的`--no-server, -n`应用

```
➜  ~ mitmproxy --help
usage: mitmproxy [options]
...
Proxy Options:
  --no-server, -n
  --server              Start a proxy server. Enabled by default.
```
* 第1个Terminal窗口里正常启动mitmproxy：`➜  ~ mitmproxy`
* 开启第2个Terminal窗口带`--no-server`选项启动mitmproxy：`➜  ~ mitmproxy --no-server`；按`K`/`C`/`O`/`?`查看快捷键、Command、Options、帮助

| 第2个mitmproxy专门用于查看快捷键、Command、Options、帮助 |
| --- |
| ![](http://ww4.sinaimg.cn/large/006tNc79ly1g5d9drzuxfj31ic0ku0uz.jpg) |



### 案例实战

以`东方头条App - 幸运大转盘`这个游戏为实战

| 东方头条App - 幸运大转盘 |
| --- | 
| ![](http://ww1.sinaimg.cn/large/006tNc79gy1g5d0cz6rndj30n00kgabm.jpg) |
| 1.点击'领取金币'：会发出https://.../zhuanpan/get_zhuanpan_new网络请求 |
| 2.点击'立即领取'：会发出https://.../zhuanpan/get_gold网络请求 |

应用目的：通过mitmproxy的replay功能来减少手动操作时间
知识点：`Filter expressions`, `Options`, `Command`

* 1.启动mitmproxy

```
➜  ~ mitmproxy
```
* 2.点开`东方头条App`到`幸运大转盘`界面
* 3.点击'领取金币'；点击'立即领取'；
* 问题：这时候mitmrpoxy的`Flow界面`已包含上面的网络请求，网络请求非常多，怎么找到需要的请求
* 解答：应用mitmrpoxy的`Filter expressions`
* 4.按`f`快捷键：设置`view_filter`这个`Option`

| 按`f`快捷键, 设置`view_filter` |
| --- |
| ![](http://ww1.sinaimg.cn/large/006tNc79ly1g5d1kj7qvvj316z0u0adk.jpg) |
* 5.输入`~u zhuanpan`, 按回车执行命令

| 输入`~u zhuanpan` |
| --- |
| ![](http://ww3.sinaimg.cn/large/006tNc79ly1g5d1zhmp90j316z0u0gr2.jpg)
![](http://ww4.sinaimg.cn/large/006tNc79ly1g5d1un59ssj316z0u0dl4.jpg) |

* 知识点：`~u zhuanpan`是`Filter expressions`：`~u regex`，用来过滤URL符合regex正则表达式的网络请求；可以按`？`跳转到`Help界面`查看全部的`Filter expressions`

* 6.用`j`导航快捷键定位到`zhuanpan/get_zhuanpan_new`网络请求；按下`m`快捷键将这条网络请求标记

| 按下`m`标记网络请求 |
| --- |
| ![](http://ww1.sinaimg.cn/large/006tNc79gy1g5d2sgnjiqj316z0u0n2j.jpg) |

* 7.用同样的操作，标记`zhuanpan/get_gold`网络请求

| 按下`m`标记网络请求 |
| --- |
| ![](http://ww1.sinaimg.cn/large/006tNc79gy1g5d2va55pyj316z0u079q.jpg) |

* 8.按`:`快捷键, 进入命令输入模式；输入`rep`, 按`tab`补全命令; 输入`@marked`; 按回车执行命令

| 按`tab`补全命令 |
| --- |
| ![](http://ww4.sinaimg.cn/large/006tNc79gy1g5d3gwr0noj316z0u07a6.jpg) ![](http://ww3.sinaimg.cn/large/006tNc79gy1g5d3q3ou9pj316z0u0q5x.jpg)|

出于演示使用mitmrpoxy的目的，才增加了许多不必要的步骤；简洁方法：
* 去除步骤4、5、6、7
* 步骤8改为`: replay.client "(~u zhuanpan/get_zhuanpan_new) | (~u zhuanpan/get_gold)"`

案例到此结束，小结下用到的快捷键、命令：

| 快捷键 | command | 说明 |
| --- | --- | --- |
| `f` | : set view_fliter= | 只显示符合条件的网络请求 |
| `m` | flow.mark.toggle @focus | Toggle mark on this flow |
|  | : replay.client @marked | 重放多条标记的网络请求 |


相关快捷键：

| 快捷键 | 界面 | command | 说明 |
| --- | --- | --- | --- |
| M | flowlist | view.marked.toggle | Toggle viewing marked flows |
| U | flowlist | flow.mark @all false | Un-set all marks |
| r | flowlist | replay.client @focus | Replay this flow |


一些用到`Filter expressions`的`Options`:
view_filter、save_stream_filter、intercept

相关文档：
https://docs.mitmproxy.org/stable/concepts-options/
https://docs.mitmproxy.org/stable/concepts-filters/

### 问题

上面的实战有以下几个问题：
* 第1次收集操作时，不是每次都抽到金币，也有可能抽到广告；怎样每次都跳过广告？
* 游戏有20次机会，要手动输入多次`replay.client @marked`才能把20次机会用完；怎样才能减少手动操作？

这些问题我们通过编写脚本来解决。这里使用`mitmproxy`的其它功能为编写脚本提供方便
把实战的已被标记的2个网络请求保存为文件，方便查看：

| 快捷键 | 界面 | command | 说明 |
| --- | --- | --- | --- |
| w | flowlist | console.command save.file @shown | Save listed flows to file |

* 1.按`w`快捷键, 把`@shown`修改为`@marked`; 指定保存路径；按回车执行命令

| 按`w`保存为文件 |
| --- |
| ![](http://ww1.sinaimg.cn/large/006tNc79ly1g5ggp27f86j31c80u0dll.jpg) ![](http://ww4.sinaimg.cn/large/006tNc79ly1g5ggphx21nj31c80u0ag4.jpg)|
| 输入路径时，可以按`tab`来补全路径 |
| 最好不要使用`~`：像我自己Mac上输入`~/zhuanpan.mitm`，没有保存成功；当然你也可以测试下使用`~`的路径能否保存成功 |
| 输入的文件的后缀名是可以随意指定的；保存的文件为二进制格式 |

* 2.开启第2个Terminal窗口带--no-server选项启动mitmproxy

```
➜  ~ mitmproxy --no-server
```

| 快捷键 | 界面 | command | 说明 |
| --- | --- | --- | --- |
| L | flowlist | console.command view.load | Load flows from file |

* 3.按`L`快捷键, 把步骤1保存的文件加载进来

| 按`L`加载文件 |
| --- |
| ![](http://ww3.sinaimg.cn/large/006tNc79ly1g5ghc549tnj31c80u0q7b.jpg) ![](http://ww4.sinaimg.cn/large/006tNc79ly1g5ghcff17bj31c80u0q7e.jpg)|

好了，编写脚本的准备工作结束！
小结下用到的快捷键、命令：

| 快捷键 | 界面 | command | 说明 |
| --- | --- | --- | --- |
| w | flowlist | console.command save.file @shown | Save listed flows to file |
| L | flowlist | console.command view.load | Load flows from file |


相关快捷键：

| 快捷键 | 界面 | command | 说明 |
| --- | --- | --- | --- |
| e | flowlist | console.command export.file {choice} @focus | Export this flow to file |

快捷键`w`与`e`的区别

| w | e |
| --- | --- |
| 文件为二进制文件 | 文件为文本文件 |
| 保存的信息完整 | 只保存请求信息，不保存响应信息 |
| 能一次保存多条网络请求信息 | 一次只能保存一条网络请求信息 |








## mitmdump
`Mitmproxy`是用python实现的，编写相应的`Addon`脚本也是用python

### shell脚本
先用在`mitmproxy`的`e`快捷键来辅助编写shell脚本，来解决下上面的实战问题
* 1. 用`e`快捷键分别保存`zhuanpan/get_zhuanpan_new`、`zhuanpan/get_gold`网络请求为文件`get_zhuanpan_new.sh`、`get_gold.sh`

get_zhuanpan_new.sh文件内容：[get_gold.sh内容类似，不再列出]
```
curl -H 'Host:zhuanpan.dftoutiao.com' -H 'Content-Type:application/x-www-form-urlencoded' -H 'Connection:keep-alive' -H 'Accept:*/*' -H 'User-Agent:DFTT/2.4.8 (iPhone; iOS 12.3.1; Scale/3.00)' -H 'Accept-Language:zh-Hans-CN;q=1, en-CN;q=0.9, zh-Hant-CN;q=0.8' -H 'Content-Length:484' -H 'Accept-Encoding:br, gzip, deflate' -X POST 'https://zhuanpan.dftoutiao.com/zhuanpan/get_zhuanpan_new' --data-binary 'accid=834536089&appqid=AppStore190602&apptypeid=DFTT&appver=2.4.8&device=iPhone%206s%20Plus%20%28A1634/A1687%29&deviceid=AE9418A1-561A-4F5C-AF05-1EC222A50CF3&fr=rwzx&ime=F2B14555-E2EB-4556-B757-2C55799C92C2&lt=d2RlWExGb015UjRqSkxMZk0rRkYwcTAzd0I3RmErMWRLbzZsYTc4dkFtakxLMmgvdW9xWFhYUEFNdU9XTHZMV3F6cWNhVXRPalBSMkJNUHlvTktRbnc9PQ%3D%3D&network=wifi&num=57&os=iOS%2012.3.1&position=%E6%B5%99%E6%B1%9F&sign=5aac4e159e8d205c084c9f9e6cf4e41f&softname=DFTTIOS&softtype=TouTiao&ts=1564368354'
```

* 2. 把`get_zhuanpan_new.sh`、`get_gold.sh`的内容合并到最终的文件中`zhuanpan.sh`

```shell
#!/usr/bin/env bash

function zhuanpan
{
    # mitmproxy用快捷键e导出的get_zhuanpan_new.sh文件内容原样写到这
    curl -H 'Host:zhuanpan.dftoutiao.com' -H 'Content-Type:application/x-www-form-urlencoded' -H 'Connection:keep-alive' -H 'Accept:*/*' -H 'User-Agent:DFTT/2.4.8 (iPhone; iOS 12.3.1; Scale/3.00)' -H 'Accept-Language:zh-Hans-CN;q=1, en-CN;q=0.9, zh-Hant-CN;q=0.8' -H 'Content-Length:484' -H 'Accept-Encoding:br, gzip, deflate' -X POST 'https://zhuanpan.dftoutiao.com/zhuanpan/get_zhuanpan_new' --data-binary 'accid=834536089&appqid=AppStore190602&apptypeid=DFTT&appver=2.4.8&device=iPhone%206s%20Plus%20%28A1634/A1687%29&deviceid=AE9418A1-561A-4F5C-AF05-1EC222A50CF3&fr=rwzx&ime=F2B14555-E2EB-4556-B757-2C55799C92C2&lt=d2RlWExGb015UjRqSkxMZk0rRkYwcTAzd0I3RmErMWRLbzZsYTc4dkFtakxLMmgvdW9xWFhYUEFNdU9XTHZMV3F6cWNhVXRPalBSMkJNUHlvTktRbnc9PQ%3D%3D&network=wifi&num=57&os=iOS%2012.3.1&position=%E6%B5%99%E6%B1%9F&sign=5aac4e159e8d205c084c9f9e6cf4e41f&softname=DFTTIOS&softtype=TouTiao&ts=1564368354'

    # mitmproxy用快捷键e导出的get_gold.sh文件内容原样写到这
    curl -H 'Host:zhuanpan.dftoutiao.com' -H 'Content-Type:application/x-www-form-urlencoded' -H 'Connection:keep-alive' -H 'Accept:*/*' -H 'User-Agent:DFTT/2.4.8 (iPhone; iOS 12.3.1; Scale/3.00)' -H 'Accept-Language:zh-Hans-CN;q=1, en-CN;q=0.9, zh-Hant-CN;q=0.8' -H 'Content-Length:487' -H 'Accept-Encoding:br, gzip, deflate' -X POST 'https://zhuanpan.dftoutiao.com/zhuanpan/get_gold' --data-binary 'accid=834536089&appqid=AppStore190602&apptypeid=DFTT&appver=2.4.8&device=iPhone%206s%20Plus%20%28A1634/A1687%29&deviceid=AE9418A1-561A-4F5C-AF05-1EC222A50CF3&fr=rwzx&ime=F2B14555-E2EB-4556-B757-2C55799C92C2&isfirst=0&lt=d2RlWExGb015UjRqSkxMZk0rRkYwcTAzd0I3RmErMWRLbzZsYTc4dkFtakxLMmgvdW9xWFhYUEFNdU9XTHZMV3F6cWNhVXRPalBSMkJNUHlvTktRbnc9PQ%3D%3D&network=wifi&os=iOS%2012.3.1&position=%E6%B5%99%E6%B1%9F&sign=c6f61f80d1c001ac5382ef73632e0e9e&softname=DFTTIOS&softtype=TouTiao&ts=1564368376'
}

for ((i=0; i<20; i++));
do
    zhuanpan
done
```

ok，shell脚本以编写完成
### python脚本

关于`Addon`的概念可以查看：
https://docs.mitmproxy.org/stable/addons-overview/

编写`Addon`脚本写些什么呢？先上一下模板：
```
from mitmproxy import ctx
from mitmproxy import flowfilter
from mitmproxy import http
from mitmproxy import addonmanager

class Myaddon(object):

    def __init__(self):
        pass

    def load(self, entry: addonmanager.Loader):
        pass

    def request(self, flow: http.HTTPFlow):
        pass

    def response(self, flow: http.HTTPFlow):
        pass

addons = [
    Myaddon()
]
```
**编写`Addon`脚本:就是选择性的实现上面的方法**

**具体都有哪些方法可以选择性实现，可以查看如下文档：**

|  |  |
| --- | --- |
| 文档 | https://docs.mitmproxy.org/stable/addons-events/ |
| 源代码 | docs/src/examples/addons/events.py |
| 源代码 | mitmproxy/eventsequence.py |


开始实现`Addon`脚本：
* 1. 新建文件`zhuangpan_mitm.py`, 实现`__init__`方法：

```
import json
import re

from mitmproxy import ctx
from mitmproxy import flowfilter
from mitmproxy import http

class Zhuangpan(object):
    def __init__(self):
        self.filter = flowfilter.parse(r'(~u zhuanpan/get_zhuanpan_new) | (~u zhuanpan/get_gold)')
        self.new_fliter = flowfilter.parse(r'~u zhuanpan/get_zhuanpan_new') 
        self.get_fliter = flowfilter.parse(r'~u zhuanpan/get_gold')
        self.flows = []
        self.urls = set()
        self.remain = 0

addons = [
    Zhuangpan()
]
```

这里用到了Mitmproxy的api:`flowfilter.parse`：

```
# mitmproxy/flowfilter.py文件
def parse(s: str) -> TFilter:

# 还定义了：
def match(flt, flow):
    """
        Matches a flow against a compiled filter expression.
        Returns True if matched, False if not.
        ....
    """
```

* 2. 实现`request`方法：

```
class Zhuangpan(object):
    ...
    def request(self, flow: http.HTTPFlow):               
        if flowfilter.match(self.filter, flow):
            url = flow.request.url
            if not url in self.urls: 
                ctx.log.alert(url)
                self.flows.append(flow)
                self.urls.add(url)

```
* 3. 实现`response`方法：


```
class Zhuangpan(object):                       
    ...
    def response(self, flow: http.HTTPFlow):
        if flowfilter.match(self.new_fliter, flow):
            flow.response.replace(r'"gold":0', '"gold":999')

            text = flow.response.text
            data = json.loads(text)
            self.remain = data.get('data').get('cur_num')
            ctx.log.alert('remain count:{}'.format(self.remain))

        if flowfilter.match(self.get_fliter, flow):
            if self.remain > 0 and len(self.urls) >= 2:                
                flows = [f.copy() for f in self.flows]
                ctx.master.commands.call("replay.client", flows)
```

使用`ctx.log.xxx`等方法来代替使用`print`或`logging.warning`等方法：  
* 在mitmproxy中，`ctx.log.xxx`记录的信息会出现在`Event`界面,         而其它方法不会出现在`Event`界面
* 在mitmdump中，`ctx.log.xxx`记录的信息事件显示的顺序正确,         而其它方法显示的顺序不正确

### 使用

```
➜  ~ mitmdump --scripts zhuangpan_mitm.py
Loading script zhuangpan_mitm.py
Proxy server listening at http://*:8080

  # --scripts SCRIPT, -s SCRIPT
                        Execute a script. May be passed multiple times.
```


```
mitmdump --set userid=zhj -s "mitm_user_xxx.py"  -s math_mitm.py '~u mapi.hddgood.com'

mitmdump --set replacements='/~s/"video_url":"(.+)"}/"video_url":"https://vd3.bdstatic.com/abc.mp4"}'

# 代码里可以调用
ctx.master.commands.call("replay.client", [flow])

ctx.master.commands.execute("view.focus.go 0")
```
## 其它
问题：只关注某个域名下的流量，怎么设置？
解决：ignore_hosts
https://docs.mitmproxy.org/stable/howto-ignoredomains/

```
# Ignore everything but example.com and mitmproxy.org:
--ignore-hosts '^(?!example\.com)(?!mitmproxy\.org)'

正则表达式：反前瞻
反前瞻：要匹配某个模式时，需要在它 后面找不到含有给定前瞻模式的内容
foo(?!bar)  Negative lookahead assertion. The pattern foo will only match if not followed by a match of pattern bar.
```
## 代码阅读

源码地址：https://github.com/mitmproxy/mitmproxy

mitmproxy/tools/_main.py
```
入口方法：
def mitmproxy(args=None) -> typing.Optional[int]:
    run(console.master.ConsoleMaster)

def mitmdump(args=None) -> typing.Optional[int]:
    run(dump.DumpMaster)
    
主要代码
def run(master_cls):

    opts   = options.Options()
    master = master_cls(opts)
    
    pconf  = proxy.config.ProxyConfig(opts)
    server = proxy.server.ProxyServer(pconf)
    
    master.server = server
    master.run()
    return master
    
    

master.Master
    console.master.ConsoleMaster
    dump.DumpMaster
    web.master.WebMaster

Server
    proxy.server.ProxyServer
    proxy.server.DummyServer


Master与Server关系：
    master.server = server


Master和Server对象生成：
    Master(opts: options.Options)
    Server(config: config.ProxyConfig)

ProxyConfig与Options关系：
    ProxyConfig(options: options.Options)
```
```
开始运行：
    master.run()
        master.start()

    def start(self):
        if self.server:
            ServerThread(self.server).start()

class ServerThread(basethread.BaseThread):
    def __init__(self, server):
        self.server = server
        address = getattr(self.server, "address", None)
        super().__init__(
            "ServerThread ({})".format(repr(address))
        )

    def run(self):
        self.server.serve_forever()
```
```
线程：
    ServerThread
    connection_thread

    def connection_thread(self, connection, client_address):
        with self.handler_counter:
            try:
                self.handle_client_connection(connection, client_address)
            finally:
                close_socket(connection)


    def handle_client_connection(self, conn, client_address):
        h = ConnectionHandler(
            conn,
            client_address,
            self.config,
            self.channel
        )
        h.handle()

    def handle(self):
        self.log("clientconnect", "info")

        root_layer = None
        root_layer = self._create_root_layer()
        root_layer = self.channel.ask("clientconnect", root_layer)
        root_layer()

        self.log("clientdisconnect", "info")

    def _create_root_layer(self):
        root_ctx = ...
        mode = self.config.options.mode
        if mode.startswith("upstream:"):
            return modes.HttpUpstreamProxy
        elif mode == "transparent":
            return modes.TransparentProxy(root_ctx)
        elif mode == "regular":
            return modes.HttpProxy(root_ctx)
        
```
```
addons的运行过程[生命周期]
[1]. "load"
[2]. "running"
[3]. "configure"

[1]. "load"
DumpMaster.__init__(self,options):
    super().__init__(options)
    self.addons.add(*addons.default_addons())

AddonManager.add(self, *addons):
    for i in addons:
        self.chain.append(self.register(i))
        
AddonManager.register(self, addon):
    l = Loader(self.master)
    self.invoke_addon(addon, "load", l)
    
[2]. "running"
master.run():
    loop = asyncio.get_event_loop()
    self.run_loop(loop.run_forever)
        
    master.run_loop(self, loop):
        asyncio.ensure_future(self.running())
        
    master.running(self):
        self.addons.trigger("running")
        
[3]. "configure"
class AddonManager:
    def __init__(self, master):
        self.lookup = {}
        self.chain = []
        self.master = master
        master.options.changed.connect(self._configure_all)

    def _configure_all(self, options, updated):
        self.trigger("configure", updated)

```


## 参考
[https://stackoverflow.com/questions/51893788/using-mitmproxy-inside-python-script](https://stackoverflow.com/questions/51893788/using-mitmproxy-inside-python-script)

[https://dev.to/kevcui/3-mitmproxy-tips-you-might-not-know-about-5dbg](https://dev.to/kevcui/3-mitmproxy-tips-you-might-not-know-about-5dbg)

[https://github.com/KevCui/mitm-scripts](https://github.com/KevCui/mitm-scripts)


