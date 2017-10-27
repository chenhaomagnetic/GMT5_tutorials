
### 目錄
1. [總覽](/index.md)
2. [GMT介紹及安裝](/intro_install.md)
3. [網路資源及配套軟體](/net_software.md)
4. [第零章: 基本概念及默認值](/basic_defaults.md)
5. [第一章: 製作地圖(地理投影法)](/projection.md)
6. [第二章: XY散佈圖(其他投影法)](/xy_figure.md)
7. [第三章: 等高線圖及剖面](/contour_profile.md)
8. [第四章: 地形圖與色階](/topographic_cpt.md)

---

## 8. 地形圖及色階
當想要呈現三維的資料，可以利用xyz座標系畫出立體圖，但如果三維資料要呈現在二維平面上時，
除了利用等高線之外，還可以利用顏色來做為第三維度的變化，因此衍生出了色階，
在GMT裡稱作.cpt(color palette tables)。而這種方式被廣泛利用在各種成果展示上面，
其中一個就是地形圖，本章將介紹如何繪製地形圖以及搭配的色階檔。

## 8.1 目的
本章將學習如何繪製
  1. 簡介色階檔(cpt, color palette tables)
  2. 地形圖(Topography)
  3. 地形暈渲面(Hillshading)

## 8.2 學習的指令與概念

* `grdgradient`: 計算網格檔的梯度與照明度
* `grdimage`: 繪製色階地圖
* `grd2cpt`: 利用網格檔資料建立色階檔
* `psscale`: 繪製色階
* `makecpt`: 製作色階檔

## 8.3 簡介色階檔
色階的概念是將某一區段的資料用一種顏色來表示，這種技巧被應用在很多方面，像是熱影像圖、地形圖等等。
GMT在安裝的時後，同時有提供一些色階檔作為使用，這些檔案被安裝在==GMT根目錄/share/cpt==，
將打開==abyss.cpt==，裡面的內容:
```bash
#       $Id$
#
# Color table for bathymetry modeled after IBCSO
# at depth but turning lighter towards sea level.
# Designed by P. Wessel, SOEST
# COLOR_MODEL = RGB
-8000      black  	-7000	20/30/53
-7000	20/30/53  	-6000	38/60/106
-6000	38/60/106	-5000	46/80/133
-5000	46/80/133	-4000	53/99/160
-4000	53/99/160	-3000	72/151/211
-3000	72/151/211	-2000	90/185/233
-2000	90/185/233	-1000	141/210/239
-1000	141/210/239	0	245/255/255
B black
F white
N gray
```
一開始`#`之後的文字為註解，通常會紀錄設計者名稱、設計目的、使用的顏色模型，接著檔案的格式為四欄，
第一欄是起始範圍；第二欄色碼；第三欄是結束範圍；第四欄色碼，所以在-8000至-7000就是以黑色來表示，
在檔案的最下面**B**、**F**、**N**分別代表背景色(Background color)、前景色(Foreground color)及無值的顏色(NaN color)，
簡單地說，就是小於範圍中最小值所用的顏色、大於範圍中最大值值所用的顏色及無資料時所使用的顏色。
來看GMT有提供的色階檔畫出來的樣子:
<p align="center">
  <img src="fig/8_3_cpt_1.png"/>
</p>
圖中列出36個GMT默認提供的色階檔(GMT version 5.2.1)，將每個色階做正規化的步驟，上方色階代表顏色連續的色階檔，
下方則是顏色離散的色階檔(間隔0.25)，如果原始的色階檔就不連續，那做出來就是不連續的色階檔。
而兩者檔案中，有什麼差別呢？
```bash
# 不連續的色階
0	166/206/227	1	166/206/227
1	31/120/180	2	31/120/180

# 連續的色階
0	38/60/106	1 46/80/133
1	46/80/133	2 53/99/160
```
有看出差別了嗎？在不連續的色階中，0~1都是用同一種顏色，而連續的色階中0~1是不同顏色，而接續下去1~2也是如此，
GMT就是用這種方式，來區分二者的差異。接下來，示範如何用`makecpt`來製作色階檔。自己手動編輯色階檔也是可以，
但需要對顏色的敏感度非常好，時常色階檔的製作，都是利用現成的檔案進行範圍的改寫。
```bash
makecpt -Ccool.cpt -T0/50/10 -D -Z > tmp.cpt
# 產生的tmp.cpt內容如下
0	cyan	10	51/204/255
10	51/204/255	20	102/153/255
20	102/153/255	30	153/102/255
30	153/102/255	40	204/51/255
40	204/51/255	50	magenta
B	cyan
F	magenta
N	127.5
```
學習到的指令:
* `makecpt`:
  * `-C`輸入的色階檔。
  * `-T`範圍的設定，有三種方式:
    1 **最小值/最大值/間隔**
    2 **讀取一個檔案**
    3 **間隔點1,間隔點2,間隔點3,...**
  * `-D`讓前景色與背景色對應最小及最大值的色碼
  * `-Z`製作連續的色階

學會了如何改寫色階檔之後，就是如何調整範圍，來配合資料，達到良好呈獻效果。此外，推薦一個網站，
[cpt-city](http://soliton.vm.bytemark.co.uk/pub/cpt-city/)，裡面收集了大量的現成色階檔，供使用參考。

## 8.4 地形圖
陽明山國家公園是以大屯山火山群為主的火山地型，規劃了眾多的景點，供民眾認識火山地形、地貌。
本節將利用陽明山區域的20公尺數值地形，配合GMT內建安裝的數值檔，來演示不同的色階檔，
對繪製地形圖所帶來的影響。
切割的範圍`121.27/121.85/25.05/25.35`，可以自行練習`grdcut`，或者直接下載下方載點，

使用的資料檔:
- [陽明山數值高程](dat/yangmingShan.grd)

成果圖
<p align="center">
  <img src="fig/8_4_yangmingShan_1.png"/>
</p>

批次檔
```bash
set ps=8_4_yangmingShan.ps
set cpt=dem1.cpt

# use default dem1.cpt
gmt grdimage yangmingShan.grd -R121.27/121.85/25.05/25.35 -JM13 -BWeSn -Ba ^
-C%cpt% -P -Y21 -K > %ps%
gmt pscoast -R -JM -Df -W1 -S34/201/237 -K -O >> %ps%
gmt psscale -C%cpt% -Dx14/0+w7/.5+e -Ba100+l"Elevation (m)" -K -O >> %ps%
echo 121.3 25.32 1. | gmt pstext -R -JM -F+f24p -G250 -K -O >> %ps%

# discrete 0~1200 dem1.cpt 
makecpt -C%cpt% -T0/1200/100 > tmp.cpt
gmt grdimage yangmingShan.grd -R121.27/121.85/25.05/25.35 -JM13 -BWeSn -Ba -Ctmp.cpt ^
-M -Y-9 -K -O >> %ps%
gmt pscoast -R -JM -Df -W1 -S34/201/237 -K -O >> %ps%
gmt psscale -Ctmp.cpt -Dx14/0+w7/.5 -Ba200+l"Elevation (m)" -M -K -O >> %ps%
echo 121.3 25.32 2. | gmt pstext -R -JM -F+f24p -G250 -K -O >> %ps%

# continuous 0~1200 dem1.cpt 
makecpt -C%cpt% -T0/1200/100 -Z > tmp.cpt
gmt grdimage yangmingShan.grd -R121.27/121.85/25.05/25.35 -JM13 -BWeSn -Ba -Ctmp.cpt ^
-Y-9 -K -O >> %ps%
gmt pscoast -R -JM -Df -W1 -S34/201/237 -K -O >> %ps%
gmt psscale -Ctmp.cpt -Dx14/0+w7/.5 -Ba200+l"Elevation (m)" -I -K -O >> %ps%
echo 121.3 25.32 3. | gmt pstext -R -JM -F+f24p -G250 -K -O >> %ps%

gmt psxy -R -JM -T -O >> %ps%
gmt psconvert %ps% -Tg -A -P
del tmp*
```

本節學習到的新指令:
1. 使用dem1.cpt繪製地形圖。
  * `grdimage`投影網格或是影像(images)至地圖上:
    * `-C`輸入色階檔。
    * `-M`黑白畫面!!
  * `psscale`繪製色彩條(color bar):
    * `B`調整刻度間距。
    * `-C`輸入色階檔。
    * `-D`調整色彩條的位置。
      * 參考點X位移/Y位移，其餘參考點設定可[參考6-6](xy_figure.md#m6.6)
      * **+w**長度/寬度
      * **+e**顯示前景色(**f**)、背景色(**b**)
      * **+h**改成水平色彩條；**+v**垂直色彩條(默認值)
      * **+m**將**a**(annotation, 註解), **l**(label, 標籤), **u**(unit 單位)，
      位置改放至對面，**c**(標籤轉成垂直)
      * **+n**在一開始加上無值的的矩形範圍
  * `-F`加上邊框:
    * **+g**填色
    * **+i**間距/筆觸，在加上一個內部的邊框
    * **+p**邊框筆觸
    * **+r**弧度，圓角矩形邊框
    * **+s**陰影
  * `-I`開啟照明效果。
  * `-M`轉成黑白模式。
  * `-S`取消自動區分不同顏色的黑線。
2. 將色階檔範圍改成0至1200，間距100，變成離散的色階檔，加上`-M`讓地圖與色彩條都變成黑白模式。
3. 將色階檔範圍改成0至1200，間距100，`-Z`變成連續的色階檔，`-I`開啟彩色條照明效果。



## 8.5 地形暈渲面

## 8.6 習題

## 8.7 參考批次檔
列出本章節使用的批次檔，供讀者參考使用，檔案路經可能會有些許不同，再自行修改。

---

[上一章](/contour_profile.md) -- [下一章](/topography_cpt.md)