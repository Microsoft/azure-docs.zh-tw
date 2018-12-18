某些 Bing 回應會包含 Bing 所提供的縮圖影像 URL。 您可能會調整縮圖影像的大小並加以裁剪。 

> [!NOTE]
> 請確定縮圖的大小和裁剪可提供搜尋案例並採用第三方權利，如「Bing 搜尋 API 使用和顯示需求」中所要求。


若要調整影像大小，在縮圖的 URL 中包含 w (寬度) 和 h (高度) 查詢參數。 指定寬度和高度 (以像素為單位)。 例如︰  
  
`https://<host>/th?id=JN.5l3yzwy%2f%2fHj59U6XhssIQ&pid=Api&w=200&h=200`  
  
如果您調整影像大小，其外觀比例維持不變。 若要維持外觀比例，填白可能會新增至影像的邊界。 例如，如果您將 480x359 影像的大小調整為 200x200 但不裁剪，則全寬度可包含影像，但高度包含 25 個像素的影像頂端和底部填白。 除了左邊界和右邊界包含填白以外，如果影像為 359x480，情況也是如此。 如果您裁剪影像，則不會新增填白。  

 
下圖顯示縮圖影像的原始大小 (480x300)。  
  
![原始橫向影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-landscape.PNG)  
  
下圖顯示大小調整為 200x200 的影像。 外觀比例維持不變，而上邊界和下邊界會填白 (納入黑色邊界以顯示填補)。  
  
![已調整大小的橫向映像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-landscape-resized.PNG)  



如果您指定的維度大於影像的原始寬度和高度，則影像的左邊界和上邊界會填白。  
  
若要裁剪影像，請包含 c (裁剪) 查詢參數。 以下是您可指定的可能值。  
  
- 4&mdash;盲目比例  
- 7&mdash;智慧比例  
  
如果您要求智慧比例裁剪 (c=7)，則會從影像的感興趣區域中央向外裁剪影像，同時維持影像的外觀比例。 感興趣區域是 Bing 判斷的影像區域，其中包含大部分的匯入組件。 下圖顯示感興趣區域範例。  
  
![感興趣區域](./media/cognitive-services-bing-resize-crop/bing-resize-crop-regionofinterest.PNG)

如果您調整影像大小並要求智慧比例裁剪，則影像會縮減為所要求的大小，同時維持外觀比例。 然後會根據調整大小的維度來裁剪影像。 例如，如果已調整大小的寬度小於或等於高度，則會從感興趣區域的中央向左右兩邊裁剪影像。 否則，則會從感興趣區域的中央向上下兩邊裁剪影像。  
  
 
下圖顯示使用智慧比例裁剪縮減為 200x200 的影像。  
  
![裁剪為 200x200 的橫向影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-landscape200x200c7.PNG)
  
下圖顯示使用智慧比例裁剪縮減為 200x100 的影像。  
   
![裁剪為 200x100 的橫向影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-landscape200x100c7.PNG)
  
下圖顯示使用智慧比例裁剪縮減為 100x200 的影像。  
  
![裁剪為 100x200 的橫向影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-landscape100x200c7.PNG)



如果 Bing 無法判斷影像的感興趣區域，Bing 會使用盲目比例裁剪。  
  
如果您要求盲目比例裁剪 (c=4)，則 Bing 會使用下列規則來裁剪影像。  
  
- 如果 (原始影像寬度 / 原始影像高度) < (要求的影像寬度 / 要求的影像高度)，則會從左上角度量影像並於底部進行剪輯。  
- 如果 (原始影像寬度 / 原始影像高度) > (要求的影像寬度 / 要求的影像高度)，則會從中央度量影像並且向左右兩邊剪輯。  



下圖顯示 225x300 的直向映像。  
  
![原始向日葵影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-sunflower.PNG)
  
下圖顯示使用盲目比例裁剪縮減為 200x200 的影像。 從左上角度量影像會導致影像的下半部被裁切。  
  
![裁剪為 200x200 的向日葵影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-sunflower200x200c4.PNG)
  
下圖顯示使用盲目比例裁剪縮減為 200x100 的影像。 從左上角度量影像會導致影像的下半部被裁切。  
  
![裁剪為 200x100 的向日葵影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-sunflower200x100c4.PNG)
  
下圖顯示使用盲目比例裁剪縮減為 100x200 的影像。 從中央度量影像會導致影像的左右兩邊被裁切。  
  
![裁剪為 100x200 的向日葵影像](./media/cognitive-services-bing-resize-crop/bing-resize-crop-sunflower100x200c4.PNG)

