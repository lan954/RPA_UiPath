# Tesseract OCR x UiPath   
![Python3.6](https://img.shields.io/badge/Python-3.6-blue.svg)

> 當網站登入具驗證碼(Capctha)機制時，先藉由 OpenCV 進行圖片前處理(Image Preprocessing)後，再透過 Tesseract OCR 影像識別，使得 RPA 機器人輸入正確的登入資訊，且成功登入網站。   


## 範例說明  
本專案以 RPA 機器人登入[財團法人保險事業發展中心 保險統計資料庫加值服務](http://insdb.tii.org.tw/pivot/)為範例，本次執行登入過程中，RPA 機器人在第 6 次執行時登入成功:  
- 註: 本範例執行日期 2020/11/1  

![image](./Demo.gif)

在這 6 次嘗試登入時，各次驗證碼圖片的前處理及 OCR 識別結果如下:     
| 次數 | 原始驗證碼圖片 | 圖片前處理 | Tesseract OCR 識別 |
| :----------: | ---------- | ----------- | :-----------: |
| 1 | ![image](./Captcha_Images/1st_Captcha.png) | ![image](./Captcha_Images/1st_Prediction__34.png) | **34** |
| 2 | ![image](./Captcha_Images/2nd_Captcha.png) | ![image](./Captcha_Images/2nd_Prediction__4441.png) | **4441** |
| 3 | ![image](./Captcha_Images/3rd_Captcha.png) | ![image](./Captcha_Images/3rd_Prediction__1425.png) | **1425** |
| 4 | ![image](./Captcha_Images/4th_Captcha.png) | ![image](./Captcha_Images/4th_Prediction__6037.png) | **6037** |
| 5 | ![image](./Captcha_Images/5th_Captcha.png) | ![image](./Captcha_Images/5th_Prediction__21410.png) | **21410** |
| 6 | ![image](./Captcha_Images/6th_Captcha.png) | ![image](./Captcha_Images/6th_Prediction__6615.png) | **6615** |
<br/>   
 
   
## OpenCV 及 Tesseract OCR 步驟    
透過 OpenCV 套件(Open Source Computer Vision Library)所提供圖片前處理的工具，能讓驗證碼圖片中的數字更清楚地呈現。
 
- ### Step 1 讀取驗證碼圖片  
```command
import cv2

# img_path : 驗證碼圖片的路徑(含副檔名 .png)
bgr_img = cv2.imread( img_path )

# Convert BGR to RGB
rgb_img = bgr_img[ :,:,::-1 ]
```   
![image](./Captcha_Images_Preprocessing/Step_1_Loading.png) &larr; 原始驗證碼圖片

- ### Step 2 縮放驗證碼圖片   
>> 將原始圖片從 104 x 24 (寬x高)的原大小縮放成 104 x 32 的大小:
```command
img_pixel = cv2.resize( rgb_img, (104,32) )
```  
![image](./Captcha_Images_Preprocessing/Step_2_Resize.png) 

- ### Step 3 圖片二值化  
>> 透過圖片二值化(Image Binarization)的方法，能使圖片呈現出明顯的黑白效果，從而清楚地凸顯出圖片中的目標輪廓:
```command
img_pixel = cv2.threshold( img_pixel, 20, 255, cv2.THRESH_BINARY )[1]
```  
![image](./Captcha_Images_Preprocessing/Step_3_Thresholding.png)

- ### Step 4 圖片去躁  
>> 透過圖片去躁(Image Denoising)的手法，能移除圖片中不必要且多餘的雜訊，從而保留住圖片中較重要的資訊:
```command
img_pixel = cv2.fastNlMeansDenoisingColored( img_pixel, None, 55, 55, 5, 15 )
```  
![image](./Captcha_Images_Preprocessing/Step_4_Denoising.png)

- ### Step 5 圖片二值化  
```command
img_pixel = cv2.threshold( img_pixel, 50, 255, cv2.THRESH_BINARY )[1]
```  
![image](./Captcha_Images_Preprocessing/Step_5_Thresholding.png)

- ### Step 6 圖片侵蝕  
>> 影像形態學(Morphology)中的侵蝕(Erosion)手法，其作用為將圖像中高亮度的區域進行縮減；在本專案的範例中，經過二值化及去躁處理過後的驗證碼圖片，侵蝕能加粗圖片中的數字，使資訊更加明顯可見:
```command
img_pixel = cv2.erode( img_pixel, (5,5), iterations=1 )
```  
![image](./Captcha_Images_Preprocessing/Step_6_Erosion.png)

- ### Step 7 Tesseract OCR 辨識  
>> 經由上述前處理的過程後，使用 Tesseract OCR 套件對處理過後的圖片進行辨識:  
```command
#pip install pytesseract
import pytesseract

pytesseract.pytesseract.tesseract_cmd = r'C:\\Program Files\\Tesseract-OCR\\tesseract.exe'
text = pytesseract.image_to_string( img_pixel, lang='eng', config='--psm 7 --oem 3 -c tessedit_char_whitelist=0123456789' ) 
print( text )
```  
<br/>  


## 結論    
在這個簡單範例中，雖可透過 OpenCV 的圖片前處理與 Tesseract OCR 的影像辨識，來幫助我們辨識出驗證碼中的資訊，但此方法仍有許多可改進的空間: 
- 首先，辨識的準確度並不夠高，經作者實測的經驗，平均每次 RPA 機器人嘗試登入的次數為 5 ~ 8 次，也就是說，每次執行約需 5 ~ 8 次的嘗試才能登入成功；
- 再者，若當驗證碼的圖片更加複雜時，例如: 英文數字混合、字體更加歪斜、更多的干擾線等，將大大降低此方法的可行性。   

因此，對於其他網站的驗證碼識別，若經上述兩點考量後，發覺此方法並不可行，建議可透過訓練一個 AI 模型: 卷積遞迴神經網絡(CRNN)，來克服此方法在技術上的侷限與障礙。  
> 註: 如何訓練一個卷積遞迴神經網絡(CRNN)? 可參考[ CRNN_with_CTC_Loss ](https://github.com/YenLinWu/CRNN_with_CTC_Loss)。
<br/>  


## Python 語法參考     
- [Geometric Transformations of Images](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_geometric_transformations/py_geometric_transformations.html?highlight=resize "OpenCV 圖片縮放") 
- [Image Thresholding](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_thresholding/py_thresholding.html "OpenCV 圖片二值化")
- [Image Denoising](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_photo/py_non_local_means/py_non_local_means.html "OpenCV 圖片去躁")
- [Morphological Transformations](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_morphological_ops/py_morphological_ops.html "OpenCV 圖片侵蝕")
- [Tesseract安裝](https://ithelp.ithome.com.tw/articles/10233316 "Tesseract 安裝步驟")


## 作者
<span> - &copy; Tom Wu (<a href="https://github.com/YenLinWu">Github</a>) </span>  