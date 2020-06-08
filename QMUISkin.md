无论是QMUI的主题配置，还是换肤，这些你首先得明白下面的使用：

```
attr    ?attr
```

我的理解是（我没有去了解过，只是凭着代码自我理解）：

attr 是一个变量，给一个名字 name , 给它一个类型 format , 一个完整的如下：

```
<!-- 主色调 -->
<attr name="app_skin_primary_color" format="color"/>
```
然后我们在主题里为其设置值

```
 <style name="BaseTheme" parent="QMUI.Compat.NoActionBar">
 
     ....
     <item name="app_skin_primary_color">@color/app_color_theme_6</item>
     ....
    
 </style>

```

最后使用

```
    <View
        ...
        android:background="?attr/app_skin_primary_color"/>
```

知道了这两个是干嘛用的，那么就可以开始了。

[QMUI换肤](https://github.com/Tencent/QMUI_Android/wiki/QMUI-%E6%8D%A2%E8%82%A4)里用的最多的，最根本的就是第四点【为 View 配置 skin】。

现在就刚刚我设置的一个主题色**app_skin_primary_color**设为一个**TextView**的字体颜色，按照这个文档里的那么就是

```
    <TextView
        ...
        app:qmui_skin_text_color="?attr/app_skin_primary_color" />
```

当主题切换后，这个设置了的TextView的字体颜色就会随着配置的主题颜色的不一样而发生改变。

其他属性同理。



下面是一些零碎点：


### 1.如何让自定义的View也能换肤？

比如有一个自定义的ProgressBar,Bar的颜色我们要根据主题色的变化而变化。

首先，View实现IQMUISkinHandlerView接口，然后在

```
    @Override
    public void handle(@NonNull QMUISkinManager manager, int skinIndex, @NonNull Resources.Theme theme, @Nullable SimpleArrayMap<String, Integer> attrs) {
       // 获取当前的主题色
       barColor = QMUISkinHelper.getSkinColor(this,R.attr.app_skin_primary_color);
       ...
       // 画笔设置主题色
       barPaint.setColor(barColor);
       ...
       // 重绘
		   invalidate();
    }
```

handle就是通知你，主题改变了，你要做什么改变就在这里快点做啊。

### 2.如何让一个View不要换肤？

比如QMUITopBarLayout,有的界面我就希望它是透明的颜色，不要随着主题改变而改变。

（目前QMUITopBarLayout还没有这个，我是Copy出来的，自己改，QMUI后续版本会有）

首先实现IQMUISkinDispatchInterceptor，在这里返回true,换肤就不会往下执行了,也会向子View传递了。
这个我们可以在QMUI提供的QMUISkinManager里有这样写到

```
 private void runDispatch(@NonNull View view, int skinIndex, Resources.Theme theme) {
       ...
       
        if (view instanceof IQMUISkinDispatchInterceptor) {
            if (((IQMUISkinDispatchInterceptor) view).intercept(skinIndex, theme)) {
                return;
            }
        }
        
        ...

```

### 3.QMUIGroupListView

当ICON有渐变色时，这个时候显示的ICON会因为tintColor让整个ICON全部变成一个颜色，所以这个时候要禁用掉，方法如下：

```
QMUICommonListItemView itemView;
... itemView 的其他配置
if(!isIconWithTintColor()){
  // 去除 icon 的换肤设置
  QMUICommonListItemView.SkinConfig skinConfig = new QMUICommonListItemView.SkinConfig();
  skinConfig.iconTintColorRes = 0;
  itemView.setSkinConfig(skinConfig);
}
``

### 4.DialogFragment、Fragment、Dialog等

如果依附的Activity是QMUI提供的，那么只需要在这些里面注入QMUISkinManager

```
public abstract class BaseDialogFragment extends DialogFragment {

    /**
     * 是否使用QMUI的皮肤管理
     * @return
     */
    protected boolean useQMUISkinManager() {
        return true;
    }

    private QMUISkinManager mSkinManager;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (useQMUISkinManager())
            mSkinManager = QMUISkinManager.defaultInstance(getContext());
    }

    @Override
    public void onStart() {
        super.onStart();
        if (mSkinManager != null) {
            mSkinManager.register(this);
        }
    }

    @Override
    public void onStop() {
        super.onStop();
        if (mSkinManager != null) {
            mSkinManager.unRegister(this);
        }
    }
}

```

如果连Activity也不是，则根据[QMUISkin文档](https://github.com/Tencent/QMUI_Android/wiki/QMUI-%E6%8D%A2%E8%82%A4)第3点第1条写入即可。


