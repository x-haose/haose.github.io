---
slug: 4cfb56a9
author: "昊色居士"
title: "逆向学习之有道翻译"
description: 
date: 2022-08-02T22:04:57+08:00
image: rand
math: 
license: 
hidden: false
comments: true
draft: false
tags: [
    "爬虫", "Python", "逆向"
]
categories: [
    "Python", "逆向"
]
series:  [

]
---

# 逆向学习之有道翻译

## 声明
>本人所有逆向、破解及爬虫相关教程均是以纯技术的角度来探讨研究和学习，严禁使用教程中提到的技术去破解、滥用、伤害其他公司及个人的利益，以及将以下内容用于商业或者非法用途。

## 〇、为啥？

看到聚合翻译的软件，就心想咱要是也想搞一个类似的得咋弄呢？肯定要有几家常用翻译翻译的接口，然后去聚合调用才行。就想先拿[有道翻译](https://fanyi.youdao.com/)的接口试试水，并记录一下过程。

## 一、找到API接口

首先还是要在请求中找到我们要的api，使用的工具就是chrome浏览器开发者工具。

过滤选择下面的 `Fetch/XHR` 然后随便输入几个字符看看下面的请求



![](https://files.mdnice.com/user/29990/442f025e-af09-45e3-baf6-ad11c12d9b7f.png)



可以发现每次输入框发生改变都会调用图中圈起的api，在看一下参数和响应内容发现就是我们想要的。

## 二、观察接口

先看一下接口的详细情况：

- URL：[https://fanyi.youdao.com/translate_o](https://fanyi.youdao.com/translate_o)
- URL参数：
  
  
    | 参数名 | 参数值 | 猜测的用途 |
    | --- | --- | --- |
    | smartresult | dict | 固定的 |
    | smartresult | rule | 固定的 |
- Form Data参数：
  
  
    | 参数名 | 参数值 | 猜测的用途 |
    | --- | --- | --- |
    | i | aaa | 要翻译啥 |
    | from | AUTO | 从啥语言翻的 |
    | to | AUTO | 往哪翻的 |
    | smartresult | dict | 固定的 |
    | client | fanyideskweb | 固定的 |
    | salt | 16456680560149 | 像是时间戳 |
    | sign | 3d4a58570b60a9e4729d84ffc539ec23 | 加密字符串 |
    | lts | 1645668056014 | 像是时间戳 |
    | bv | 866ddc825824adb95a25e4ff4107f5a0 | 加密字符串 |
    | doctype | json | 固定的 |
    | version | 2.1 | 固定的 |
    | keyfrom | fanyi.web | 固定的 |
    | action | FY_BY_REALTlME | 固定的 |
- cookies：
  
  
    | OUTFOX_SEARCH_USER_ID | "-477347448@10.108.162.135" |
    | --- | --- |
    | JSESSIONID | aaaj-vk-PYAZR7X05Wk8x |
    | OUTFOX_SEARCH_USER_ID_NCOO | 49135154.68687222 |
    | JSESSIONID | abcQQkpI3SuAO4ybQ-k8x |
    | DICT_UGC | be3af0da19b5c5e6aa4e17bd8d90b28a| |
    | _ntes_nnid | 6c20d2b4a1d415e9f66b45f42896a2bb,1645411239529 |
    | SESSION_FROM_COOKIE | fanyiweb |
    | YOUDAO_FANYI_SELECTOR | OFF |
    | ___rl__test__cookies | 1645668063927 |
- 请求头：请求头看着是有很多的，但常用必传的就那么几个如：`Origin` 、`Referer` 、`User-Agent` ，这几个参数大部分是都会验证的，`Cookies` 是根据情况来传递的
  
    

上面列出来了这个请求的参数和参数用途的一些猜想，还有这个参数所携带的cookies。

对于参数来说有些参数就是固定的不能管就行，有些参数是经过计算出来的，要找到对能算法才能模拟。

对于cookies来说可能会有很多，不一定是都需要的，可以根据cookies的时效性，cookies的名称来粗略的判断一下，在通过请求模拟工具如 `postman` 、 `Apifox`等进一步的判断一下哪几个cookies才是真正要用到的

## 三、寻找参数

首先在chrome开发者工具里面找到对应的请求，然后再找到发起这个请求的方法，这里有两个方法可以找到对应的js代码：

- 通过network的Initiator标签这个查看请求栈


![](https://files.mdnice.com/user/29990/f358622a-7480-4b12-99e7-aa55882b3897.png)


- 通过打请求断点的方式，直接切换到 `Sources` 页面，在 `XHR/fech Breakpoints` 添加要监听的请求，请求中包括这个字符串就会断掉


![](https://files.mdnice.com/user/29990/9451ac2b-bfaf-404e-9957-43659099670e.png)


使用以上哪种方法都可以，然后改变输入框中字符串触发断点。观察 `Call Stack` 找到参数来源


![](https://files.mdnice.com/user/29990/75a55395-810f-48a0-ad97-d5986c20ae49.png)


断点可能会断在发请求的底层，要观察调用栈找到发起请求的业务代码是哪里


![](https://files.mdnice.com/user/29990/b078f695-a60a-428c-b548-8c9edb3d8945.png)


可以发现这里就是发起请求的地方，参数也能对应上，查看加密参数的生成会发现是由一个叫

`v.generateSaltSign(n)` 的方法生成的，下面看看这个方法里面实现的逻辑就可以了，在打一个断点达到方法处，然后可以单步调试进入进去或直接在 `Console` 面板输入函数名查看


![](https://files.mdnice.com/user/29990/e6a33e8c-dc3a-4643-98a5-883007333cda.png)


函数的逻辑如下：

```jsx
var n = e("./jquery-1.7");
    e("./utils");
    e("./md5");
**var r =** function(e) {
        var t = n.md5(navigator.appVersion)
          , r = "" + (new Date).getTime()
          , i = r + parseInt(10 * Math.random(), 10);
        return {
            ts: r,
            bv: t,
            salt: i,
            sign: n.md5("fanyideskweb" + e + i + "Ygy_4c=r#e#4EX^NUGUc5")
        }
    };
t.generateSaltSign = r
```

代码逻辑很简单，就是时间戳、md5加密和随机数之类的一些，直接模拟就好，参数 `e` 是要翻译的字符串，我们把它转成python的代码看看：

```python
import time
from hashlib import md5
from random import randint, uniform

def calc_md5(text):
    return md5(text.encode('utf-8')).hexdigest()

def generate_salt_sign(trans_text):
    app_version = (
        "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
        " Chrome/97.0.4692.99 Safari/537.36"
    )
    ts = int(time.time() * 1000)
    salt = f"{ts}{randint(0, 10)}"
    bv = calc_md5(app_version)
    return {
        'ts': str(ts),
        'bv': bv,
        'salt': salt,
        'sign': calc_md5(f"fanyideskweb{trans_text}{salt}Y2FYu%TNSbMCxc3t2u^XT")
    }
```

到此为止请求体中的参数全部找到了，我们用python请求一下看看：

```python
import time
from hashlib import md5
from random import randint, uniform
from pprint import pprint

import httpx

def calc_md5(text):
    return md5(text.encode('utf-8')).hexdigest()

def generate_salt_sign(msg):
    app_version = (
        "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
        " Chrome/97.0.4692.99 Safari/537.36"
    )
    ts = int(time.time() * 1000)
    salt = f"{ts}{randint(0, 10)}"
    bv = calc_md5(app_version)
    return {
        'ts': str(ts),
        'bv': bv,
        'salt': salt,
        'sign': calc_md5(f"fanyideskweb{msg}{salt}Y2FYu%TNSbMCxc3t2u^XT")
    }

def main(msg):
    sign_data = generate_salt_sign(msg)
    url = 'https://fanyi.youdao.com/translate_o?smartresult=dict&smartresult=rule'
    data = {
        'i': msg,
        'from': "AUTO",
        'to': "AUTO",
        'smartresult': 'dict',
        'client': 'fanyideskweb',
        'salt': sign_data['salt'],
        'sign': sign_data['sign'],
        'lts': sign_data['ts'],
        'bv': sign_data['bv'],
        'doctype': 'json',
        'version': "2.1",
        'keyfrom': "fanyi.web",
        'action': 'FY_BY_REALTlME'
    }
    headers = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
            " Chrome/97.0.4692.99 Safari/537.36"
        ),
        "Origin": "https://fanyi.youdao.com",
        "Referer": "https://fanyi.youdao.com/",
    }
    resp = httpx.post(url=url, data=data, headers=headers)
    pprint(resp.json())

if __name__ == '__main__':
    main("my")
```

让我们看下运行结果：

```json
{"errorCode": 50}
```

## 四、寻找Cookies

结果返回了一个错误，原因呢猜测是 `cookies` 的问题，可以从请求中复制一个到代码中确认下我们的猜想：

```python
headers = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
            " Chrome/97.0.4692.99 Safari/537.36"
        ),
        "Origin": "https://fanyi.youdao.com",
        "Referer": "https://fanyi.youdao.com/",
        "Cookie": 'OUTFOX_SEARCH_USER_ID_NCOO=273246508.64580524; OUTFOX_SEARCH_USER_ID="-699189264@10.108.162.139"; ___rl__test__cookies=1646103314327'
    }
```

```python
{'errorCode': 0,
 'smartResult': {'entries': ['',
                             'pron. 我的\r\n',
                             'int. 哎呀（表示惊奇等）；喔唷\r\n',
                             'n. （My）人名；（越）美；（老、柬）米\r\n'],
                 'type': 1},
 'translateResult': [[{'src': 'my', 'tgt': '我的'}]],
 'type': 'en2zh-CHS'}
```

不出所料返回了正确的结果，下一步就是逐步确认下需要的 `cookies` 是哪几个，上面的cookies一共是三组，可以挨个删除试试，最后得出结果是只需要 `OUTFOX_SEARCH_USER_ID` 的cookies，接下来就是找到这个cookies设置的地方，一个cookies如果会保存到浏览器中只会有两种方法：

- JavaScript代码保存
- 通过请求响应头中的 **`Set-Cookie`**

先看看第一种方法：

在浏览器调试页面按快捷键 :  `Ctrl+Shift+F` 全局搜索这个cookies，看看有没有哪里的代码设置这个cookies


![](https://files.mdnice.com/user/29990/3ca71a5a-3b7b-4007-bba3-e1f08c941d8e.png)


全局搜索后发现只有 `OUTFOX_SEARCH_USER_ID_NCOO` cookies的生成，没有`OUTFOX_SEARCH_USER_ID` 的生成。

然后我们接下来看第二个方法：

先把cookies清空


![](https://files.mdnice.com/user/29990/4b8e4d09-0313-42f8-9204-80f2a669cbd5.png)


再把请求过滤改为 `All` 然后清空所有请求，改变输入框内容后观察请求：


![](https://files.mdnice.com/user/29990/aa548e1a-2fe3-4b8a-9709-a61d58059c67.png)



![](https://files.mdnice.com/user/29990/da77cc61-5e65-4db3-ad41-2d137b6213fe.png)



![](https://files.mdnice.com/user/29990/eea25322-5e8e-415d-b735-64fb33476bf9.png)


可以看到清空cookies后它发起了一个不带目标 cookies`OUTFOX_SEARCH_USER_ID` 的请求吗，然后同样报错了，再看另一个请求：


![](https://files.mdnice.com/user/29990/322641b3-eac9-47f9-97af-c001af1b183f.png)


可以看到通过这个请求获取了设置cookies，接下来就是使用同样的方法查看这个请求的参数：


![](https://files.mdnice.com/user/29990/ae258aea-b8c1-4a3f-b97d-cc96b9832e7d.png)


同样去掉一些固定的参数后只有两个是我们需要模拟的，让我们来找一下相关代码：


![](https://files.mdnice.com/user/29990/a1e60f34-136e-4e39-8f58-360738edf32b.png)


会发现里面的变量 `c` 就是带入的参数， `t` 就是获取动态参数的方法，进入t函数内部就看看，找出目标变量：


![](https://files.mdnice.com/user/29990/88c62168-4a19-4a35-8b30-513c104b961e.png)


观察上面的代码后可以发现：

- 设置了cookie：___rl__test__cookies 值为当日时间戳
- 设置变量`G` 等于 `OUTFOX_SEARCH_USER_ID_NCOO` 的cookie，如果没有则设置为：

```jsx
2147483647 * Math.random()
```

- 设置了值 `_ncoo` 等于变量 `G`
- 设置了值 `_nssn` 登录变量 `F` 但 `F` 的值为空所有可以不管
- 设置了值 `_ntms` 为时间戳 `a`

所以获取cookies请求的参数已经找到了，先转成python代码：

```python
params = {
        "_npid": "fanyiweb",
        "_ncat": "event",
        "_ncoo": str(2147483647 * uniform(0, 1)),
        "nssn": "NULL",
        "_nver": "1.2.0",
        "_ntms": str(int(time.time() * 1000)),
        "_nhrf": "newweb_translate_text"
  }
```

提示：random.uniform(0, 1)是代替js种的Math.random(), 其作用都是获取0到1之间的浮点数

由于是调用一个请求获取的cookie然后要带入到另一个请求中，所以要两个请求保持同一个会话

## 五、编写代码

下面是示例代码：

```python
import time
from hashlib import md5
from random import randint, uniform
from pprint import pprint

from httpx import Client

def calc_md5(text):
    return md5(text.encode('utf-8')).hexdigest()

def generate_salt_sign(msg):
    app_version = (
        "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
        " Chrome/97.0.4692.99 Safari/537.36"
    )
    ts = int(time.time() * 1000)
    salt = f"{ts}{randint(0, 10)}"
    bv = calc_md5(app_version)
    return {
        'ts': str(ts),
        'bv': bv,
        'salt': salt,
        'sign': calc_md5(f"fanyideskweb{msg}{salt}Y2FYu%TNSbMCxc3t2u^XT")
    }

def main(msg):
    client = Client()

    params = {
        "_npid": "fanyiweb",
        "_ncat": "event",
        "_ncoo": str(2147483647 * uniform(0, 1)),
        "nssn": "NULL",
        "_nver": "1.2.0",
        "_ntms": str(int(time.time() * 1000)),
        "_nhrf": "newweb_translate_text"
    }
    client.get('https://rlogs.youdao.com/rlog.php', params=params)

    sign_data = generate_salt_sign(msg)
    url = 'https://fanyi.youdao.com/translate_o?smartresult=dict&smartresult=rule'
    data = {
        'i': msg,
        'from': "AUTO",
        'to': "AUTO",
        'smartresult': 'dict',
        'client': 'fanyideskweb',
        'salt': sign_data['salt'],
        'sign': sign_data['sign'],
        'lts': sign_data['ts'],
        'bv': sign_data['bv'],
        'doctype': 'json',
        'version': "2.1",
        'keyfrom': "fanyi.web",
        'action': 'FY_BY_REALTlME'
    }
    headers = {
        "User-Agent": (
            "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
            " Chrome/97.0.4692.99 Safari/537.36"
        ),
        "Origin": "https://fanyi.youdao.com",
        "Referer": "https://fanyi.youdao.com/",
        "Cookie": 'OUTFOX_SEARCH_USER_ID="-699189264@10.108.162.139"'
    }
    resp = client.post(url=url, data=data, headers=headers)
    pprint(resp.json())

if __name__ == '__main__':
    main("my")
```

运行结果：

```python
{'errorCode': 0,
 'smartResult': {'entries': ['',
                             'pron. 我的\r\n',
                             'int. 哎呀（表示惊奇等）；喔唷\r\n',
                             'n. （My）人名；（越）美；（老、柬）米\r\n'],
                 'type': 1},
 'translateResult': [[{'src': 'my', 'tgt': '我的'}]],
 'type': 'en2zh-CHS'}
```

这里我们来分析下响应结果：

- smartResult：词典的结果
- translateResult：翻译的结果，有来源和目标
- type：翻译的类型就是从什么语言翻译到什么语言

会看到翻译完后，界面上面的显示也会由 `自动检测语言` 改变为返回的内容，只不过是中文的，所以查一下 `en2zh-CHS` 会不会查到什么对应关系：


![](https://files.mdnice.com/user/29990/35c4f781-807c-42a5-afb6-cf07e3c5f73f.png)


可以看到对应关系是直接写在html中的，取出来就好，然后在对代码做一些小调整：

```python
import asyncio
import time
from hashlib import md5
from random import randint, uniform

from httpx import AsyncClient

language_dict = {
    "zh-CHS2en": "中文  »  英语",
    "en2zh-CHS": "英语  »  中文",
    "zh-CHS2ja": "中文  »  日语",
    "ja2zh-CHS": "日语  »  中文",
    "zh-CHS2ko": "中文  »  韩语",
    "ko2zh-CHS": "韩语  »  中文",
    "zh-CHS2fr": "中文  »  法语",
    "fr2zh-CHS": "法语  »  中文",
    "zh-CHS2de": "中文  »  德语",
    "de2zh-CHS": "德语  »  中文",
    "zh-CHS2ru": "中文  »  俄语",
    "ru2zh-CHS": "俄语  »  中文",
    "zh-CHS2es": "中文  »  西班牙语",
    "es2zh-CHS": "西班牙语  »  中文",
    "zh-CHS2pt": "中文  »  葡萄牙语",
    "pt2zh-CHS": "葡萄牙语  »  中文",
    "zh-CHS2it": "中文  »  意大利语",
    "it2zh-CHS": "意大利语  »  中文",
    "zh-CHS2vi": "中文  »  越南语",
    "vi2zh-CHS": "越南语  »  中文",
    "zh-CHS2id": "中文  »  印尼语",
    "id2zh-CHS": "印尼语  »  中文",
    "zh-CHS2ar": "中文  »  阿拉伯语",
    "ar2zh-CHS": "阿拉伯语  »  中文",
    "zh-CHS2nl": "中文  »  荷兰语",
    "nl2zh-CHS": "荷兰语  »  中文",
    "zh-CHS2th": "中文  »  泰语",
    "th2zh-CHS": "泰语  »  中文"
}

def calc_md5(text):
    return md5(text.encode('utf-8')).hexdigest()

def generate_salt_sign(trans_text):
    app_version = (
        "5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
        " Chrome/97.0.4692.99 Safari/537.36"
    )
    ts = int(time.time() * 1000)
    salt = f"{ts}{randint(0, 10)}"
    bv = calc_md5(app_version)
    return {
        'ts': str(ts),
        'bv': bv,
        'salt': salt,
        'sign': calc_md5(f"fanyideskweb{trans_text}{salt}Y2FYu%TNSbMCxc3t2u^XT")
    }

class YouDaoDict(object):
    language_type = ""
    translate_result = ""
    dict_result = ""

    def __init__(
            self,
            trans_text: str,
            trans_from: str = "AUTO",
            trans_to: str = "AUTO"
    ):
        """
        初始化
        Args:
            trans_text: 翻译的文本
            trans_from:
            trans_to:
        """
        self.trans_text = trans_text
        self.trans_from = trans_from
        self.trans_to = trans_to

        headers = {
            "User-Agent": ("Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)"
                           " Chrome/97.0.4692.99 Safari/537.36"),
            "Origin": "https://fanyi.youdao.com",
            "Referer": "https://fanyi.youdao.com/",
        }
        self.client = AsyncClient(headers=headers)

    async def close(self):
        await self.client.aclose()

    async def _set_cookie(self):
        params = {
            "_npid": "fanyiweb",
            "_ncat": "event",
            "_ncoo": str(2147483647 * uniform(0, 1)),
            "nssn": "NULL",
            "_nver": "1.2.0",
            "_ntms": str(int(time.time() * 1000)),
            "_nhrf": "newweb_translate_text"
        }
        await self.client.get('https://rlogs.youdao.com/rlog.php', params=params)

    async def translate_text(self):
        await self._set_cookie()

        sign_data = generate_salt_sign(self.trans_text)
        data = {
            'i': self.trans_text,
            'from': "AUTO",
            'to': "AUTO",
            'smartresult': 'dict',
            'client': 'fanyideskweb',
            'salt': sign_data['salt'],
            'sign': sign_data['sign'],
            'lts': sign_data['ts'],
            'bv': sign_data['bv'],
            'doctype': 'json',
            'version': "2.1",
            'keyfrom': "fanyi.web",
            'action': 'FY_BY_REALTlME'
        }
        url = 'https://fanyi.youdao.com/translate_o?smartresult=dict&smartresult=rule'

        resp = await self.client.post(url=url, data=data)
        resp_data = resp.json()

        self.language_type = language_dict[resp_data['type']]
        self.translate_result: dict = resp_data['translateResult'][0][0]
        try:
            self.dict_result: str = ''.join(resp_data['smartResult']['entries'])
        except (KeyError, TypeError):
            self.dict_result = ""

    def __repr__(self):
        translate_result_str = (
            f"{self.language_type}\n"
            f"翻译结果：{self.translate_result['src']} » {self.translate_result['tgt']}\n"
            f"词典结果：\n{self.dict_result}"
        )

        return translate_result_str

async def main():
    youdao = YouDaoDict(trans_text="my")
    await youdao.translate_text()
    await youdao.close()
    print(youdao)

if __name__ == '__main__':
    asyncio.run(main())
```

运行结果：

```
英语  »  中文
翻译结果：my » 我的
词典结果：
pron. 我的
int. 哎呀（表示惊奇等）；喔唷
n. （My）人名；（越）美；（老、柬）米
```

## 六、结束

本篇文章呢难度相对不大，但涉及到的知识点还有有一些的：

- chrome浏览器开发者工具调试方法，如：观察请求、追踪请求代码栈、打请求断点、全局搜索代码、清空cookies等
- 如何分析js代码
- 浏览器cookies保存的方式
- python中保持同一个会话以带入上一个请求中的cookies