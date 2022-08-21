# SELENIUM-NOTE

#### 安装

将下载的驱动文件放在python和浏览器的根目录

#### 简单使用

```python
#打开百度 搜素python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get('https://www.baidu.com')
# driver.find_element_by_id('kw').send_keys('python')
# driver.find_element_by_id('su').click()
driver.find_element(by=By.ID,value='kw').send_keys('python')
driver.find_element(by=By.ID,value='su').click()
time.sleep(6)
driver.quit()
```

#### driver对象常用属性或方法

```python
from selenium import webdriver
url = 'https://www.baidu.com'
driver = webdriver.Chrome()
driver.get(url=url)

# 显示源码
print(driver.page_source)
# 响应对应的URL
print(driver.current_url)
# 显示标题
print(driver.title)
# 回退
driver.back()
# 前进
driver.forward()
# 关闭当前标签页
driver.close()
# 关闭浏览器
driver.quit()
# 页面截图
driver.save_screenshot('baidu.png')
```

#### 简单元素操作

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

url = 'https://www.zhipin.com/chengdu/'
driver = webdriver.Chrome()
driver.get(url=url)

el_list = driver.find_elements(By.XPATH,value='//*[@id="main"]/div/div[3]/ul[1]/li/div/a/div/p[1]')
for el in el_list:
    print(el.text)

driver.find_element(By.XPATH,value='//*[@id="wrap"]/div[3]/div/div[1]/div[1]/form/div[2]/p/input').send_keys('python')
driver.find_element(By.CLASS_NAME,value='btn-search').click()
# driver.quit()
```

#### cookies操作

```python
from selenium import webdriver
from selenium.webdriver.common.by import By

url = 'https://www.zhipin.com/chengdu/'
driver = webdriver.Chrome()
driver.get(url=url)
# 删除cookies的某一个属性
driver.delete_cookie('lastCity')
# 删除所有
driver.delete_all_cookies()
# 获取保存为字典
cookies = {data['name']:data['value'] for data in driver.get_cookies()}
# cookies = {}
# for cookie in driver.get_cookies():
#     cookies[cookie['name']] = cookie['value']
print(cookies)
```

#### 执行JS代码

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import time

url = 'https://cd.lianjia.com/'
driver = webdriver.Chrome()
driver.get(url)
# 滚动条的拖动
js = 'scrollTo(0,1000)'
driver.execute_script(js)
time.sleep(2)
el_button = driver.find_element(By.XPATH,value='//*[@id="xiaoqulist"]/li[1]/a')
print(el_button)
```

#### 页面等待

```python
# 强制等待
time.sleep(3)
# 隐式等待 所有元素的定位操作等待
driver.implicitly_wait(10)
# 显式等待
from selenium import webdriver  
from selenium.webdriver.support.wait import WebDriverWait  
from selenium.webdriver.support import expected_conditions as EC  
from selenium.webdriver.common.by import By 

driver = webdriver.Chrome()

driver.get('https://www.baidu.com')

# 显式等待
WebDriverWait(driver, 20, 0.5).until(
    EC.presence_of_element_located((By.LINK_TEXT, '好123')))  
# 参数20表示最长等待20秒
# 参数0.5表示0.5秒检查一次规定的标签是否存在
# EC.presence_of_element_located((By.LINK_TEXT, '好123')) 表示通过链接文本内容定位标签
# 每0.5秒一次检查，通过链接文本内容定位标签是否存在，如果存在就向下继续执行；如果不存在，直到20秒上限就抛出异常

print(driver.find_element_by_link_text('好123').get_attribute('href'))
driver.quit()
```

#### Chrome无头模式

```python
#selenium不再支持PhantomJS 使用Chrome无头模式代替
from selenium import webdriver
#创建Chrome浏览器设置变量
chrome_options = webdriver.ChromeOptions()
#无界面模式
chrome_options.add_argument('--headless')
chrome_options.add_argument('--disable-gpu')
#实例化Chrome driver
driver=webdriver.Chrome(options=chrome_options)
```

#### 代理/UA配置

```python
from selenium import webdriver
options = webdriver.ChromeOptions() # 创建一个配置对象
options.add_argument('--proxy-server=http://202.20.16.82:9527') # 使用代理ip
chrome_options.add_argument('--user-agent=Mozilla/5.0 ...')# 配置UA
driver = webdriver.Chrome(options=options) # 实例化带有配置的driver对象
```

