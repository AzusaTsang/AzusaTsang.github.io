---
title: Use Selenium and Pocket to scrap articles on websites automatically and accurately
layout: post
---

## Intro

[Pocket](https://app.getpocket.com) is a free saved list service website. If you save an article website in it, it will automatically transform the website to a pure article. Its result is far way better and more accurate than those Python scrapping toolkit like `goose3` and it support websites in various languages. So we can use it to scrap articles on websites very easily.

But because Pocket need to log in and need javascript to run the service, we cannot just use request or other similar toolkit to scrap the article directly. So we can use Selenium, which is a toolkit that can let you use code to control a web browser, to control a browser to do the log in operation then get the article we want to scrap.

## Steps

- First you need to save the articles in your Pocket account and install Selenium. Selenium support many programming languages. Here I use Python to install selenium.

```cmd
pip install selenium 
```

- Then use Selenium and [the Chromdrive](https://chromedriver.chromium.org/) (or any other web browser Selenium supports) to open the Pocket page.

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

driver_path='./chromedriver.exe'    # your chromedriver path
options = Options()
options.add_argument("user-data-dir=./chromecache/pocket")     
    # you can specify an user folder so that you can need not to log in each time
driver = webdriver.Chrome(driver_path,chrome_options=options)
driver.get('https://app.getpocket.com')
```

- After logging in, get the article list. The article links are at xpath `//article/a`. If a website can be transformed into an article by Pocket, the article link will starts with `https://getpocket.com/read/`.

```python
articles=[href for a in driver.find_elements_by_xpath('//article/a') 
          if (href:=a.get_attribute('href')).startswith('https://getpocket.com/read/')]
```

- Then we can get in each link to get the article texts. Here I use implicit wait to wait the webpages get loaded.

```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

for a in articles:
    driver.get(a)
    text=WebDriverWait(driver,10).until(EC.presence_of_element_located((By.XPATH,'//article/article'))).text
    print(text)   # or save the artcles any how you like
```

- You can also scrap the images in the articles.

After your first log in, you can just combine all the codes together and let it scrap automatically.

## Full Code

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver_path='./chromedriver.exe'    # your chromedriver path
options = Options()
options.add_argument("user-data-dir=./chromecache/pocket")     
    # you can specify an user folder so that you can need not to log in each time
driver = webdriver.Chrome(driver_path,chrome_options=options)
driver.get('https://app.getpocket.com')

WebDriverWait(driver,10).until(EC.presence_of_element_located((By.XPATH,'//article/a')))
    # wait until all '//article/a' are loaded
articles=[href for a in driver.find_elements_by_xpath('//article/a') 
          if (href:=a.get_attribute('href')).startswith('https://getpocket.com/read/')]

for a in articles:
    driver.get(a)
    text=WebDriverWait(driver,10).until(EC.presence_of_element_located((By.XPATH,'//article/article'))).text
        # wait until the article is loaded
    print(text)   # or save the artcles any how you like
```
