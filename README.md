### 完整程式碼  
```python
import requests, random, csv              # 匯入必要模組
from bs4 import BeautifulSoup             # 匯入 HTML 解析工具

def get_comic_info():
    while True:  # while True 是為了「保證拿到一筆有效的漫畫資料」，不管遇到什麼失敗都會重試，直到成功為止。
        n = random.randint(1, 3086)  # 隨機選一篇漫畫（1 <= n <= 3086）
        url = f'https://xkcd.com/{n}/'  # 組成漫畫網址

        r = requests.get(url)  # 發送 GET 請求

        if r.status_code == 200:  # 檢查回應狀態碼是否為 200（代表請求成功）
            # 請求成功，處理內容
            soup = BeautifulSoup(r.text, 'html.parser')  # 解析 HTML
            title = soup.find('div', id='ctitle')  # 找標題
        else:
            get_comic_info()  # 若請求不成功，則重新呼叫

        if not title:  # 如果找不到標題，表示頁面異常
            continue   # 跳過這次，重試

        return {  # 成功擷取資料後，回傳字典格式
            'Comic Number': n,              # 漫畫編號
            'Title': title.text.strip(),    # 標題文字（去除多餘空白）
            'Comic URL': url                # 網頁網址
        }
def save_to_csv(data, f='xkcd_comics.csv'):
    # 將資料儲存到 CSV 檔案中（append，避免覆蓋原資料）
    with open(f, "a", newline='', encoding='utf-8') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=data.keys())  # 建立 CSV 寫入並寫入，欄位為資料的 key
        writer.writerow(data)  # 寫入一筆漫畫資料

# 主程式執行流程
comic = get_comic_info()  # 取得一筆隨機漫畫資訊
save_to_csv(comic)        # 將該筆資料寫入 CSV 檔案
```
### 程式碼製作步驟  
* 先把需要做到的事情分成幾個 pieces，最後再 combine。
* 我們需要隨機產生數字、寫入 csv 檔案、爬蟲
* 有學過的觀念先寫出來，爬蟲最後再參考網路資料撰寫
* 思考需要什麼->回顧這學期內容->有哪些內容是我們在報告中需要用到的先整理出來->little piece of code->flow chart、logic->combine code->debug  
1. 我們需要用到的模組有(需要 import):  

    | 需要的功能 | import |
    | --------- | ------ |
    | 隨機產生數字 | random |
    | 匯入 csv | csv |
    | 發送 HTTP 請求，抓取網頁資料 | reqests |
    | 解析網頁 | BeautifulSoup |
   ```python
   import requests, random, csv              # 匯入必要模組
   from bs4 import BeautifulSoup             # 取得網頁時，得到的只是原始 HTML 文字。BeautifulSoup 可以看懂 HTML 結構，讓我們可以很容易地找到想要的部分。
   ```
2. 定義一個函示可以隨機產生漫畫編號並產出網址  
   * 定義 comic 為這個函示呼叫出的資訊
    ```python
    def get_comic_info():  
        n = random.randint(1, 3086)  # 隨機選一篇漫畫（1 <= n <= 3086）
        url = f'https://xkcd.com/{n}/'  # 組成漫畫網址，並定義為 url
    comic = get_comic_info() #呼叫 get_comic_info() 這個函式
    ```
3. 希望寫入在 csv 的資料有漫畫編號、標題文字、網頁網址  
   * 這些資料要用字典儲存，有 key 與 value
     ```python
     return {  # 成功擷取資料後，回傳字典格式
     'Comic Number': n,              # 漫畫編號
     'Title': title.text.strip(),    # 標題文字（去除多餘空白）
     'Comic URL': url                # 網頁網址
     }
     ```
4. 定義一個函示可以將資料寫入 CSV 檔案
   * 希望以字典的格式儲存，因此使用 csv.DictWriter
   * 將步驟 2 的 comic 資訊寫入 csv
     ```python
     def save_to_csv(data, f='xkcd_comics.csv'):
         # 將資料儲存到 CSV 檔案中（append，避免覆蓋原資料）
         with open(f, "a", newline='', encoding='utf-8') as csvfile:
             writer = csv.DictWriter(csvfile, fieldnames=data.keys())  # 建立 CSV 並寫入，欄位為資料的 key
             writer.writerow(data)  # 寫入一筆漫畫資料
     
     comic = get_comic_info()  # 取得一筆隨機漫畫資訊
     save_to_csv(comic)        # 將該筆資料寫入 CSV 檔案
     ```
5. 爬蟲階段製作: 先對網頁發送請求，檢查是否有回應  
    * 如果有回應，繼續執行
    * 沒有回應，重新呼叫重新呼叫 get_comic_info() 函式
      ```python
      r = requests.get(url)  # 發送 GET 請求
      if r.status_code == 200:  # 檢查回應狀態碼是否為 200（代表請求成功）
      else:
          get_comic_info()  # 若請求不成功，則重新呼叫
      ```
6. 爬蟲階段製作: 放在步驟 5 的 if 條件式裡(如果 if 成立，則解析 HTML)
   ```python
   soup = BeautifulSoup(r.text, 'html.parser')  # 解析 HTML
   title = soup.find('div', id='ctitle')  # 找標題
   ```
7. 爬蟲階段製作: 確認步驟 6 是否找得到標題
    * 找不到: 重試
    * 找得到: 回傳步驟 3 的字典格式
     ```python
     if not title:  # 如果找不到標題，表示頁面異常
     continue   # 跳過這次，重試
     ```
8. 合併程式碼: 程式碼第 18 行報錯，需進行 debug
   ```python
    import requests, random, csv              # 匯入必要模組
    from bs4 import BeautifulSoup             # 匯入 HTML 解析工具
    
    def get_comic_info():
            n = random.randint(1, 3086)  # 隨機選一篇漫畫（1 <= n <= 3086）
            url = f'https://xkcd.com/{n}/'  # 組成漫畫網址
    
            r = requests.get(url)  # 發送 GET 請求
    
            if r.status_code == 200:  # 檢查回應狀態碼是否為 200（代表請求成功）
                # 請求成功，處理內容
                soup = BeautifulSoup(r.text, 'html.parser')  # 解析 HTML
                title = soup.find('div', id='ctitle')  # 找標題
            else:
                get_comic_info()  # 若請求不成功，則重新呼叫
    
            if not title:  # 如果找不到標題，表示頁面異常
                continue   # 跳過這次，重試
    
            return {  # 成功擷取資料後，回傳字典格式
                'Comic Number': n,              # 漫畫編號
                'Title': title.text.strip(),    # 標題文字（去除多餘空白）
                'Comic URL': url                # 網頁網址
            }
    
    def save_to_csv(data, f='xkcd_comics.csv'):
        # 將資料儲存到 CSV 檔案中（append，避免覆蓋原資料）
        with open(f, "a", newline='', encoding='utf-8') as csvfile:
            writer = csv.DictWriter(csvfile, fieldnames=data.keys())  # 建立 CSV 寫入並寫入，欄位為資料的 key
            writer.writerow(data)  # 寫入一筆漫畫資料
    
    # 主程式執行流程
    comic = get_comic_info()  # 取得一筆隨機漫畫資訊
    save_to_csv(comic)        # 將該筆資料寫入 CSV 檔案
   ```
   * 跑出的報錯內容  
   ```python
      Cell In[11],   line 18
        continue   # 跳過這次，重試
        ^
    SyntaxError: 'continue' not properly in loop
   ```
9. 完整程式碼: 經 google 後發現，continue 須放在迴圈裡，像是 while 或 for，因此選擇用 while True， 當條件成立時繼續往下執行。
    ```python
      import requests, random, csv              # 匯入必要模組
      from bs4 import BeautifulSoup             # 匯入 HTML 解析工具
      
      def get_comic_info():
          while True:  # while True 是為了「保證拿到一筆有效的漫畫資料」，不管遇到什麼失敗都會重試，直到成功為止。
              n = random.randint(1, 3086)  # 隨機選一篇漫畫（1 <= n <= 3086）
              url = f'https://xkcd.com/{n}/'  # 組成漫畫網址
      
              r = requests.get(url)  # 發送 GET 請求
      
              if r.status_code == 200:  # 檢查回應狀態碼是否為 200（代表請求成功）
                  # 請求成功，處理內容
                  soup = BeautifulSoup(r.text, 'html.parser')  # 解析 HTML
                  title = soup.find('div', id='ctitle')  # 找標題
              else:
                  get_comic_info()  # 若請求不成功，則重新呼叫
      
              if not title:  # 如果找不到標題，表示頁面異常
                  continue   # 跳過這次，重試
      
              return {  # 成功擷取資料後，回傳字典格式
                  'Comic Number': n,              # 漫畫編號
                  'Title': title.text.strip(),    # 標題文字（去除多餘空白）
                  'Comic URL': url                # 網頁網址
              }
      
      def save_to_csv(data, f='xkcd_comics.csv'):
          # 將資料儲存到 CSV 檔案中（append，避免覆蓋原資料）
          with open(f, "a", newline='', encoding='utf-8') as csvfile:
              writer = csv.DictWriter(csvfile, fieldnames=data.keys())  # 建立 CSV 寫入並寫入，欄位為資料的 key
              writer.writerow(data)  # 寫入一筆漫畫資料
      
      # 主程式執行流程
      comic = get_comic_info()  # 取得一筆隨機漫畫資訊
      save_to_csv(comic)        # 將該筆資料寫入 CSV 檔案
    ```
