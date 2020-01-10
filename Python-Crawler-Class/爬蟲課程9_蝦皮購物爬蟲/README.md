# Python 網路爬蟲課程 9

:::warning
:warning: 注意事項 :warning:
* 課程內容以 Python 爬蟲為主軸，Python 基礎不會著墨過深，基礎的部份遇到困難請同學善用 [Google](https://www.google.com/)
* 課程內容是本人的嘔心瀝血之作，若有引用請標明出處
* 本文最後編輯日期：2020/01/10
:::

## 課堂使用環境安裝 :computer:
* 本課程使用 [Anaconda](https://www.anaconda.com/download/) 做為教學環境
* 請下載對應電腦作業系統的 Python 3.6 版本
* 詳細安裝流程 :point_right: [Python Anaconda 環境安裝教學](/h5XtZtlRSB2TQP-marFL5g)
* [Python 網路爬蟲課程 2](/8ZVF56fBQ-ydJC5Sj0Gubg) 之後的課程均使用 [Google Chrome](https://www.google.com/intl/zh-TW_ALL/chrome/) 作為課堂所用，請在課堂開始前先行安裝完成

## 透過 Selenium 爬取蝦皮拍賣實戰
本課程延伸上台課的後段，透過 Selenium 爬取蝦皮購物網站
![](https://i.imgur.com/B7spJiO.png)

在爬蟲中，很重要的一個要素是網頁的翻頁動作，因此我們來觀察在蝦皮是如何翻頁的
![](https://i.imgur.com/cRjPjZD.png)

點擊第二頁
![](https://i.imgur.com/1nesDqF.png)

此時能發現網頁在第二頁時，網址中多了一個 `page=1`，繼續點擊第三頁觀察
![](https://i.imgur.com/eOFzKtE.png)

在第三頁中，可以發現網址中的 page 是 `page=2`，以此類推，那第一頁在蝦皮中怎麼呈現呢？點回第一頁看看
![](https://i.imgur.com/ADRAUYf.png)

可以看到，第一頁的網址為 `page=0`，有了這樣的了解之後，來寫程式吧！
```python=
import requests
from bs4 import BeautifulSoup
import time
import re
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium import webdriver

url = 'https://shopee.tw'

driver = webdriver.Chrome('~/chromedriver')
driver.get(url)
time.sleep(3)

#找到搜尋欄，並輸入 iPhone
search_bar = driver.find_element_by_class_name('shopee-searchbar-input__input')
search_bar.send_keys('iPhone')

search_bar.send_keys(Keys.ENTER)

res = driver.page_source
soup = BeautifulSoup(res, 'html.parser')

url = 'https://shopee.tw/search/?keyword=iphone&page={}&sortBy=relevancy'
page = 0

while True:
    driver = webdriver.Chrome('~/chromedriver')
    driver.get(url.format(page))
    time.sleep(3)
    
    res = driver.page_source
    soup = BeautifulSoup(res, 'html.parser')
    
    for i in soup.find_all(class_ = 'col-xs-2-4 shopee-search-item-result__item'):
        title = i.find(class_ = 'shopee-item-card__text-name').text
        price = i.find(class_ = 'shopee-item-card__current-price').text
        love = i.find(class_ = 'shopee-item-card__btn-like__text').text
        href = 'https://shopee.tw' + i.find(class_ = 'shopee-item-card--link').get('href')
        print([title, price, love, href], '第 ' + str(page + 1) + '頁')
    
    page += 1
    driver.close()
```
輸出：
```
['🍎保證原廠品質 iPhone充電線 Apple充電線 iPhone X 8 7 6 Plus iPad 現貨 傳輸線', '$230 - $699', '3781', 'https://shopee.tw/%F0%9F%8D%8E%E4%BF%9D%E8%AD%89%E5%8E%9F%E5%BB%A0%E5%93%81%E8%B3%AA-iPhone%E5%85%85%E9%9B%BB%E7%B7%9A-Apple%E5%85%85%E9%9B%BB%E7%B7%9A-iPhone-X-8-7-6-Plus-iPad-%E7%8F%BE%E8%B2%A8-%E5%82%B3%E8%BC%B8%E7%B7%9A-i.1400057.81064035'] 第 1頁
['iPhone 口袋相簿 隨身碟 手機隨身碟 8 7 6 OTG 蘋果 容量擴充', '$690 - $990', '0', 'https://shopee.tw/iPhone-%E5%8F%A3%E8%A2%8B%E7%9B%B8%E7%B0%BF-%E9%9A%A8%E8%BA%AB%E7%A2%9F-%E6%89%8B%E6%A9%9F%E9%9A%A8%E8%BA%AB%E7%A2%9F-8-7-6-OTG-%E8%98%8B%E6%9E%9C-%E5%AE%B9%E9%87%8F%E6%93%B4%E5%85%85-i.4300289.1345291022'] 第 1頁
['【iPhone】Mcdodo 100%智能斷電 快充線 傳輸線 充電線 線 iphone 呼吸線 數據線 iPhone線', '$199 - $259', '52', 'https://shopee.tw/%E3%80%90iPhone%E3%80%91Mcdodo-100-%E6%99%BA%E8%83%BD%E6%96%B7%E9%9B%BB-%E5%BF%AB%E5%85%85%E7%B7%9A-%E5%82%B3%E8%BC%B8%E7%B7%9A-%E5%85%85%E9%9B%BB%E7%B7%9A-%E7%B7%9A-iphone-%E5%91%BC%E5%90%B8%E7%B7%9A-%E6%95%B8%E6%93%9A%E7%B7%9A-iPhone%E7%B7%9A-i.10380287.1175000620'] 第 1頁
['台灣公司貨 雷射防偽標簽  RK3036 AnyCast 手機電視棒 hdmi av  MHL M5 plus HDMI', '$60 - $299', '770', 'https://shopee.tw/%E5%8F%B0%E7%81%A3%E5%85%AC%E5%8F%B8%E8%B2%A8-%E9%9B%B7%E5%B0%84%E9%98%B2%E5%81%BD%E6%A8%99%E7%B0%BD-RK3036-AnyCast-%E6%89%8B%E6%A9%9F%E9%9B%BB%E8%A6%96%E6%A3%92-hdmi-av-MHL-M5-plus-HDMI-i.43997244.694861124'] 第 1頁
['用不壞 新版磁吸充電線 三合一 充電線 蘋果/安卓/Type-c 充電線 iphone充電線 oppo', '$55 - $150', '153', 'https://shopee.tw/%E7%94%A8%E4%B8%8D%E5%A3%9E-%E6%96%B0%E7%89%88%E7%A3%81%E5%90%B8%E5%85%85%E9%9B%BB%E7%B7%9A-%E4%B8%89%E5%90%88%E4%B8%80-%E5%85%85%E9%9B%BB%E7%B7%9A-%E8%98%8B%E6%9E%9C-%E5%AE%89%E5%8D%93-Type-c-%E5%85%85%E9%9B%BB%E7%B7%9A-iphone%E5%85%85%E9%9B%BB%E7%B7%9A-oppo-i.3992961.923579518'] 第 1頁
['免卡分期_現金分期_Apple iPhone6s iPhone6sPlus 32G 128G 粉 灰 金 銀_空機分期', '$16,900', '34', 'https://shopee.tw/%E5%85%8D%E5%8D%A1%E5%88%86%E6%9C%9F_%E7%8F%BE%E9%87%91%E5%88%86%E6%9C%9F_Apple-iPhone6s-iPhone6sPlus-32G-128G-%E7%B2%89-%E7%81%B0-%E9%87%91-%E9%8A%80_%E7%A9%BA%E6%A9%9F%E5%88%86%E6%9C%9F-i.1776065.1008340068'] 第 1頁
['「缺貨中」全新未拆APPLE iPhone 6s Plus 32GB 128GB 金色 銀色 玫瑰金 灰色', '$20,500 - $25,500', '24', 'https://shopee.tw/%E3%80%8C%E7%BC%BA%E8%B2%A8%E4%B8%AD%E3%80%8D%E5%85%A8%E6%96%B0%E6%9C%AA%E6%8B%86APPLE-iPhone-6s-Plus-32GB-128GB-%E9%87%91%E8%89%B2-%E9%8A%80%E8%89%B2-%E7%8E%AB%E7%91%B0%E9%87%91-%E7%81%B0%E8%89%B2-i.3029869.469395444'] 第 1頁
['實體店面 原廠保固一年 iPhone6 iPhone 6s Plus 5.5 128G 金色 玫瑰金 銀色 太空灰', '$19,900', '82', 'https://shopee.tw/%E5%AF%A6%E9%AB%94%E5%BA%97%E9%9D%A2-%E5%8E%9F%E5%BB%A0%E4%BF%9D%E5%9B%BA%E4%B8%80%E5%B9%B4-iPhone6-iPhone-6s-Plus-5.5-128G-%E9%87%91%E8%89%B2-%E7%8E%AB%E7%91%B0%E9%87%91-%E9%8A%80%E8%89%B2-%E5%A4%AA%E7%A9%BA%E7%81%B0-i.10326803.629889710'] 第 1頁
['Apple iPhone 6s plus 64g 整新機 展示機 福利品 外觀全新無傷 保固三個月 臺中可面交', '$12,500', '173', 'https://shopee.tw/Apple-iPhone-6s-plus-64g-%E6%95%B4%E6%96%B0%E6%A9%9F-%E5%B1%95%E7%A4%BA%E6%A9%9F-%E7%A6%8F%E5%88%A9%E5%93%81-%E5%A4%96%E8%A7%80%E5%85%A8%E6%96%B0%E7%84%A1%E5%82%B7-%E4%BF%9D%E5%9B%BA%E4%B8%89%E5%80%8B%E6%9C%88-%E8%87%BA%E4%B8%AD%E5%8F%AF%E9%9D%A2%E4%BA%A4-i.3714830.392539309'] 第 1頁
['全新未拆 APPLE iPhone 6s Plus 128G 金銀白太空灰黑玫瑰金 5.5吋 公司貨保固1年 高雄可面交', '$22,500', '55', 'https://shopee.tw/%E5%85%A8%E6%96%B0%E6%9C%AA%E6%8B%86-APPLE-iPhone-6s-Plus-128G-%E9%87%91%E9%8A%80%E7%99%BD%E5%A4%AA%E7%A9%BA%E7%81%B0%E9%BB%91%E7%8E%AB%E7%91%B0%E9%87%91-5.5%E5%90%8B-%E5%85%AC%E5%8F%B8%E8%B2%A8%E4%BF%9D%E5%9B%BA1%E5%B9%B4-%E9%AB%98%E9%9B%84%E5%8F%AF%E9%9D%A2%E4%BA%A4-i.538545.318540804'] 第 1頁
['免卡分期_現金分期_Apple iPhone6s iPhone6sPlus 32G 128G 粉 灰 金 銀_空機分期', '$16,900', '0', 'https://shopee.tw/%E5%85%8D%E5%8D%A1%E5%88%86%E6%9C%9F_%E7%8F%BE%E9%87%91%E5%88%86%E6%9C%9F_Apple-iPhone6s-iPhone6sPlus-32G-128G-%E7%B2%89-%E7%81%B0-%E9%87%91-%E9%8A%80_%E7%A9%BA%E6%A9%9F%E5%88%86%E6%9C%9F-i.2818536.1298548558'] 第 1頁
['- AT. Select - 9成新 極新 現貨 空機 iPhone6S 4.7吋 64g 二手 太空灰 金色 玫瑰金', '$7,700 - $8,700', '170', 'https://shopee.tw/-AT.-Select-9%E6%88%90%E6%96%B0-%E6%A5%B5%E6%96%B0-%E7%8F%BE%E8%B2%A8-%E7%A9%BA%E6%A9%9F-iPhone6S-4.7%E5%90%8B-64g-%E4%BA%8C%E6%89%8B-%E5%A4%AA%E7%A9%BA%E7%81%B0-%E9%87%91%E8%89%B2-%E7%8E%AB%E7%91%B0%E9%87%91-i.33641094.1177201020'] 第 1頁
['限時特價空機iPhone6S 16G 64G 保固七天 無傷盒裝 功能正常 保固 另有多種規格iPhone 歡迎詢問', '$5,999 - $7,999', '1159', 'https://shopee.tw/%E9%99%90%E6%99%82%E7%89%B9%E5%83%B9%E7%A9%BA%E6%A9%9FiPhone6S-16G-64G-%E4%BF%9D%E5%9B%BA%E4%B8%83%E5%A4%A9-%E7%84%A1%E5%82%B7%E7%9B%92%E8%A3%9D-%E5%8A%9F%E8%83%BD%E6%AD%A3%E5%B8%B8-%E4%BF%9D%E5%9B%BA-%E5%8F%A6%E6%9C%89%E5%A4%9A%E7%A8%AE%E8%A6%8F%E6%A0%BCiPhone-%E6%AD%A1%E8%BF%8E%E8%A9%A2%E5%95%8F-i.7301478.156289561'] 第 1頁
['現貨iPhone6 4.7吋 64g 功能正常 無傷 銀色 灰色  土豪金 另有其他規格歡迎詢問 二手空機 保固七天', '$4,999 - $11,999', '963', 'https://shopee.tw/%E7%8F%BE%E8%B2%A8iPhone6-4.7%E5%90%8B-64g-%E5%8A%9F%E8%83%BD%E6%AD%A3%E5%B8%B8-%E7%84%A1%E5%82%B7-%E9%8A%80%E8%89%B2-%E7%81%B0%E8%89%B2-%E5%9C%9F%E8%B1%AA%E9%87%91-%E5%8F%A6%E6%9C%89%E5%85%B6%E4%BB%96%E8%A6%8F%E6%A0%BC%E6%AD%A1%E8%BF%8E%E8%A9%A2%E5%95%8F-%E4%BA%8C%E6%89%8B%E7%A9%BA%E6%A9%9F-%E4%BF%9D%E5%9B%BA%E4%B8%83%E5%A4%A9-i.7301478.174247004'] 第 1頁
['【實體店面 歡迎自取】Apple iphone6 16G /中古機/二手機/無傷/99%新/ 空機 /金、銀、太空灰', '$4,999 - $5,249', '480', 'https://shopee.tw/%E3%80%90%E5%AF%A6%E9%AB%94%E5%BA%97%E9%9D%A2-%E6%AD%A1%E8%BF%8E%E8%87%AA%E5%8F%96%E3%80%91Apple-iphone6-16G-%E4%B8%AD%E5%8F%A4%E6%A9%9F-%E4%BA%8C%E6%89%8B%E6%A9%9F-%E7%84%A1%E5%82%B7-99-%E6%96%B0-%E7%A9%BA%E6%A9%9F-%E9%87%91%E3%80%81%E9%8A%80%E3%80%81%E5%A4%AA%E7%A9%BA%E7%81%B0-i.69889264.1167988617'] 第 1頁
['iphone6 64G 銀 漂亮美機 台北/中壢/新竹/台中/台南 可面交', '$6,300', '62', 'https://shopee.tw/iphone6-64G-%E9%8A%80-%E6%BC%82%E4%BA%AE%E7%BE%8E%E6%A9%9F-%E5%8F%B0%E5%8C%97-%E4%B8%AD%E5%A3%A2-%E6%96%B0%E7%AB%B9-%E5%8F%B0%E4%B8%AD-%E5%8F%B0%E5%8D%97-%E5%8F%AF%E9%9D%A2%E4%BA%A4-i.21230502.1249701049'] 第 1頁
['二手美機 iPhone6s 4.7吋 64G 金色 無傷 全機包膜 功能正常 有配件 盒子 9成新', '$9,500', '52', 'https://shopee.tw/%E4%BA%8C%E6%89%8B%E7%BE%8E%E6%A9%9F-iPhone6s-4.7%E5%90%8B-64G-%E9%87%91%E8%89%B2-%E7%84%A1%E5%82%B7-%E5%85%A8%E6%A9%9F%E5%8C%85%E8%86%9C-%E5%8A%9F%E8%83%BD%E6%AD%A3%E5%B8%B8-%E6%9C%89%E9%85%8D%E4%BB%B6-%E7%9B%92%E5%AD%90-9%E6%88%90%E6%96%B0-i.2517316.927394684'] 第 1頁
['全新品未拆 Apple iPhone 6S 128G 4.7吋 四色可選 全新台灣公司貨 可搭各大電信門號新辦續約跳槽', '$19,900', '256', 'https://shopee.tw/%E5%85%A8%E6%96%B0%E5%93%81%E6%9C%AA%E6%8B%86-Apple-iPhone-6S-128G-4.7%E5%90%8B-%E5%9B%9B%E8%89%B2%E5%8F%AF%E9%81%B8-%E5%85%A8%E6%96%B0%E5%8F%B0%E7%81%A3%E5%85%AC%E5%8F%B8%E8%B2%A8-%E5%8F%AF%E6%90%AD%E5%90%84%E5%A4%A7%E9%9B%BB%E4%BF%A1%E9%96%80%E8%99%9F%E6%96%B0%E8%BE%A6%E7%BA%8C%E7%B4%84%E8%B7%B3%E6%A7%BD-i.4924219.36994763'] 第 1頁
['現場快速維修/IPHONE6S 64G 金/外觀良好 功能正常 /最高價收購中古手機/洽0900440054', '$9,800', '109', 'https://shopee.tw/%E7%8F%BE%E5%A0%B4%E5%BF%AB%E9%80%9F%E7%B6%AD%E4%BF%AE-IPHONE6S-64G-%E9%87%91-%E5%A4%96%E8%A7%80%E8%89%AF%E5%A5%BD-%E5%8A%9F%E8%83%BD%E6%AD%A3%E5%B8%B8-%E6%9C%80%E9%AB%98%E5%83%B9%E6%94%B6%E8%B3%BC%E4%B8%AD%E5%8F%A4%E6%89%8B%E6%A9%9F-%E6%B4%BD0900440054-i.17779681.809565402'] 第 1頁
['**最殺小舖**中古iphone5s 金 16g 二手apple 蘋果手機 功能外觀正常9成正常使用痕跡 美機 女用一手', '$4,000', '128', 'https://shopee.tw/**%E6%9C%80%E6%AE%BA%E5%B0%8F%E8%88%96**%E4%B8%AD%E5%8F%A4iphone5s-%E9%87%91-16g-%E4%BA%8C%E6%89%8Bapple-%E8%98%8B%E6%9E%9C%E6%89%8B%E6%A9%9F-%E5%8A%9F%E8%83%BD%E5%A4%96%E8%A7%80%E6%AD%A3%E5%B8%B89%E6%88%90%E6%AD%A3%E5%B8%B8%E4%BD%BF%E7%94%A8%E7%97%95%E8%B7%A1-%E7%BE%8E%E6%A9%9F-%E5%A5%B3%E7%94%A8%E4%B8%80%E6%89%8B-i.3512498.503501563'] 第 1頁
['iPhone6 16g 金色 太空灰 銀色 9.9成新 無傷美機 二手中古機（桃園 台北歡迎面交）', '$4,899 - $5,149', '197', 'https://shopee.tw/iPhone6-16g-%E9%87%91%E8%89%B2-%E5%A4%AA%E7%A9%BA%E7%81%B0-%E9%8A%80%E8%89%B2-9.9%E6%88%90%E6%96%B0-%E7%84%A1%E5%82%B7%E7%BE%8E%E6%A9%9F-%E4%BA%8C%E6%89%8B%E4%B8%AD%E5%8F%A4%E6%A9%9F%EF%BC%88%E6%A1%83%E5%9C%92-%E5%8F%B0%E5%8C%97%E6%AD%A1%E8%BF%8E%E9%9D%A2%E4%BA%A4%EF%BC%89-i.57728690.1216276968'] 第 1頁
['iphone6 64g 金色 銀色 太空灰 9.9成新 無傷美機 二手中古機（桃園 台北歡迎面交）', '$6,199 - $6,449', '171', 'https://shopee.tw/iphone6-64g-%E9%87%91%E8%89%B2-%E9%8A%80%E8%89%B2-%E5%A4%AA%E7%A9%BA%E7%81%B0-9.9%E6%88%90%E6%96%B0-%E7%84%A1%E5%82%B7%E7%BE%8E%E6%A9%9F-%E4%BA%8C%E6%89%8B%E4%B8%AD%E5%8F%A4%E6%A9%9F%EF%BC%88%E6%A1%83%E5%9C%92-%E5%8F%B0%E5%8C%97%E6%AD%A1%E8%BF%8E%E9%9D%A2%E4%BA%A4%EF%BC%89-i.57728690.1201573735'] 第 1頁
['二手Iphone6s 64g金色9成新 可面交可貨到付款 實體店面一定安心💯', '$8,500', '65', 'https://shopee.tw/%E4%BA%8C%E6%89%8BIphone6s-64g%E9%87%91%E8%89%B29%E6%88%90%E6%96%B0-%E5%8F%AF%E9%9D%A2%E4%BA%A4%E5%8F%AF%E8%B2%A8%E5%88%B0%E4%BB%98%E6%AC%BE-%E5%AF%A6%E9%AB%94%E5%BA%97%E9%9D%A2%E4%B8%80%E5%AE%9A%E5%AE%89%E5%BF%83%F0%9F%92%AF-i.4144645.810720965'] 第 1頁
['☆星創通訊☆二手 蘋果 APPLE IPHONE5s  64GB  4吋 太空灰 中古機 2手 空機 中古', '$5,900', '14', 'https://shopee.tw/%E2%98%86%E6%98%9F%E5%89%B5%E9%80%9A%E8%A8%8A%E2%98%86%E4%BA%8C%E6%89%8B-%E8%98%8B%E6%9E%9C-APPLE-IPHONE5s-64GB-4%E5%90%8B-%E5%A4%AA%E7%A9%BA%E7%81%B0-%E4%B8%AD%E5%8F%A4%E6%A9%9F-2%E6%89%8B-%E7%A9%BA%E6%A9%9F-%E4%B8%AD%E5%8F%A4-i.9334854.622872933'] 第 1頁
['95%新 展示 中古二手 福利機 Apple iPhone 6S 64 128G 金銀灰粉 空機 手機平板貼換 門號攜碼', '$11,999', '93', 'https://shopee.tw/95-%E6%96%B0-%E5%B1%95%E7%A4%BA-%E4%B8%AD%E5%8F%A4%E4%BA%8C%E6%89%8B-%E7%A6%8F%E5%88%A9%E6%A9%9F-Apple-iPhone-6S-64-128G-%E9%87%91%E9%8A%80%E7%81%B0%E7%B2%89-%E7%A9%BA%E6%A9%9F-%E6%89%8B%E6%A9%9F%E5%B9%B3%E6%9D%BF%E8%B2%BC%E6%8F%9B-%E9%96%80%E8%99%9F%E6%94%9C%E7%A2%BC-i.5256057.146072297'] 第 1頁
['iphone6 64G 金 漂亮美機 台北/中壢/新竹/台中/台南 可面交', '$6,300', '50', 'https://shopee.tw/iphone6-64G-%E9%87%91-%E6%BC%82%E4%BA%AE%E7%BE%8E%E6%A9%9F-%E5%8F%B0%E5%8C%97-%E4%B8%AD%E5%A3%A2-%E6%96%B0%E7%AB%B9-%E5%8F%B0%E4%B8%AD-%E5%8F%B0%E5%8D%97-%E5%8F%AF%E9%9D%A2%E4%BA%A4-i.21230502.1249705815'] 第 1頁
['IPhone6 32G 全新未拆封 高雄都可面交 高雄皆有實體店面哦↘️↘️10500', '$10,550 - $10,990', '17', 'https://shopee.tw/IPhone6-32G-%E5%85%A8%E6%96%B0%E6%9C%AA%E6%8B%86%E5%B0%81-%E9%AB%98%E9%9B%84%E9%83%BD%E5%8F%AF%E9%9D%A2%E4%BA%A4-%E9%AB%98%E9%9B%84%E7%9A%86%E6%9C%89%E5%AF%A6%E9%AB%94%E5%BA%97%E9%9D%A2%E5%93%A6%E2%86%98%EF%B8%8F%E2%86%98%EF%B8%8F10500-i.5350643.1141565518'] 第 1頁
['免卡分期_空機分期_Apple iPhone7 Plus 32G 128G iPhone 7 粉 霧黑 金 銀_現金分期', '$23,500', '10', 'https://shopee.tw/%E5%85%8D%E5%8D%A1%E5%88%86%E6%9C%9F_%E7%A9%BA%E6%A9%9F%E5%88%86%E6%9C%9F_Apple-iPhone7-Plus-32G-128G-iPhone-7-%E7%B2%89-%E9%9C%A7%E9%BB%91-%E9%87%91-%E9%8A%80_%E7%8F%BE%E9%87%91%E5%88%86%E6%9C%9F-i.67586166.1123799639'] 第 1頁
['【愛瘋人iM 蘋果專賣】iPhone 6s 128G 金 灰 銀 外觀漂亮無傷!! 交易百台 品質信賴', '$14,700', '30', 'https://shopee.tw/%E3%80%90%E6%84%9B%E7%98%8B%E4%BA%BAiM-%E8%98%8B%E6%9E%9C%E5%B0%88%E8%B3%A3%E3%80%91iPhone-6s-128G-%E9%87%91-%E7%81%B0-%E9%8A%80-%E5%A4%96%E8%A7%80%E6%BC%82%E4%BA%AE%E7%84%A1%E5%82%B7!!-%E4%BA%A4%E6%98%93%E7%99%BE%E5%8F%B0-%E5%93%81%E8%B3%AA%E4%BF%A1%E8%B3%B4-i.1311175.590637328'] 第 1頁
['免運費 iPhone 6s plus 64G(送鋼化膜+空壓殼) 16G/128G/5.5吋/1200萬畫素/4G上網', '$6,500 - $10,999', '775', 'https://shopee.tw/%E5%85%8D%E9%81%8B%E8%B2%BB-iPhone-6s-plus-64G(%E9%80%81%E9%8B%BC%E5%8C%96%E8%86%9C-%E7%A9%BA%E5%A3%93%E6%AE%BC)-16G-128G-5.5%E5%90%8B-1200%E8%90%AC%E7%95%AB%E7%B4%A0-4G%E4%B8%8A%E7%B6%B2-i.6776298.182275626'] 第 1頁
['**最殺小舖**女用中古一手iphone6s 16g  灰色 女用 一手 正常使用痕跡 另有i6 i6p i6sp', '$7,500', '64', 'https://shopee.tw/**%E6%9C%80%E6%AE%BA%E5%B0%8F%E8%88%96**%E5%A5%B3%E7%94%A8%E4%B8%AD%E5%8F%A4%E4%B8%80%E6%89%8Biphone6s-16g-%E7%81%B0%E8%89%B2-%E5%A5%B3%E7%94%A8-%E4%B8%80%E6%89%8B-%E6%AD%A3%E5%B8%B8%E4%BD%BF%E7%94%A8%E7%97%95%E8%B7%A1-%E5%8F%A6%E6%9C%89i6-i6p-i6sp-i.3512498.698565033'] 第 1頁
['嚴選  Iphone6 64g 金/銀/ 灰僅售5800東東通訊 新竹二手機買賣', '$5,800', '508', 'https://shopee.tw/%E5%9A%B4%E9%81%B8-Iphone6-64g-%E9%87%91-%E9%8A%80-%E7%81%B0%E5%83%85%E5%94%AE5800%E6%9D%B1%E6%9D%B1%E9%80%9A%E8%A8%8A-%E6%96%B0%E7%AB%B9%E4%BA%8C%E6%89%8B%E6%A9%9F%E8%B2%B7%E8%B3%A3-i.22462176.520601556'] 第 1頁
['嚴選 Iphone6 128 金/灰/銀 僅售7800 東東通訊 新竹二手機買賣', '$7,800', '120', 'https://shopee.tw/%E5%9A%B4%E9%81%B8-Iphone6-128-%E9%87%91-%E7%81%B0-%E9%8A%80-%E5%83%85%E5%94%AE7800-%E6%9D%B1%E6%9D%B1%E9%80%9A%E8%A8%8A-%E6%96%B0%E7%AB%B9%E4%BA%8C%E6%89%8B%E6%A9%9F%E8%B2%B7%E8%B3%A3-i.22462176.418995091'] 第 1頁
['中古iphone6 64G銀 金外觀九成新功能正常 無維修紀錄 另有6 plus 6s iphone7', '$6,500', '80', 'https://shopee.tw/%E4%B8%AD%E5%8F%A4iphone6-64G%E9%8A%80-%E9%87%91%E5%A4%96%E8%A7%80%E4%B9%9D%E6%88%90%E6%96%B0%E5%8A%9F%E8%83%BD%E6%AD%A3%E5%B8%B8-%E7%84%A1%E7%B6%AD%E4%BF%AE%E7%B4%80%E9%8C%84-%E5%8F%A6%E6%9C%896-plus-6s-iphone7-i.9839728.1135754411'] 第 1頁
['售9.9新 iPhone 6s plus128g玫瑰金 配件在 功能正常 外觀無傷', '$12,500', '271', 'https://shopee.tw/%E5%94%AE9.9%E6%96%B0-iPhone-6s-plus128g%E7%8E%AB%E7%91%B0%E9%87%91-%E9%85%8D%E4%BB%B6%E5%9C%A8-%E5%8A%9F%E8%83%BD%E6%AD%A3%E5%B8%B8-%E5%A4%96%E8%A7%80%E7%84%A1%E5%82%B7-i.10721174.1113662268'] 第 1頁
['一手女用美機 iphone6 太空灰 64g 9.5成新 機況良好 用能正常', '$12,900', '183', 'https://shopee.tw/%E4%B8%80%E6%89%8B%E5%A5%B3%E7%94%A8%E7%BE%8E%E6%A9%9F-iphone6-%E5%A4%AA%E7%A9%BA%E7%81%B0-64g-9.5%E6%88%90%E6%96%B0-%E6%A9%9F%E6%B3%81%E8%89%AF%E5%A5%BD-%E7%94%A8%E8%83%BD%E6%AD%A3%E5%B8%B8-i.1431058.163107983'] 第 1頁
['免卡分期_現金分期_Apple iPhone 7 iPhone7 PLUS iP7 32G 128G 256G_空機分期', '$20,900', '11', 'https://shopee.tw/%E5%85%8D%E5%8D%A1%E5%88%86%E6%9C%9F_%E7%8F%BE%E9%87%91%E5%88%86%E6%9C%9F_Apple-iPhone-7-iPhone7-PLUS-iP7-32G-128G-256G_%E7%A9%BA%E6%A9%9F%E5%88%86%E6%9C%9F-i.1776065.466603138'] 第 1頁
['免卡分期_現金分期_Apple iPhone7 iPhone7Plus 32G 128G _全省線上申請_板橋實體門市', '$21,900', '8', 'https://shopee.tw/%E5%85%8D%E5%8D%A1%E5%88%86%E6%9C%9F_%E7%8F%BE%E9%87%91%E5%88%86%E6%9C%9F_Apple-iPhone7-iPhone7Plus-32G-128G-_%E5%85%A8%E7%9C%81%E7%B7%9A%E4%B8%8A%E7%94%B3%E8%AB%8B_%E6%9D%BF%E6%A9%8B%E5%AF%A6%E9%AB%94%E9%96%80%E5%B8%82-i.2038308.1091117238'] 第 1頁
['免卡分期_現金分期_空機分期 Apple iPhone8 紅 iPhone8 Plus 64G 256G 黑 金 銀', '$24,500', '61', 'https://shopee.tw/%E5%85%8D%E5%8D%A1%E5%88%86%E6%9C%9F_%E7%8F%BE%E9%87%91%E5%88%86%E6%9C%9F_%E7%A9%BA%E6%A9%9F%E5%88%86%E6%9C%9F-Apple-iPhone8-%E7%B4%85-iPhone8-Plus-64G-256G-%E9%BB%91-%E9%87%91-%E9%8A%80-i.2038308.1084392761'] 第 1頁
['Iphone6S 64G 金 粉 銀 灰 蘆洲 三重 西門町都可面交 蘆洲有實體店面 歡迎詢問', '$8,600', '155', 'https://shopee.tw/Iphone6S-64G-%E9%87%91-%E7%B2%89-%E9%8A%80-%E7%81%B0-%E8%98%86%E6%B4%B2-%E4%B8%89%E9%87%8D-%E8%A5%BF%E9%96%80%E7%94%BA%E9%83%BD%E5%8F%AF%E9%9D%A2%E4%BA%A4-%E8%98%86%E6%B4%B2%E6%9C%89%E5%AF%A6%E9%AB%94%E5%BA%97%E9%9D%A2-%E6%AD%A1%E8%BF%8E%E8%A9%A2%E5%95%8F-i.7204573.496391934'] 第 1頁
['蘋果公司貨APPLE IPHONE6 64G 4g金4.7吋 95%新 外觀超新功能正常盒裝配件期', '$6,599', '100', 'https://shopee.tw/%E8%98%8B%E6%9E%9C%E5%85%AC%E5%8F%B8%E8%B2%A8APPLE-IPHONE6-64G-4g%E9%87%914.7%E5%90%8B-95-%E6%96%B0-%E5%A4%96%E8%A7%80%E8%B6%85%E6%96%B0%E5%8A%9F%E8%83%BD%E6%AD%A3%E5%B8%B8%E7%9B%92%E8%A3%9D%E9%85%8D%E4%BB%B6%E6%9C%9F-i.53975464.1089138258'] 第 1頁
['自售 iphone6S 128g 土豪金 9成5新 系統 ios10.3.3 全機包膜 全機無傷', '$11,100', '33', 'https://shopee.tw/%E8%87%AA%E5%94%AE-iphone6S-128g-%E5%9C%9F%E8%B1%AA%E9%87%91-9%E6%88%905%E6%96%B0-%E7%B3%BB%E7%B5%B1-ios10.3.3-%E5%85%A8%E6%A9%9F%E5%8C%85%E8%86%9C-%E5%85%A8%E6%A9%9F%E7%84%A1%E5%82%B7-i.3479137.992235154'] 第 1頁
['新機 iPhone 6s 32G 空機 4.7吋IOS蘋果 搭配門號遠傳電信999 吃到飽超優惠 內洽高雄【承靜數位】', '$3,000', '93', 'https://shopee.tw/%E6%96%B0%E6%A9%9F-iPhone-6s-32G-%E7%A9%BA%E6%A9%9F-4.7%E5%90%8BIOS%E8%98%8B%E6%9E%9C-%E6%90%AD%E9%85%8D%E9%96%80%E8%99%9F%E9%81%A0%E5%82%B3%E9%9B%BB%E4%BF%A1999-%E5%90%83%E5%88%B0%E9%A3%BD%E8%B6%85%E5%84%AA%E6%83%A0-%E5%85%A7%E6%B4%BD%E9%AB%98%E9%9B%84%E3%80%90%E6%89%BF%E9%9D%9C%E6%95%B8%E4%BD%8D%E3%80%91-i.9877694.786939348'] 第 1頁
['藍星通訊 各家媒體一致推薦 IPHONE6 7  8 PLUS 128G 64G 32G 中古機 分期專案價', '$6,500', '115', 'https://shopee.tw/%E8%97%8D%E6%98%9F%E9%80%9A%E8%A8%8A-%E5%90%84%E5%AE%B6%E5%AA%92%E9%AB%94%E4%B8%80%E8%87%B4%E6%8E%A8%E8%96%A6-IPHONE6-7-8-PLUS-128G-64G-32G-%E4%B8%AD%E5%8F%A4%E6%A9%9F-%E5%88%86%E6%9C%9F%E5%B0%88%E6%A1%88%E5%83%B9-i.17071508.216925649'] 第 1頁
['iphone6 金 64g 16G 32 G 成新無傷 機況良好 用能正常', '$5,500 - $6,000', '168', 'https://shopee.tw/iphone6-%E9%87%91-64g-16G-32-G-%E6%88%90%E6%96%B0%E7%84%A1%E5%82%B7-%E6%A9%9F%E6%B3%81%E8%89%AF%E5%A5%BD-%E7%94%A8%E8%83%BD%E6%AD%A3%E5%B8%B8-i.1710456.621168200'] 第 1頁
['現貨【最高🍎品質】傳輸線 充電線 線 Apple線 iphone充電線 數據線 iPhone線 ipad', '$110 - $699', '326', 'https://shopee.tw/%E7%8F%BE%E8%B2%A8%E3%80%90%E6%9C%80%E9%AB%98%F0%9F%8D%8E%E5%93%81%E8%B3%AA%E3%80%91%E5%82%B3%E8%BC%B8%E7%B7%9A-%E5%85%85%E9%9B%BB%E7%B7%9A-%E7%B7%9A-Apple%E7%B7%9A-iphone%E5%85%85%E9%9B%BB%E7%B7%9A-%E6%95%B8%E6%93%9A%E7%B7%9A-iPhone%E7%B7%9A-ipad-i.10380287.943939150'] 第 1頁
['限時特價!超值批發價! 蘋果音源線 蘋果充電線 通話/充電 一分二音頻線  二合一 iPhone 蘋果 耳機', '$149', '66', 'https://shopee.tw/%E9%99%90%E6%99%82%E7%89%B9%E5%83%B9!%E8%B6%85%E5%80%BC%E6%89%B9%E7%99%BC%E5%83%B9!-%E8%98%8B%E6%9E%9C%E9%9F%B3%E6%BA%90%E7%B7%9A-%E8%98%8B%E6%9E%9C%E5%85%85%E9%9B%BB%E7%B7%9A-%E9%80%9A%E8%A9%B1-%E5%85%85%E9%9B%BB-%E4%B8%80%E5%88%86%E4%BA%8C%E9%9F%B3%E9%A0%BB%E7%B7%9A-%E4%BA%8C%E5%90%88%E4%B8%80-iPhone-%E8%98%8B%E6%9E%9C-%E8%80%B3%E6%A9%9F-i.4827179.1184362381'] 第 1頁
['iPhoneX康寧5D玻璃貼i8滿版i7玻璃保護貼iPhone8Plus iPhone7 iPhone8頂級i6防塵iX', '$185 - $200', '764', 'https://shopee.tw/iPhoneX%E5%BA%B7%E5%AF%A75D%E7%8E%BB%E7%92%83%E8%B2%BCi8%E6%BB%BF%E7%89%88i7%E7%8E%BB%E7%92%83%E4%BF%9D%E8%AD%B7%E8%B2%BCiPhone8Plus-iPhone7-iPhone8%E9%A0%82%E7%B4%9Ai6%E9%98%B2%E5%A1%B5iX-i.59150433.960268844'] 第 1頁
['🍎保證原廠品質 iPhone充電線 Apple充電線 傳輸線 iPhone X 8 7 6 Plus iPad 充電器', '$225 - $390', '125', 'https://shopee.tw/%F0%9F%8D%8E%E4%BF%9D%E8%AD%89%E5%8E%9F%E5%BB%A0%E5%93%81%E8%B3%AA-iPhone%E5%85%85%E9%9B%BB%E7%B7%9A-Apple%E5%85%85%E9%9B%BB%E7%B7%9A-%E5%82%B3%E8%BC%B8%E7%B7%9A-iPhone-X-8-7-6-Plus-iPad-%E5%85%85%E9%9B%BB%E5%99%A8-i.6755406.1026875730'] 第 1頁
['現貨母親節禮物蘋果手機殼I8/I8+ iPhone i7系列I5SEiphone6手機殼背夾背殼式行動電源手機殼無缐充電', '$299', '752', 'https://shopee.tw/%E7%8F%BE%E8%B2%A8%E6%AF%8D%E8%A6%AA%E7%AF%80%E7%A6%AE%E7%89%A9%E8%98%8B%E6%9E%9C%E6%89%8B%E6%A9%9F%E6%AE%BCI8-I8-iPhone-i7%E7%B3%BB%E5%88%97I5SEiphone6%E6%89%8B%E6%A9%9F%E6%AE%BC%E8%83%8C%E5%A4%BE%E8%83%8C%E6%AE%BC%E5%BC%8F%E8%A1%8C%E5%8B%95%E9%9B%BB%E6%BA%90%E6%89%8B%E6%A9%9F%E6%AE%BC%E7%84%A1%E7%BC%90%E5%85%85%E9%9B%BB-i.40269829.738795364'] 第 1頁
['🍎保證原廠品質 iPhone充電線 Apple充電線 iPhone X 8 7 6 Plus iPad 現貨 傳輸線', '$320 - $699', '171', 'https://shopee.tw/%F0%9F%8D%8E%E4%BF%9D%E8%AD%89%E5%8E%9F%E5%BB%A0%E5%93%81%E8%B3%AA-iPhone%E5%85%85%E9%9B%BB%E7%B7%9A-Apple%E5%85%85%E9%9B%BB%E7%B7%9A-iPhone-X-8-7-6-Plus-iPad-%E7%8F%BE%E8%B2%A8-%E5%82%B3%E8%BC%B8%E7%B7%9A-i.1400057.114973793'] 第 2頁
['🍎保證原廠品質 iPhone X 8 7 6 5 plus充電線 傳輸線保固一年 iPhone 充電頭 1A頭豆腐頭', '$109 - $699', '11', 'https://shopee.tw/%F0%9F%8D%8E%E4%BF%9D%E8%AD%89%E5%8E%9F%E5%BB%A0%E5%93%81%E8%B3%AA-iPhone-X-8-7-6-5-plus%E5%85%85%E9%9B%BB%E7%B7%9A-%E5%82%B3%E8%BC%B8%E7%B7%9A%E4%BF%9D%E5%9B%BA%E4%B8%80%E5%B9%B4-iPhone-%E5%85%85%E9%9B%BB%E9%A0%AD-1A%E9%A0%AD%E8%B1%86%E8%85%90%E9%A0%AD-i.75949675.1274334367'] 第 2頁
...
```

這樣一來就大功告成啦，這堂課到這邊就結束囉，我們 [Python 網路爬蟲課程 10](/e_dsgueUQm-wWVjc5SZrww) 見 :eyes:

---
:::success
第九堂課結束囉:100:
這次又有作業了：請各位撰寫程式爬取五頁，接著轉換別項商品，共爬三項商品
:::

###### tags: `Python` `Python 網路爬蟲課程` `Crawler` `網路爬蟲` `SQLite`