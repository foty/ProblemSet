##### 使用 TabLayout，Activity应用主题不当问题：
起因:有个项目要使用一种比较古老风格(项目原因，而不是要做成这个古老)。呐，就是类似这种风格  
<图片1>  
所有的弹窗提示等都是这种风格。主题样式代码:
```
    <style name="ThemeNoTitle" parent="android:Theme">
        //...省略代码//
        
    </style>
```
后来引进TabLayout，在它的activity应用这种主题后,产生以下错误：
<图片2>

在错误日志中其实就已经说明错误原因以及解决方案。总的来说是因为包含TabLayout的那个activity的应用主题使用不
恰当，应该使用Theme.AppCompat主题或者继承自Theme.AppCompat的主题。而我的应用主题是继承至`android:Theme`。
处理方案也很明显：直接使用Theme.AppCompat主题或者继承自Theme.AppCompat的主题，或者不应用自定义主题，使用系统
默认的。   
但是这样就不符合我应用的古朴风格了，所以还是应该要去处理这个问题的。在一番探索下发现了一个解决办法。  
仔细看错误信息，发现这个错误抛出的具体位置是在一个TabLayout类中调用ThemeUtils.checkAppCompatTheme(context)方
法导致的：
<图片3>
以及ThemeUtils的代码:
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
另外：  
我的这个项目属于比较老的，compileSdkVersion = 23。因此可以这样处理，用同样的方式写个新demo(androidX)运行，同样使用非Theme.AppCompat主题
或者没有继承自Theme.AppCompat的主题，TabLayout依然会报错，但是报错信息不一样。错误定位不在TabLayout类，而是在activity上。感兴趣的
可以自己试验，就不多说了。最好的处理方式还是去修改主题吧，毕竟现在还用那么古老的风格真的不多。



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

