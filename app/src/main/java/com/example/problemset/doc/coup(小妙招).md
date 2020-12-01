场景1：启动页使用纯图片时，可能出现图片被拉伸，放大等问题。   
优化方案：纯色背景+layer-list  
具体描述：如同使用shape一样，创建<layer-list>使用即可。  
示例： 
```html
<layer-list xmlns:android="http://schemas.android.com/apk/res/android" >
     <item>
         <shape>
             <solid android:color="@android:color/white" />
         </shape>
     </item>
     <item
         android:bottom="5dp"
         android:left="2px"
         android:right="2px">
         <bitmap 
          android:src="@drawable/icon"/>
     </item>
 </layer-list>
```
<p/>

场景2：