# SCRAPY-NOTE

##### 创建项目

```powershell
scrapy startproject tutorial
```

##### 生成爬虫模板文件

```powershell
scrapy genspider itcast
```

##### 起始地址

```python
 def start_requests(self):
        urls = [
            'https://www.itcast.cn/channel/teacher.shtml#ajavaee'
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)
```

##### 快捷方式（默认实现start_requests()）

```python
start_urls = ['https://www.itcast.cn/channel/teacher.shtml#ajavaee']
```

##### 启动爬虫

```powershell
scrapy crawl itcast
```

##### parse方法写爬取逻辑

```python
    #使用get()和getall()
    def parse(self, response):
        #爬取教师信息
        node_list = response.xpath('//div[@class="li_txt"]')
        print(len(node_list))

        for node in node_list:
            item = ScrapyTestItem()

            # item['name'] = node.xpath('./h3/text()').extract_first()
            # item['title'] = node.xpath('./h4/text()')[0].extract()
            # item['desc'] = node.xpath('./p/text()')[0].extract()
            item['name'] = node.xpath('./h3/text()').get()
            item['title'] = node.xpath('./h4/text()')[0].get()
            item['desc'] = node.xpath('./p/text()').get()
            item['test'] = node.xpath('./p/text()').get(default='not-found')
            # item['test'] = node.xpath('./p/text()').getall()
            yield item           
```

#### response响应对象的常用属性

- response.url：当前响应的url地址
- response.request.url：当前响应对应的请求的url地址
- response.headers：响应头
- response.requests.headers：当前响应的请求头
- response.body：响应体，也就是html代码，byte类型
- response.status：响应状态码

##### 保存数据

```python
# 重写管道类的process_item方法
 def process_item(self, item, spider):
        item = dict(item)
        json_data = json.dumps(item,ensure_ascii=False) + ',\n'
        self.file.write(json_data)
        return item
```

```python
# settings.py中启用管道
ITEM_PIPELINES = {
   'scrapy_test.pipelines.ScrapyTestPipeline': 300,
}
```

##### 数据建模

```python
#方便检测出字段错误

# items.py
class ScrapyTestItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    name = scrapy.Field()
    title = scrapy.Field()
    desc = scrapy.Field()
    test = scrapy.Field()
    pass

# myspider.py
from scrapy_test.items import ScrapyTestItem
def parse(self, response):
	item = ScrapyTestItem()
    '''
    '''

# pipelines.py
class ScrapyTestPipeline(object):
    def process_item(self, item, spider):
        item = dict(item)
        '''
        '''
        return item
```

##### meta参数的使用

```python
    # 当前页的爬取方法
    def parse(self, response, *args ,**kwargs):
        node_list = response.xpath('//*[@class="position-tb"]/tbody/tr')
        for num, node in enumerate(node_list):
            if num % 2 == 0:
                item = WangyiItem()
                item['name'] = node.xpath('./td[1]/a/text()').get()
                item['link'] = response.urljoin(node.xpath('./td[1]/a/@href').get())
                # item['link'] = 'https://hr.163.com/' +  node.xpath('./td[1]/a/@href')
                item['depart'] = node.xpath('./td[2]/text()').get()
                item['category'] = node.xpath('./td[3]/text()').get()
                item['type'] = node.xpath('./td[4]/text()').get()
                item['address'] = node.xpath('./td[5]/text()').get()
                item['num'] = node.xpath('./td[6]/text()').get().strip()
                item['date'] = node.xpath('./td[7]/text()').get()
                # yield item
                # 构建详情页面的请求
                yield scrapy.Request(
                    url=item['link'],
                    callback=self.parse_detail,
                    meta = {'item': item} #将当前页提取的数据传递
                )

        # 模拟翻页
        part_url = response.xpath('/html/body/div[2]/div[2]/div[2]/div/a[last()]/@href').get()
        # 判断最后一页
        if part_url != 'javascript:void(0)':
            next_url = response.urljoin(part_url)
            # 构建请求对象返回给引擎
            yield scrapy.Request(
                url=next_url,
                callback=self.parse,
            )
            
        
     # 翻页方法
    def parse_detail(self,response):
        item = response.meta['item']
        item['duty'] = response.xpath('/html/body/div[2]/div[2]/div[1]/div/div/div[2]/div[1]/div/text()').getall()
        item['require'] = response.xpath('/html/body/div[2]/div[2]/div[1]/div/div/div[2]/div[2]/div/text()').getall()
        yield item     
```

##### 使用FormRequest

```python
class GittwoSpider(scrapy.Spider):
    name = 'gittwo'
    allowed_domains = ['github.com']
    start_urls = ['https://github.com/login']

    def parse(self, response ,*args, **kwargs):
        token = response.xpath('//*[@id="login"]/div[4]/form/input[1]/@value').get()

        post_data = {
            'commit': 'Sign in',
            'authenticity_token': token,
            'login': 'HighGrooce',
            'password': 'wjy864755286',
            'trusted_device': '',
            'webauthn-support': 'supported',
            'webauthn-iuvpaa-support': 'unsupported',
            'return_to': 'https://github.com/login',
            'allow_signup': '',
            'client_id': '',
            'integration': '',
            'required_field_52f2': '',
            'timestamp': '1658628652720',
            'timestamp_secret': '2f65442d13b3020ec74fcdfa104ffce941a8cd83bb50e966795c3807ce0e67c0'
        }
        # print(post_data)
        yield scrapy.FormRequest(
            url='https://github.com/session',
            callback=self.after_login,
            formdata=post_data
        )

    def after_login(self, response):
        yield scrapy.Request(
            url='https://github.com/HighGrooce',
            callback=self.check_login
        )

    def check_login(self, response):
        print(response.xpath('/html/head/title/text()').get())
```

##### 使用cookies

```python
# 要将获取的cookies转换为字典
class GitloginSpider(scrapy.Spider):
    name = 'gitlogin'
    allowed_domains = ['github.com']
    start_urls = ['https://github.com/HighGrooce']

    def start_requests(self):
        url = self.start_urls[0]
        temp = '爬取的cookie'
        cookies = {data.split("=")[0]:data.split("=")[-1] for data in temp.split('; ')}
        yield scrapy.Request(
            url=url,
            callback=self.parse,
            cookies=cookies
        )
```

##### 管道的高级使用

```python
# piplines.py 包含导入MongoDB
import json
from pymongo import MongoClient


class WangyiPipeline:
    def open_spider(self, spider):
        if spider.name == 'job':
            self.file = open('wangyi.json','w')

    def process_item(self, item, spider):
        if spider.name == 'job':
            item = dict(item)
            str_data = json.dumps(item,ensure_ascii=False) + ',\n'
            self.file.write(str_data)
        return item

    def close_spider(self, spider):
        if spider.name == 'job':
            self.file.close()

class Wangyi2Pipeline:
    def open_spider(self, spider):
        if spider.name == 'job2':
            self.file = open('wangyi2.json','w')

    def process_item(self, item, spider):
        if spider.name == 'job2':
            item = dict(item)
            str_data = json.dumps(item,ensure_ascii=False) + ',\n'
            self.file.write(str_data)
        return item

    def close_spider(self, spider):
        if spider.name == 'job2':
            self.file.close()

class MongoPipline(object):
    def open_spider(self, spider):
        if spider.name == 'job2':
            self.client = MongoClient('127.0.0.1',27017)
            self.db = self.client['client']
            self.col = self.db['wangyi']

    def process_item(self, item, spider):
        if spider.name == 'job2':
            data = dict(item)
            self.col.insert_many(data)
            return item

    def close_spider(self, spider):
        if spider.name == 'job2':
            self.client.close()
    
#开启管道 settings.py
ITEM_PIPELINES = {
   'wangyi.pipelines.WangyiPipeline': 300,
   'wangyi.pipelines.Wangyi2Pipeline': 301,
   'wangyi.pipelines.MongoPipline': 299,
}

```

#### 下载中间件-随机UA

```python
# middlewares.py
class RandomUserAgent(object):
    def process_request(self, request, spider):
        # print(request.headers['User-Agent'])
        ua = random.choice(USER_AGENT_LIST)
        request.headers['User-Agent'] = ua

# settings.py
# UA列表
USER_AGENT_LIST = [
    "Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_6_8; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
    "Mozilla/5.0 (Windows; U; Windows NT 6.1; en-us) AppleWebKit/534.50 (KHTML, like Gecko) Version/5.1 Safari/534.50",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; rv:38.0) Gecko/20100101 Firefox/38.0",
    "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E; .NET CLR 2.0.50727; .NET CLR 3.0.30729; .NET CLR 3.5.30729; InfoPath.3; rv:11.0) like Gecko",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.6; rv:2.0.1) Gecko/20100101 Firefox/4.0.1",
    "Mozilla/5.0 (Windows NT 6.1; rv:2.0.1) Gecko/20100101 Firefox/4.0.1",
    "Opera/9.80 (Macintosh; Intel Mac OS X 10.6.8; U; en) Presto/2.8.131 Version/11.11",
    "Opera/9.80 (Windows NT 6.1; U; en) Presto/2.8.131 Version/11.11",
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_7_0) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.56 Safari/535.11",
    "Mozilla/5.0 (iPhone; U; CPU iPhone OS 4_3_3 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5",
    "Mozilla/5.0 (iPod; U; CPU iPhone OS 4_3_3 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5",
    "Mozilla/5.0 (iPad; U; CPU OS 4_3_3 like Mac OS X; en-us) AppleWebKit/533.17.9 (KHTML, like Gecko) Version/5.0.2 Mobile/8J2 Safari/6533.18.5",
    "Mozilla/5.0 (Linux; U; Android 2.3.7; en-us; Nexus One Build/FRF91) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1",
    "MQQBrowser/26 Mozilla/5.0 (Linux; U; Android 2.3.7; zh-cn; MB200 Build/GRJ22; CyanogenMod-7) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1",
    "Opera/9.80 (Android 2.3.4; Linux; Opera Mobi/build-1107180945; U; en-GB) Presto/2.8.149 Version/11.10",
    "Mozilla/5.0 (Linux; U; Android 3.0; en-us; Xoom Build/HRI39) AppleWebKit/534.13 (KHTML, like Gecko) Version/4.0 Safari/534.13",
    "Mozilla/5.0 (BlackBerry; U; BlackBerry 9800; en) AppleWebKit/534.1+ (KHTML, like Gecko) Version/6.0.0.337 Mobile Safari/534.1+",
    "Mozilla/5.0 (hp-tablet; Linux; hpwOS/3.0.0; U; en-US) AppleWebKit/534.6 (KHTML, like Gecko) wOSBrowser/233.70 Safari/534.6 TouchPad/1.0",
    "Mozilla/5.0 (SymbianOS/9.4; Series60/5.0 NokiaN97-1/20.0.019; Profile/MIDP-2.1 Configuration/CLDC-1.1) AppleWebKit/525 (KHTML, like Gecko) BrowserNG/7.1.18124",
    "Mozilla/5.0 (compatible; MSIE 9.0; Windows Phone OS 7.5; Trident/5.0; IEMobile/9.0; HTC; Titan)",
    "UCWEB7.0.2.37/28/999",
    "NOKIA5700/ UCWEB7.0.2.37/28/999",
    "Openwave/ UCWEB7.0.2.37/28/999",
	"Mozilla/6.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Version/8.0 Mobile/10A5376e Safari/8536.25",
]
# 开启中间件
DOWNLOADER_MIDDLEWARES = {
   'douban.middlewares.RandomUserAgent': 543,
}
```

#### 下载中间件-随机IP代理

```python
# middlewares.py
class RandomProxy(object):
    def process_request(self, request, spider):
        proxy = random.choice(PROXY_LIST)
        print(proxy)
        if 'user_passwd' in proxy:
            # 对账号密码进行编码
            b64_up = base64.b64encode(proxy['user_passwd'].encode())
            # 设置认证
            request.headers['Proxy-Authorization'] = 'Basic ' + b64_up.decode()
            # 设置代理
            request.meta['proxy'] = proxy['ip_port']
        else:
            # 设置代理
            request.meta['proxy'] = proxy['ip_port']

# settings.py
PROXY_LIST = [
    {"ip_port":"47.106.105.236:80"},
    {"ip_port":"223.82.60.202:8060"},
    {"ip_port":"101.200.127.149:3129"},
]

DOWNLOADER_MIDDLEWARES = {
   'douban.middlewares.RandomProxy': 542
}


            
```

#### 下载中间件-selenium渲染

```python
# middlewares.py
from selenium import webdriver
import time
from scrapy.http import HtmlResponse

class SeleniumMiddleware(object):

    def process_request(self,request,spider):
        url = request.url

        if 'str_signal' in url:
            driver = webdriver.Chrome()
            driver.get(url)
            time.sleep(2)
            data = driver.page_source
            driver.close()
            res = HtmlResponse(url=url,request=request,body=data,encoding='utf-8')
            return res
  
# settings.py
DOWNLOADER_MIDDLEWARES = {
   'douban.middlewares.SeleniumMiddleware': 542
}
```

##### SCRAPY SHELL

##### SCRAPY命令行工具

```powershell
# 查看可用命令
scrapy -h
```

![scrapy -h](C:\Users\W519\Desktop\Note\img\scrapy -h.png)

```powershell
# 创建项目 scrapy startproject <project_name> [project_dir]
$ scrapy startproject myproject 

# 创建爬虫 scrapy genspider [-t template] <name> <domain or URL>
$ scrapy genspider -l
Available templates:
  basic
  crawl
  csvfeed
  xmlfeed

$ scrapy genspider example example.com
Created spider 'example' using template 'basic'

$ scrapy genspider -t crawl scrapyorg scrapy.org
Created spider 'scrapyorg' using template 'crawl'

# 执行爬虫 scrapy crawl <spider> 必须先有项目
$ scrapy crawl myspider
[ ... myspider starts crawling ... ]

# 契约检查 Run contract checks
$ scrapy check -l
first_spider
  * parse
  * parse_item
second_spider
  * parse
  * parse_item

$ scrapy check
[FAILED] first_spider:parse_item
>>> 'RetailPricex' field is missing

[FAILED] first_spider:parse
>>> Returned 92 requests, expected 0..4

# 显示当前项目爬虫数 scrapy list
$ scrapy list
spider1
spider2

# 编辑调试爬虫 使用环境变量中定义的编辑器 scrapy edit <spider>
$ scrapy edit spider1

# 下载指定URL写入标准输出 scrapy fetch <url>
# --spider=SPIDER：绕过蜘蛛自动检测并强制使用特定蜘蛛
# --headers: 打印响应的 HTTP 标头而不是响应的正文
# --no-redirect：不遵循 HTTP 3xx 重定向（默认是遵循它们）


```

##### scrapy中三个内置对象

request请求对象：由url，method，post_data，headers等构成
response响应对象：由url，body，status，headers等构成
item数据对象：本质是一个字典

##### SCRAPY 中response.body 与 response.text区别

body:byte类型。text:string类型，text 是response.body经过response.encoding经过解码得到(response.text = response.body.decode(response.encoding)