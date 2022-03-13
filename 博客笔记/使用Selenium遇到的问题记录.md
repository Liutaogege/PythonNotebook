# 问题汇总

## 1.找不到元素报错

原因：找到的元素没有在页面中加载出来

```
selenium.common.exceptions.NoSuchElementException: Message: no such element: Unable to locate element: {"method":"css selector","selector":"[id="wizNoteLoginUserId"]"}
```

**解决办法**：

1. 显式等待:WebDriverWait方法
   
   ```python
   from selenium import webdriver
   from selenium.webdriver.common.by import By
   from selenium.webdriver.support.ui import WebDriverWait
   from selenium.webdriver.support import expected_conditions as EC
   
   driver = webdriver.Chrome()
   driver.get("https://www.baidu.com/")
   try:
       element = WebDriverWait(driver, 10).until(
           EC.presence_of_element_located((By.XPATH, '//*[@id="su"]'))
       )
       text = driver.page_source
       print("text", text)
   finally:
       driver.quit()
   ```

2. 隐式等待:implicitly_wait方法
   
   ```python
   from selenium import webdriver
   
   driver = webdriver.Chrome()
   driver.implicitly_wait(10) # seconds
   driver.get("https://www.baidu.com/")
   myDynamicElement = driver.find_element_by_xpath('//*[@id="su"]')
   text = driver.page_source
   print("text", text)
   ```

3. 沉睡等待：time.sleep()方法
   
   这个方法就是python自带的，如果以上两种方法都不管用，才采用这种

## 2.点击元素报错

```
selenium.common.exceptions.ElementClickInterceptedException: Message: element click intercepted: Eleme
```

原因分析：虽然有这个元素，但是这个元素不在当前窗口，点击不到它，所以需要想办法使得该元素出现在当前窗口

解决办法：使得该元素滚动到当前窗口的顶部

```python
# root即是这个元素
from selenium import webdriver
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains

root = browser.find_elements_by_class_name('MuiListItemIcon-root')
browser.execute_script("arguments[0].scrollIntoView();", root)

# 按钮点击办法
for n in range(3):
     browser.execute_script("arguments[0].scrollIntoView();", note)
     ActionChains(browser).key_down(Keys.ARROW_DOWN).perform()
```

## 3.进入嵌套的iframe元素获取到目标元素以后，如果不退出来会获取不到元素报错

```python
# 进入嵌套的iframe方法：switch_to.frame
new_browser.switch_to.frame(self.browser.find_element_by_class_name("wiz-editor-iframe"))

# 回到父级
browser.switch_to.parent_frame()
```

## 实例：爬取为知笔记

注：爬取html的效果比较好，但是转换为md文件后效果很差

```python
from selenium import webdriver
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains
import html2text as ht
import time
import threading
import os

glock = threading.Lock()


class Downloading:
    browser = webdriver.Chrome(executable_path="/Users/jared/Downloads/chromedriver")
    text_maker = ht.HTML2Text()
    page_list = []
    clicked_icons = []
    not_downloaded = []
    login_name = ''
    passwd = ''

    def __init__(self, name, passwd):
        self.login_name = name
        self.passwd = passwd
        self.icons = self.browser.find_elements_by_class_name('MuiListItemIcon-root')[10:17]

    def get_icons(self):
        icons_all = self.browser.find_elements_by_class_name('MuiListItemIcon-root')
        icn_len = len(icons_all)
        icons = icons_all[10: icn_len - 1]
        return icons

    def click_icons(self, icons):
        while True:
            try:
                for icon in icons:
                    if icon not in self.clicked_icons:
                        icon.click()
                        time.sleep(0.5)
                        self.browser.implicitly_wait(5)
                        self.clicked_icons.append(icon)
            except:
                icons = self.get_icons()
            if len(icons) < 48:
                icons = self.get_icons()
            if len(icons) == 48:
                print("结束")
                break

    def login(self):
        self.browser.get('https://www.wiz.cn/wapp/tag/b948d637-6680-4438-9e2b-a8266432ea1f')
        self.browser.find_element_by_id('wizNoteLoginUserId').send_keys(self.login_name)
        self.browser.find_element_by_id('wizNoteLoginPassword').send_keys(self.passwd)
        self.browser.find_element_by_class_name('MuiButton-contained').click()
        WebDriverWait(self.browser, 20).until(
            EC.visibility_of_element_located((By.CLASS_NAME, 'MuiListItemText-inset')))
        time.sleep(2)

    def get_note_list(self):
        self.click_icons(self.icons)
        all_roots = self.browser.find_elements_by_class_name('MuiListItemText-inset')[1:]
        not_downloaded = []
        for root in all_roots:
            root_name = root.find_element_by_class_name('MuiListItemText-primary').text
            self.browser.execute_script("arguments[0].scrollIntoView();", root)
            root.click()
            # 点击以后死睡2秒，因为有些元素加载很慢
            time.sleep(2)
            note_list_root = self.browser.find_elements_by_class_name('MuiList-root')[-1]
            note_list = note_list_root.find_elements_by_class_name('MuiListItem-root')
            if not note_list:
                continue
            for index, note in enumerate(note_list):
                self.browser.implicitly_wait(2)
                self.browser.execute_script("arguments[0].scrollIntoView();", note)
                note.click()
                title_name = note.find_element_by_class_name('MuiTypography-root').text.split('\n')[0]
                css_sel = 'p[title="' + title_name + '"]'

                # 有可能有些因为某个特殊题目没有给加上去，报错的continue，手动去下载
                try:
                    WebDriverWait(self.browser, 2).until(
                        EC.visibility_of_element_located((By.CSS_SELECTOR, css_sel)))
                except Exception as e:
                    not_downloaded.append(title_name)
                    continue
                new_browser = self.browser
                new_browser.switch_to.frame(self.browser.find_element_by_class_name("wiz-editor-iframe"))
                page_source = new_browser.page_source
                self.page_list.append({'page_source': page_source, 'root_name': root_name, 'title_name': title_name})
                new_browser.switch_to.parent_frame()
                # for n in range(3):
                #     new_browser.execute_script("arguments[0].scrollIntoView();", note)
                    # ActionChains(new_browser).key_down(Keys.ARROW_DOWN).perform()
        self.not_downloaded = not_downloaded

    def download_page(self):
        while True:
            glock.acquire()
            if len(self.page_list) == 0:
                glock.release()
                continue
            else:
                page_dict = self.page_list.pop()
                glock.release()
                # 修改文件名
                page_source = page_dict['page_source']
                title_name = page_dict['title_name'].replace('/', '|')
                file_path = '/Users/jared/Downloads/weizhi/' + page_dict['root_name']
                if not os.path.exists(file_path):
                    os.makedirs(file_path)

                try:
                    text_maker_text = self.text_maker.handle(page_source)
                    file_name = file_path + '/' + title_name + '.md'
                    with open(file_name, 'w+', encoding='UTF-8') as f:
                        f.write(text_maker_text)
                except:
                    file_name = file_path + '/' + title_name + '.html'
                    with open(file_name, 'w+', encoding='UTF-8') as f:
                        f.write(page_source)

    def main(self):
        self.login()
        for x in range(4):
            consumer = threading.Thread(target=self.download_page)
            consumer.start()
        self.get_note_list()
        print(self.not_downloaded)


if __name__ == '__main__':
    login_name = 'xxx'
    login_passwd = 'xxx'
    doan = Downloading()
    doan.main()
```
