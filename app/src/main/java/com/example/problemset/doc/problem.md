##### 使用 TabLayout，Activity应用主题不当问题：
原因: 项目要使用一种比较古老风格(项目原因，而不是要做成这个古老)。所有的弹窗提示等都是这种风格。主题样式代码:
```
    <style name="ThemeNoTitle" parent="android:Theme">
        //...省略代码//
        
    </style>
```
后来引进TabLayout，在它的activity应用这种主题后,产生以下错误：  
`TabLayout: Error inflating class android.support.design.widget.TabLayout`  

在错误日志中其实就已经说明错误原因以及解决方案。总的来说是因为包含TabLayout的那个activity的应用主题使用不
恰当，应该使用Theme.AppCompat主题或者继承自Theme.AppCompat的主题。而我的应用主题是继承至`android:Theme`。
处理方案也很明显：直接使用Theme.AppCompat主题或者继承自Theme.AppCompat的主题，或者不应用自定义主题，使用系统
默认的。   
但是这样就不符合我应用的古朴风格了，所以还是应该要去处理这个问题的。在一番探索下发现了一个解决办法。  
仔细看错误信息，发现这个错误抛出的具体位置是在一个TabLayout类中调用ThemeUtils.checkAppCompatTheme(context)方
法导致的。ThemeUtils的代码:
```java
class ThemeUtils {
    private static final int[] APPCOMPAT_CHECK_ATTRS;

    ThemeUtils() {
    }

    static void checkAppCompatTheme(Context context) {
        TypedArray a = context.obtainStyledAttributes(APPCOMPAT_CHECK_ATTRS);
        boolean failed = !a.hasValue(0);
        if (a != null) {
            a.recycle();
        }
        if (failed) {
            throw new IllegalArgumentException("You need to use a Theme.AppCompat theme (or descendant) with the design library.");
        }
    }
    static {
        APPCOMPAT_CHECK_ATTRS = new int[]{attr.colorPrimary};
    }
}
```
从这段代码中可以发现导致抛出异常原因是在TypedArray中没有找到某个或者某些属性，这些属性来自一个APPCOMPAT_CHECK_ATTRS的数组。
看到``attr.colorPrimary``这个很容易就联想到主题当中的某个属性-colorPrimary。如下系统的style下：
```
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```
自己声明的主题又确实没有定义colorPrimary这条。于是在自己的主题下添加这个属性：
```
    <style name="ThemeNoTitle" parent="android:Theme">
       <item name="colorPrimary">#ffffff</item>
        //...省略代码//
    </style>
```
运行后TabLayout不再报错，问题解决。  

上述已发布成blog。地址:<https://blog.csdn.net/FooTyzZ/article/details/107669822>

<p>

##### UnsatisfiedLinkError,couldn't find "*.so"

部分完整的日志错误(省略部分敏感信息)
```html
 java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader[DexPathList[[zip file "*.base.apk"],nativeLibraryDirectories=[/vendor/lib, /system/lib]]] couldn't find "***.so"
        at java.lang.Runtime.loadLibrary(Runtime.java:366)
        at java.lang.System.loadLibrary(System.java:988)
        at com.* Jni.<clinit>(*.java:8)
        at com.* <init>(*.java:26)
```
错误分析: 应用使用到了.so库方法，但是没有找到so库。  

原因：还是老古董工程，AS版本为3.5.3，但项目内仍用以前的2.3.3，gradle版本为4.4。在某一天需要修改时发现无法编译通过，便修改项目使用版本为3.5.3。之后出现
以上问题。(祖传代码)

解决:   
在 build.gradle文件(app)下的 defaultConfig{ }内添加NDK配置：
```
 ndk {
    abiFilters "x86", "armeabi" // ...还有其它的cpu架构，根据自己需要配置
 }
```
若以上处理没有解决可尝试在gradle.properties文件添加：(非亲测)
```groovy
 android.useDeprecatedNdk=true
```

<p>

##### Glide(3.7)-- 加载圆角图片，四角出现黑色区域问题。  
原因：  

解决：  
使用.transform()手动转换解码，直接copy代码：
```
Glide.with(context)
        .load(imgFile)
        .dontAnimate()
        .transform(new BitmapTransformation(context) {
            @Override
            protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
                if (toTransform == null) return null;
                Bitmap result = pool.get(toTransform.getWidth(), toTransform.getHeight(), Bitmap.Config.ARGB_8888);
                if (result == null) {
                    result = Bitmap.createBitmap(toTransform.getWidth(), toTransform.getHeight(), Bitmap.Config.ARGB_8888);
                }
                Canvas canvas = new Canvas(result);
                Paint paint = new Paint();
                paint.setShader(new BitmapShader(toTransform, BitmapShader.TileMode.CLAMP, BitmapShader.TileMode.CLAMP));
                paint.setAntiAlias(true);
                RectF rectF = new RectF(0f, 0f, toTransform.getWidth(), toTransform.getHeight());
                //设置圆角大小
                canvas.drawRoundRect(rectF, DisplayUtil.dp2px(4), DisplayUtil.dp2px(4), paint);
                return result;
            }
            @Override
            public String getId() {
                return "一个id";
            }
            })
        .override(DisplayUtil.dp2px(50),DisplayUtil.dp2px(50))
        .into(imageView);
```

在`getId()`方法需要返回一个id，这个id可以使用自己项目的包名或者其他自定义字符串。

<p>

##### Glide(3.7)-- 列表加载图片出现大小不一的情况。  
原因：  

解决：  
使用.override()设置大小。或者在.transform()多增加一个参数。如CenterCrop对象。理论上只要调整.transform()方法参数即可，实际看情况使用。
完整代码：
```
 Glide.transform(new CenterCrop(context),new BitmapTransformation(context) {...})
```

<p>

##### 65535问题(旧问题，现在不知是否被优化了-2020)
原因: android系统将项目转换成dex文件时会对dex文件优化，将项目内的方法检索记录保存在链表中，链表长度类型为short。
导致项目内方法数量不能超过65536个。

解决：设置成多dex模式。   
处理步骤  
1.添加'com.android.support:multidex:1.0.0' 依赖。  
2.在自定义的Application类中重写attachBaseContext()方法，添加初始化代码: MultiDex.install(this)。

<p>

##### kotlin+Retrofit 上传RequestBody类型参数识别为？
操作环境: 使用retrofit上传图片，使用RequestBody类型作为参数上传，结果参数无法被识别出来具体描述(报错log):

> Parameter type must not include a type variable or wildcard: java.util.Map<java.lang.String, 
? extends okhttp3.RequestBody> (parameter #1)

解决方案:添加 @JvmSuppressWildcards注解RequestBody参数。  
具体代码如下:
```
@POST("/test/text")
@Multipart
fun getData(@PartMap map: Map<String, @JvmSuppressWildcards RequestBody>): Observable<ResponseData>
```
<p>

##### debug版本程序正常运行，release版本解析数据bean却为null。

解决方案(参考方向之一)：检查是否开启混淆，bean是否添加到了混淆文件。

<p>


##### RadioGroup/RadioButton 实现单选功能初始化都不选中状态问题
操作环境：单选选择，结果保存之后需要恢复到默认都不选中状态，使用
```
RadioButton1.setChecked(false)
RadioButton2.setChecked(false)
```
达到默认效果，但是再次点击时，其中一个无法被选中，等另外一个被选中后，才能被选中。  
原因： 原因未寻找  
解决方案：使用 `RadioGroup.clearCheck()`的方式替换`setChecked(false)`实现默认不选中效果。


