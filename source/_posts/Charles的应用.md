---
title: Charles的应用
date: 2019-09-28 18:27:26
tags:
- charles
---

[TOC]
## Rewrite应用

### 案例01
问题：`今日头条极速版`App每天的`阅读推送文章`任务因为每天收到的推送很少，因而不能得很高的积分，怎样把普通文章的阅读变为推送文章的阅读呢？

思考：对比普通文章的阅读与推送文章的阅读发出的网络数据，找出差异

普通文章的阅读与推送文章的阅读达到奖励标准时，都用相同的接口`https://is.snssdk.com/score_task/v1/task/get_read_bonus/`
```发送数据
# 普通阅读文章/视频
https://is.snssdk.com/score_task/v1/task/get_read_bonus/?fp=xxx&...&group_id=6689697061983486472

# 推送文章的阅读
https://is.snssdk.com/score_task/v1/task/get_read_bonus/?fp=xxx&...&&impression_type=push&group_id=6689697061983486472
```
对比上面的接口数据发现：
推送阅读只比普通阅读**多出了impression_type=push**的Query String

解决：用Charles的`Rewrite功能Add Query Param`来增加impression_type=push解决问题

![Rewrite功能-Type:Add Query Param](http://upload-images.jianshu.io/upload_images/281540-64acc9462bb437c4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的get_read_bonus重写规则Export导出的get_read_bonus.xml文件内容如下：
```
<?xml version='1.0' encoding='UTF-8' ?>
<?charles serialisation-version='2.0' ?>
<rewriteSet-array>
  <rewriteSet>
    <active>true</active>
    <name>get_read_bonus</name>
    <hosts>
      <locationPatterns>
        <locationMatch>
          <location>
            <protocol>https</protocol>
            <host>is.snssdk.com</host>
            <path>/score_task/v1/task/get_read_bonus/</path>
          </location>
          <enabled>true</enabled>
        </locationMatch>
      </locationPatterns>
    </hosts>
    <rules>
      <rewriteRule>
        <active>true</active>
        <ruleType>8</ruleType>
        <matchHeader></matchHeader>
        <matchValue></matchValue>
        <matchHeaderRegex>false</matchHeaderRegex>
        <matchValueRegex>false</matchValueRegex>
        <matchRequest>false</matchRequest>
        <matchResponse>false</matchResponse>
        <newHeader>impression_type</newHeader>
        <newValue>push</newValue>
        <newHeaderRegex>false</newHeaderRegex>
        <newValueRegex>false</newValueRegex>
        <matchWholeValue>false</matchWholeValue>
        <caseSensitive>false</caseSensitive>
        <replaceType>2</replaceType>
      </rewriteRule>
    </rules>
  </rewriteSet>
</rewriteSet-array>
```

重写规则设置正确与否验证：
![规则设置正确与否验证](http://upload-images.jianshu.io/upload_images/281540-ea8dc4743ec32001.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在上面的`Notes`列上会显示`Rewrite Tool: query added "impression_type: push"`

**应用：**
1. 打开Charles并生效上面的Rewrite设置
2. 手机设置代理为Charles的代理地址
3. 正常阅读普通文章\视频达到奖励时点
4. 查找`我的收益`页面，验证成功与否

### 案例02
问题：`趣消除`、`趣键盘`、`东方头条`等App都看广告得金币，怎样减少广告的时间？

思考：广告是哪里来的？广告时长是怎么来的？
当点击App上的按钮弹出广告时，3个App都调用了接口：
```
https://is.snssdk.com/api/ad/union/sdk/get_ads/
```
响应如下：
```
{
......
		"video": {
			"cover_height": 1280,
			"cover_url": "http://sf1-ttcdn-tos.pstatp.com/img/mosaic-legacy/1be91000a8c62c6ba6221~noop.jpg",
			"cover_width": 720,
			"endcard": "https://www.toutiaopage.com/union/endcard/1629848424707111/?rit=909946692\u0026req_id=ED6EC127-C359-4C18-A41E-3A5F6F499250u3183\u0026ad_sdk_version=1.9.9.0\u0026os=ios\u0026lang=cn\u0026style_id=1104\u0026ad_id=1629844369912839\u0026_toutiao_params=%7B%22cid%22%3A1629848424707111%2C%22device_id%22%3A9724339963504202%2C%22log_extra%22%3A%22%7B%5C%22ad_price%5C%22%3A%5C%22XOub4AAGRWZc65vgAAZFZgz-hMMMgth42hwxAg%5C%22%2C%5C%22convert_id%5C%22%3A1629408290774020%2C%5C%22orit%5C%22%3A900000000%2C%5C%22req_id%5C%22%3A%5C%22ED6EC127-C359-4C18-A41E-3A5F6F499250u3183%5C%22%2C%5C%22rit%5C%22%3A909946692%7D%22%2C%22orit%22%3A900000000%2C%22req_id%22%3A%22ED6EC127-C359-4C18-A41E-3A5F6F499250u3183%22%2C%22rit%22%3A909946692%2C%22sign%22%3A%22D41D8CD98F00B204E9800998ECF8427E%22%2C%22uid%22%3A9724339963504202%2C%22ut%22%3A14%7D\u0026append=%7B%22openurl%22%3A%22%22%2C%22postdata%22%3A%5B%7B%22__type__%22%3A%22req_id%22%2C%22cid%22%3A1629848424707111%2C%22req_id%22%3A%22ED6EC127-C359-4C18-A41E-3A5F6F499250u3183%22%2C%22rit%22%3A909946692%7D%5D%7D",
			"resolution": "720x1280",
			"size": 5628226,
			"video_duration": 29.04,
			"video_url": "http://vd2.bdstatic.com/mda-jesntzw6569xqudw/mda-jesntzw6569xqudw.mp4"
		}
	}],
......
}
```
广告就是从上面的接口获取而来的，广告时长由`video_url`字段对应的mp4的时长决定


解决：用Charles的`Rewrite功能Body`替换`video_url`字段的值
![Rewrite功能-Type:Body](https://upload-images.jianshu.io/upload_images/281540-e80209bd2b222ac6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Match Value:
"video_url":"(.+)"}

Replace Value:
"video_url":"http://vd2.bdstatic.com/mda-jesntzw6569xqudw/mda.mp4"}
```

提供一个只有3秒的素材：
```
http://vd2.bdstatic.com/mda-jesntzw6569xqudw/mda-jesntzw6569xqudw.mp4
```
### 案例03
问题：`章鱼输入法`App有看广告得金币，没有像`案例02`那样找到相应的接口api返回广告视频的URL，怎样减少广告的时间？

思考：尝试替换广告视频的请求
比如广告视频的请求如下：
```
Get https://v3-ad.ixigua.com/.../video/m/.../toutiao.mp4

替换为只有3秒的视频地址
Get http://vd2.bdstatic.com/.../3seconds.mp4
```

解决：用Charles的`Rewrite功能URL`替换请求
![Rewrite功能-Type:URL](https://upload-images.jianshu.io/upload_images/281540-aa27f7180fd280a7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 案例04
问题：`扶我起来学数学`App的`作战休息区`有一个游戏`伪装者`，在上报成绩时，接口有hash字段，修改成绩字段，hash会验证不通过，达到了防止伪造成绩的功能，怎样在hash前伪造成绩？
![成绩上报接口](https://upload-images.jianshu.io/upload_images/281540-88406cb857175e0b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
思考：hash算法一般难破解，与其破解hash算法，不如转变思路：修改传入hash的值

```
value肯定与成绩相关
hash(value)
```
那value具体是怎么样的呢？通过抓包的数据可以判定为是个h5游戏，在js代码中可能包含相要的答案
![image](https://upload-images.jianshu.io/upload_images/281540-685fef5fb145856f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在浏览器中打开上面中的`game.html`验证确实是一个h5游戏：
![image](https://upload-images.jianshu.io/upload_images/281540-05b3b4529eb81455.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

解决：
* 用Charles的`Mirror功能`把抓包的数据自动保存为文件，再在文本编辑器中检查代码
* 在`game.html`文件中查找`rest/game_report`，因为有这个网络包，所以先查找这个关键字，结果如下：

```
function _gameReport(score, callBack, hash, time){
    var oAjax = null;
    //这里进行HTTP请求
    oAjax = new XMLHttpRequest();
    oAjax.open('post',HOSTURLAPI+"/rest/game_report"+"?uid="+UID+"&gameid="+GAMEID+"&score="+score+"&tm="+time+"&hash="+hash,true);
}
```
* 查找`_gameReport`函数的调用者

```
function gameReport(score, callBack){
    var timeData = new Date().getTime();
    var hashValue = UID+GAMEID+score+timeData;
    var hash = '';
    dsBridge.call('hashCode',hashValue,function(data){
        hash = data;
        _gameReport(score, callBack, hash, timeData);
    });
}

gameReport(b[0], function(success, old_score){})
```
js代码调用到App的`hashCode`方法，`hashValue = UID+GAMEID+score+timeData`
用Charles的`Rewrite功能Body`替换：

```
Match Value:
gameReport(b[0]

Replace Value:
gameReport('99'
```
![image](https://upload-images.jianshu.io/upload_images/281540-95aa5d5d7a03c5e6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 成绩已修改，hash验证已通过


## Mirror应用
文档：https://www.charlesproxy.com/documentation/tools/mirror/

> The Mirror tool saves responses to disk as they are received, creating a mirror copy of websites as you browse them.

Mirror把响应保存为文件到硬盘上

![image](https://upload-images.jianshu.io/upload_images/281540-9a4d5b1810506685.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Mirror保存下来的文件](https://upload-images.jianshu.io/upload_images/281540-64531455206484fc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





## No Caching 和 Block Cookies应用
在案例：`扶我起来学数学`App的`伪装者`游戏通过Rewrite功能修改了js文件中的内容；但有时js文件不是每次都会传输，而是使用了缓存，这时Rewrite功能就失效了，因为没有发生网络请求；通过`No Caching 和 Block Cookies`使网络请求每次都发生

## Map Local应用
| Map Local应用场景 |
| --- |
| 修改js文件来改变App行为: |
| 1. 使用No Caching 和 Block Cookies功能保证js文件通过网络请求加载到App |
| 2. 使用Mirror功能把js文件保存到电脑上 |
| 3. 使用Map Local功能使App加载修改后的js文件 |


