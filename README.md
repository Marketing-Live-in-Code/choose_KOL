# 想找意見領袖（KOL）來品牌行銷嗎？超直觀挑選 —【 Python實戰篇】

有了前篇文章[「想找意見領袖（KOL）來品牌行銷嗎？超直觀挑選 — 【概念篇】」]()的概念後，我們可以換從數據的角度來挑選適合的KOL。現在有數千個KOL活躍在[Youtube影音平台](https://www.youtube.com/)，該如何做選擇呢？在[Influencer](https://tw.noxinfluencer.com/)網站中，幫我們整理了每個KOL的類別、粉絲數量、平均觀看數量、Nox評分。
![台灣Youtuber排行](https://i.imgur.com/sGjdVED.png)

雖然有這些資料，但還是不知道如何選擇KOL嗎？
![意見領袖分析成果圖](https://i.imgur.com/BQEwXOA.png)
意見領袖分析圖的X軸為「平均觀看次數」，Y軸則為「訂閱數量」，圓圈大小為「Nox評分」，虛線為平均值，資料一樣來自於[Influencer](https://tw.noxinfluencer.com/)幫我們整理好的資料。這樣呈現資料有什麼意義呢？

## 管理意涵
原因在於，KOL的合作金額非常浮動，而大多都與「訂閱數量」成正比，但若單看「訂閱數量」，會發現褐色的原點（第二象限）訂閱數非常高，但平均觀看次數卻不盡理想，若真的要與該KOL合作，豈不是「雷聲大雨點小」。

相對的綠色的圓點（第四象限）正好相反，雖然訂閱數不如其他KOL，但每部影片都有不錯的曝光，KOL的粉絲黏濁度很高。當然，若您的品牌預算不低，還是可以選擇紅色的圓點（第一象限），能有較高的成功行銷機率。

## 程式實作
![程式實作](https://i.imgur.com/dPwO7ko.png)

### 網路爬蟲
爬蟲的重點在於「資料從哪來？」，第一個最直觀的方式，就是直接在目的網頁中按下F12，也的確在Network — > Doc裡面發現一個封包，有整個網頁的程式碼，當然包含我們想要的資料。
![擷取封包畫面](https://i.imgur.com/ZdCt7Y1.png)

但當我們往下滾，會發現只有50筆資料，而網站會再載入另外50筆資料，由此可知，若我們直接用get請求的方式抓取資料，就只能抓取top50，如何能抓到更多資料呢？

往下滾動會發現，在分類XHR中多了一個新的封包，查看內容會發現，是第51～100筆的資料，而且除了這些資料，沒有其他雜訊。
![找到封包內容](https://i.imgur.com/6SpHng2.png)

確定我們要的封包後，就可以切換到Headers複製該網址。詳細的解讀這個網址可以發現，網址最後方的參數pageNum可能就是，決定「第幾頁的資料」。
```
https://tw.noxinfluencer.com/youtube-channel-rank/_influencer-rank?country=tw&category=all&rankSize=100&type=0&interval=weekly&pageNum=2
```
![封包網址](https://i.imgur.com/rQOLsLU.png)

接下來利用bs4套件請求，依照每個資料的class屬性，即可抓下各個youtuber的資料。若您不知道為什麼要用「kol-name」、「category-text」、「followerNum」…，您可以參考「Python爬下PTT文章內容技巧(含程式碼)」，裏頭有最詳盡易懂的解說。
```python
請求網站
 list_req = requests.get(url)
 將整個網站的程式碼爬下來
 soup = BeautifulSoup(list_req.content, "html.parser")
 找到標籤
 getAllName = soup.findAll('span',{'class','kol-name'})
 getAllClass = soup.findAll('a',{'class','category-text'})
 getAllFans = soup.findAll('td',{'class','followerNum'})
 getAllLooked = soup.findAll('td',{'class','avgView'})
 getAllStar = soup.findAll('td',{'class','nox-score'})
```
![資料抓取結果](https://i.imgur.com/F3K3Lye.png)

### 資料整理
主要是將每個資料分門別類地放到不同的陣列，方便等等轉乘DataFrame的格式。其中要注意，在資料中有許多的空白，因此必須使用replace方法，將空白給去除。
```python
#開始整理資料
theName = []
theClass = []
theFans = []
theLooked = []
theStar = []
for i in range(50):#每一頁只有五十筆資料
    theName.append(getAllName[i].text)
    theClass.append(getAllClass[i].text.replace(' ',''))
    theFans.append(getAllFans[i].find('span').text.replace(' ','')[:-1])
    theLooked.append(getAllLooked[i].find('span').text.replace(' ','')[:-1])
    theStar.append(getAllStar[i]['data-score'])
data = pd.DataFrame({
    'Youtuber名稱':theName,
    '頻道分類':theClass,
    '訂閱數量':theFans,
    '平均觀看次數':theLooked,
    'Nox評級':theStar
    })
container = pd.concat([container,data])
time.sleep(3)
```
接著將所有的數值資料進行轉換，否則預設都是類別型資料，不便於數學運算。
```python
container['訂閱數量'] = container['訂閱數量'].astype(float)
container['平均觀看次數'] = container['平均觀看次數'].astype(float)container['Nox評級'] = container['Nox評級'].astype(float)
```
您也可以將整理完的資料，利用Pandas套件中的方法to_csv()進行儲存。注意必須設定編碼為utf-8-sig，否則儲存的資料會是亂碼。另外Index=False能讓結果資料不要有index，如此一來資料看起來就不會那麼雜亂了。
```python
container.to_csv('台灣youtuber排名.csv', encoding='utf-8-sig', index=False)
```

### 繪圖
首先，要決定的是，四個象限要分別給予不同的顏色，且哪些點要顯示Youtuber的名稱。在範例中是只有顯示第一象限的Youtuber名稱，原因在於其他區域的點太過於密集，因此顯示的話會看起來非常雜亂。
```python
進行資料分析
plt.figure(figsize=(20,10))
colorlist = []
for tx,ty,ab in zip(container['訂閱數量'],container['平均觀看次數'], container['Youtuber名稱']):
    Aavg = container['訂閱數量'].mean()
    Bavg = container['平均觀看次數'].mean()
    if (tx < Aavg) & (ty < Bavg):#第三象限
        colorlist.append('#abc4d8')
    elif (tx > Aavg) & (ty < Bavg):
        colorlist.append('#abd8bf')
    elif (tx < Aavg) & (ty > Bavg):
        colorlist.append('#d8bfab')
    else:
        plt.text(tx,ty,ab, fontsize=15)# 加上文字註解
        
    colorlist.append('#d8abc4')
```
![若每個點都顯示Youtuber名稱](https://i.imgur.com/77VkZlP.png)
您可能會有疑問的地方應該是s參數，s參數控制的事圓圈的大小，而在範例中我們利用「Nox評級」的資料作為依據。為何將資料3次方又乘上50呢？原因在於讓圈圈的大小差距變大，假設兩個數值[3, 5]，兩者相距2，經過3次方後會變成[27, 125]，兩者相距變成98，更能強調兩者的差異。

```python
#繪製圓點
plt.scatter(container['訂閱數量'],container['平均觀看次數'],
             color= colorlist,
             s=container['Nox評級']*350,
             alpha=0.5)
```
繪製垂直線與水平線分別是使用axvline()與axhline()方法，linestyle參數決定線的樣式，而linewidth決定線的寬度。
```python
plt.axvline(container['訂閱數量'].mean(), color='c', linestyle='dashed', linewidth=1) # 繪製平均線    
plt.axhline(container['平均觀看次數'].mean(), color='c', linestyle='dashed', linewidth=1) # 繪製平均線
```
最後將結果圖片產出，這裡您可以選擇把最後一行savefig()方法的註解拿掉，便可以將圖片自動存檔。
```python
plt.title("意見領袖分析",fontsize=30)#標題
plt.ylabel('訂閱數量',fontsize=20)#y的標題
plt.xlabel('平均觀看次數',fontsize=20) #x的標題
plt.tight_layout()
# plt.savefig('意見領袖分析.png', dpi=300)
```
