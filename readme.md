# 微信爬虫

爬取微信公众号文章，回复 评论数，点赞数

##### **爬取原理**：

​    通过搜狗搜索引擎获取微信文章的url，再通过微信客户端向某个用户发送消息（来自搜狗的url），鼠标打开该url浏览内容。在微信客户端打开浏览该url的过程中，通过mitmproxy拦截并解析文章的url，保存解析得到的数据。

##### 使用方法

- 第一步：打开settings.py， 设置数据存储位置

```
# 爬搜狗时用到chromedriver(通过selenium)
driver_path = "/home/brook/tools/chromedriver"

# mongodb保存数据
MONGO_URL = 'mongodb://user:password@ip'

# 第一步通过sogou获得的url存储数据库
URL_DB = 'test'
URL_COLL = "sogou_url"

# 最终微信数据保存位置
WECHAT_DB = "test"
WECHAT_COLL = "wechat"
```



- 第二步：先通过关键词在搜狗中搜索相关帖子的url(该url具有时效性)

```
[brook@localhost wechat]$ python3 url_from_sogou.py 白兰地
```

该过程需要手机微信扫码登录搜狗

- 第三步：上一步的url爬取完成后，在linux上启动mitmproxy

```
[brook@localhost wechat]$ mitmproxy -s parse_response.py
```

  上面的命令意思时，启动mitmproxy, 同时用parse_response.py解析拦截到的response，默认端口是8080，如果提示端口被占用， 加个 -p 参数来设置端口， 更多mitmproxy的用法参看 [mitmproxy文档](https://docs.mitmproxy.org/stable/)

  第一次使用的时候需要安装mitmproxy

```
[brook@localhost wechat]$ pip3 install mitmproxy
```

- 第四步：找一台安装有微信客户端的windows电脑，设置代理为linux的ip，同时还需要安装证书，证书下载地址[证书链接](http://mitm.it/)

- 第五步：windows上启动微信，打开一个好友的聊天界面（建议拖到右上角后不要再移动它），点击截图工具，移动鼠标定位聊天窗口的输入框的记录像素点位置， 修改windows_run.py文件

```
import time
from pywinauto.application import Application
import pywinauto
from utils import MongoPipeline
import settings
 
mongo_url = settings.MONGO_URL
mongo_db = settings.URL_DB
mongo_coll = 'brandy'

app = Application().Connect(title='微信')

wechat = app['微信']

  
with MongoPipeline(mongo_url, mongo_db, mongo_coll) as pipe:
    for item in pipe.find():
        url = item['url']
        # 微信聊天界面输入框的像素点位置（coords参数）
        pywinauto.mouse.click(button='left', coords=(1387, 544))
        wechat.TypeKeys(url)
        wechat.TypeKeys('{ENTER}')
        # 发送url后，聊天记录里就有你发送的url，同样用截图定位url位置（coords参数）
        pywinauto.mouse.click(button='left', coords=(1570, 417))
        time.sleep(8)
```

  上面的代码主要是通过pywinauto来模拟鼠标键盘，把url写到聊天界面的文本框，按下enter发送消息，鼠标点击发送的url查看url内容，这边用截图工具定位像素点位置后不可再移动好友聊天框

最后启动该脚本

```
python3 windows_run.py
```

