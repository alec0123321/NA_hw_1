# 交大 NA 爬蟲 登入 破解驗證 取得課表
> [TOC]
## 需要的套件
1. import requests
2. import getpass
3. import matplotlib.pyplot as plt
4. from PIL import Image
5. import pytesseract
6. from bs4 import BeautifulSoup
7. import prettytable as pt

### requests
>Post及Get request
### getpass
>密碼輸入隱藏的動作
### BeautifulSoup
>抓取DOM元素取得資料
### PIL
> 灰階驗證碼的圖片使得方便辨識
### pytesseract
> Google的套件 將圖片轉換文字的工具
### matplotlib
> 處理驗證碼圖片
### prettytable
> 最後輸出table的時候將資料整合成表格


``` python
# coding=utf-8
# -*- coding:utf-8 -*-
import requests
import getpass

import matplotlib.pyplot as plt
from PIL import Image
import pytesseract
from bs4 import BeautifulSoup
import prettytable as pt

def convert_image(img, standard=127.5):
    image = img.convert('L')  # Convet RGB to grayscale

    # According to the threshold covert each pixel to value 0 or 255
    pixels = image.load()
    for x in range(image.width):
        for y in range(image.height):
            if pixels[x, y] > standard:
                pixels[x, y] = 255
            else:
                pixels[x, y] = 0
    return image

""" argument parser """


""" web and user information"""
login_url = "https://portal.nctu.edu.tw/portal/chkpas.php"
code_url = "https://portal.nctu.edu.tw/captcha/pic.php"
username = input("student_ID: ")
password = getpass.getpass("password: ")

#num = 0
while True:
    while True:
        s = requests.session()  # Create a new request session
        res = s.get(code_url)  # Request security code figure

        open("secode.png", 'wb').write(res.content)
        img = Image.open("secode.png")
        img = convert_image(img)
        secode = pytesseract.image_to_string(img)
        #num += 1
        if len(secode) is 4:
            if not secode.isdigit():
                continue
            #print(str(num) + " -> " + secode)
            break

    info = {
        "username": username,
        "Submit2": "%E7%99%BB%E5%85%A5%28Login%29",
        "pwdtype": "static",
        "password": password,
        "seccode": secode
    }

    res = s.post(login_url, data=info)

    soup = BeautifulSoup(res.content, 'html.parser')

    try:
        login_status = soup.find('title').string == "校園資訊系統"
        if login_status:
            break
    except:
        continue

res = s.get("https://portal.nctu.edu.tw/portal/relay.php?D=cos")
soup = BeautifulSoup(res.content, 'html.parser')

info2_index = [
    "txtId",
    "txtPw",
    "ldapDN",
    "idno",
    "s",
    "t",
    "txtTimestamp",
    "hashKey",
    "jwt",
    "Chk_SSO",
    "Button1"
]

info2 = {}
for index, text in enumerate(soup.find_all("input")):
    init = str(text).find("value")
    end = str(text).find("/")
    value = str(text)[init+7:end-1]
    info2[info2_index[index]] = value
    value
info2["Chk_SSO"] = "on"

res = s.post("https://course.nctu.edu.tw/jwt.asp", data=info2)
res = s.get("https://course.nctu.edu.tw/adSchedule.asp")
soup = BeautifulSoup(res.content, 'html.parser')


table = pt.PrettyTable()

day_info = []
row1 = soup.find_all("font")[0:9]
for content in row1:
    text = content.text.replace(' ', '')
    day_info.append(text)
table.field_names = day_info

rows = soup.find_all("font", size="2")

schedule = ["M", "N", "A", "B", "C", "D", "X", "E", "F", "G", "H", "Y", "I", "J", "K", "L"]

row_content = []
for index, content in enumerate(rows):
    text = content.text.replace(' ', '')
    text = text.replace('\n', '')
    text = text.replace('\r', '')
    text = text.replace('\t', '')
    text = text.replace('\xa0', '')

    if index == 0:
        row_content.append(schedule[int(index/8)])
        row_content.append(text)
    elif index%8 == 0:
        table.add_row(row_content)
        row_content = []
        row_content.append(schedule[int(index/8)])
        row_content.append(text)
    else:
        row_content.append(text)
table.add_row(row_content)

print(table)

```
### 密碼輸入執行時取得帳號密碼
![](https://i.imgur.com/wqYBgHh.png)
### 課表呈現
![](https://i.imgur.com/CPvLBgk.png)



---
 ### **使用環境**
 > macOS High Sierra 10.13.3
### !可能需要注意的地方
> 在mac上安裝matplotlib可能需要額外修改設定檔及安裝其他附加檔案才能運作
