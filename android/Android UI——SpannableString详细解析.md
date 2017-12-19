## 前言
相信很多朋友在日常开发中都遇到过这样的问题：有一段文本，需要单独给它各部分文字设置不同的样式，有的文字设置为粗体，有的文字设置特殊的颜色，有的地方要加入表情，遇到数学公式还可能要设置上下标，这时候该怎么办呢？  

有的人可能会说：简单，不同样式的文字就用不同的TextView，这样就可以完美解决了。先不说这个方法行不行得通，事实上，若采用这种方式，当碰上一段文字需要设置非常多的样式时，光是这一堆TextView就够浪费资源的了，布局还复杂，也不利于维护，因此这种方式一般不会被采用。  

那么有其他办法吗？有，并且还很简单，今天介绍的这个SpannableString就是用来解决这个问题的。

## SpannableString

#### 什么是SpannableString？
SpannableString，是CharSequence的一种，原本的CharSequence只是一串字符序列，没有任何样式，而SpannableString可以在字符序列基础上对指定的字符进行润饰，在开发中，TextView可以通过setText(CharSequence)传入SpannableString作为参数，来达到显示不同样式文字的效果。  

创建方式
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");

```
#### 如何对SpannableString进行润饰？
一般通过以下方式进行设置

```
spannableString.setSpan(Object what, int start, int end, int flags);
```
这里讲解一下几个参数的意义  
- what：对SpannableString进行润色的各种Span；  
- int：需要润色文字段开始的下标；  
- end：需要润色文字段结束的下标；  
- flags：决定开始和结束下标是否包含的标志位，有四个参数可选
    + SPAN_INCLUSIVE_EXCLUSIVE：包括开始下标，但不包括结束下标
    + SPAN_EXCLUSIVE_INCLUSIVE：不包括开始下标，但包括结束下标
    + SPAN_INCLUSIVE_INCLUSIVE：既包括开始下标，又包括结束下标
    + SPAN_EXCLUSIVE_EXCLUSIVE：不包括开始下标，也不包括结束下标

这里涉及到一个重要的角色，就是各种各样的span，它决定我们要对文字的进行怎样的润饰，而后三个参数决定润饰哪些文字，为了方便起见，后面的flags默认都使用SPAN_INCLUSIVE_EXCLUSIVE模式。

## 各种Span

先来看一张类结构图，了解各种Span之间的关系  

![Span类结构图](http://upload-images.jianshu.io/upload_images/3279407-426f8688601a6846?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看出所有Span都继承于CharacterStyle这个抽象类，另外MetricAffectingSpan、ReplacementSpan和ClickableSpan都是抽象类，下面展示一些常用的Span

#### ForegroundColorSpan
代码

```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
ForegroundColorSpan foregroundColorSpan = new ForegroundColorSpan(Color.GREEN);
spannableString.setSpan(foregroundColorSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
ForegroundColorSpan：前景色，也就是对文字上色，颜色设置为GREEN，start为4，end为7，应该是“陈奕迅”三个字显示为绿色，看一下实际效果  

![ForegroundColorSpan](http://upload-images.jianshu.io/upload_images/3279407-13a18b2550659258?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### BackgroudColorSpan
代码

```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
BackgroundColorSpan backgroundColorSpan = new BackgroundColorSpan(Color.GREEN);
spannableString.setSpan(backgroundColorSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
BackgroudColorSpan：与ForegroundColorSpan类似，对文字背景上色  

![BackgroudColorSpan](http://upload-images.jianshu.io/upload_images/3279407-f03bf6741c014b7a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ClickableSpan
代码

```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
ClickableSpan clickableSpan = new ClickableSpan() {
    @Override
    public void onClick(View widget) {
        Toast.makeText(MainActivity.this, "如果我是陈奕迅", Toast.LENGTH_SHORT).show();
    }
    @Override
    public void updateDrawState(TextPaint ds) {
        ds.setUnderlineText(false);
    }
};
spannableString.setSpan(clickableSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setMovementMethod(LinkMovementMethod.getInstance());
mTextView.setText(spannableString);
```
ClickableSpan：是一个抽象类，实现可点击效果，可以重写onClick方法实现点击事件，这里点击“陈奕迅”三个字简单地弹toast  

![ClickableSpan](http://upload-images.jianshu.io/upload_images/3279407-cc965d594f63ad6e?imageMogr2/auto-orient/strip)

### URLSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
URLSpan urlSpan = new URLSpan("https://www.baidu.com/s?ie=UTF-8&wd=陈奕迅");
spannableString.setSpan(urlSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setMovementMethod(LinkMovementMethod.getInstance());
mTextView.setText(spannableString);
```
URLSpan：实现超链接的效果，继承于ClickableSpan，点击实现跳转到浏览器  

![URLSpan](http://upload-images.jianshu.io/upload_images/3279407-9a811a4bbc37a169?imageMogr2/auto-orient/strip)


### MaskFilterSpan
代码

```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
MaskFilterSpan embossMaskFilterSpan =
    new MaskFilterSpan(new EmbossMaskFilter(new float[]{10, 10, 10}, 0.5f, 1, 1));
spannableString.setSpan(embossMaskFilterSpan, 0, 4, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
RelativeSizeSpan relativeSizeSpan = new RelativeSizeSpan(1.5f);
spannableString.setSpan(relativeSizeSpan, 0, 4, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
MaskFilterSpan blurMaskFilterSpan = new MaskFilterSpan(new BlurMaskFilter(10, Blur.NORMAL));
spannableString.setSpan(blurMaskFilterSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
MaskFilterSpan：构造方法接受MaskFilter作为参数，其中它有两个子类：EmbossMaskFilter和BlurMaskFilter  
EmbossMaskFilter实现浮雕效果
```
EmbossMaskFilter(float[] direction, float ambient, float specular, float blurRadius)
```
- direction：float数组，定义长度为3的数组标量[x,y,z]，来指定光源的方向
- ambient：环境光亮度，0~1
- specular：镜面反射系数
- blurRadius：模糊半径，必须>0

BlurMaskFilter实现模糊效果

```
BlurMaskFilter(float radius, Blur style)
```
- radius：模糊半径  
- style：有四个参数可选
    - BlurMaskFilter.Blur.NORMAL：内外模糊
    - BlurMaskFilter.Blur.OUTER：外部模糊
    - BlurMaskFilter.Blur.INNER：内部模糊
    - BlurMaskFilter.Blur.SOLID：内部加粗，外部模糊

![MaskFilterSpan](http://upload-images.jianshu.io/upload_images/3279407-5f324e66df09ad26?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### RelativeSizeSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
RelativeSizeSpan relativeSizeSpan = new RelativeSizeSpan(1.5f);
spannableString.setSpan(relativeSizeSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
RelativeSizeSpan：设置字体的相对大小，这里设置为TextView大小的1.5倍，看图  

![RelativeSizeSpan](http://upload-images.jianshu.io/upload_images/3279407-f4807205d09aaf02?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### AbsoluteSizeSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
AbsoluteSizeSpan absoluteSizeSpan = new AbsoluteSizeSpan(40, true);
spannableString.setSpan(absoluteSizeSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
AbsoluteSizeSpan：设置字体的相绝对大小，40表示文字大小，true表示单位为dip，若为false则表示px  

![AbsoluteSizeSpan](http://upload-images.jianshu.io/upload_images/3279407-7c981e0df4a6acb8?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ScaleXSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
ScaleXSpan scaleXSpan= new ScaleXSpan(1.5f);
spannableString.setSpan(scaleXSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
ScaleXSpan：设置字体x轴缩放，1.5表示x轴放大为1.5倍，效果如图  
![ScaleXSpan](http://upload-images.jianshu.io/upload_images/3279407-4a0b22b0386a660d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### StyleSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
StyleSpan boldSpan = new StyleSpan(Typeface.BOLD);
StyleSpan italicSpan = new StyleSpan(Typeface.ITALIC);
StyleSpan boldItalicSpan = new StyleSpan(Typeface.BOLD_ITALIC);
spannableString.setSpan(boldSpan, 0, 2, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(italicSpan, 2, 4, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(boldItalicSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
StyleSpan：设置文字样式，如斜体、粗体  

![StyleSpan](http://upload-images.jianshu.io/upload_images/3279407-2c05b1c69590d9d9?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### TypefaceSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
TypefaceSpan monospace = new TypefaceSpan("monospace");
TypefaceSpan serif = new TypefaceSpan("serif");
TypefaceSpan sans_serif = new TypefaceSpan("sans-serif");
spannableString.setSpan(monospace, 0, 2, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(serif, 2, 4, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(sans_serif, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
TypefaceSpan：设置文字字体类型，如monospace、serif和sans-serif等等  

![TypefaceSpan](http://upload-images.jianshu.io/upload_images/3279407-91e092103d5700aa?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### TextAppearanceSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
TextAppearanceSpan textAppearanceSpan = new TextAppearanceSpan(this, android.R.style.TextAppearance_Material);
spannableString.setSpan(textAppearanceSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
TextAppearanceSpan：设置文字外貌，通过style资源设置，这里使用系统的style资源  

![TextAppearanceSpan](http://upload-images.jianshu.io/upload_images/3279407-0faf3f36473230a4?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### UnderlineSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
UnderlineSpan underlineSpan = new UnderlineSpan();
spannableString.setSpan(underlineSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
UnderlineSpan：设置文字下划线，强调突出文字时可以使用该span  

![UnderlineSpan](http://upload-images.jianshu.io/upload_images/3279407-aded14e1038ec9cc?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### StrikethroughSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
StrikethroughSpan strikethroughSpan = new StrikethroughSpan();
spannableString.setSpan(strikethroughSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
StrikethroughSpan：设置文字删除线  

![StrikethroughSpan](http://upload-images.jianshu.io/upload_images/3279407-c62f06c0f905a2d0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### SuperscriptSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
SuperscriptSpan superscriptSpan = new SuperscriptSpan();
RelativeSizeSpan relativeSizeSpan = new RelativeSizeSpan(0.8f);
spannableString.setSpan(relativeSizeSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(superscriptSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
SuperscriptSpan：设置文字为上标  

![SuperscriptSpan](http://upload-images.jianshu.io/upload_images/3279407-d646beac88cef2de?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### SubscriptSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
SubscriptSpan subscriptSpan = new SubscriptSpan();
RelativeSizeSpan relativeSizeSpan = new RelativeSizeSpan(0.8f);
spannableString.setSpan(relativeSizeSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
spannableString.setSpan(subscriptSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
SubscriptSpan：设置文字为下标  

![SubscriptSpan](http://upload-images.jianshu.io/upload_images/3279407-94e49581ac9b16ff?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ImageSpan
代码
```
SpannableString spannableString = new SpannableString("如果我是陈奕迅");
ImageSpan imageSpan = new ImageSpan(this, R.drawable.ic_eason);
spannableString.setSpan(imageSpan, 4, 7, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
mTextView.setText(spannableString);
```
ImageSpan：设置图片  

![ImageSpan](http://upload-images.jianshu.io/upload_images/3279407-7d580ccb20d9e506?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 总结

总结一下以上提到的Span

- ForegroundColorSpan：前景色
- BackgroundColorSpan：背景色
- ClickableSpan：抽象类，可点击效果，重写onClick方法响应点击事件
- URLSpan：超链接
- MaskFilterSpan：EmbossMaskFilter浮雕效果，BlurMaskFilter模糊效果
- RelativeSpan：文字相对大小
- AbsoluteSpan：文字绝对大小
- ScaleXSpan：x轴缩放
- styleSpan：文字样式
- TypefaceSpan：文字字体类型
- TextApearanceSpan：文字外貌
- UnderlineSpan：下划线
- StrikeThroughSpan：删除线
- SuperscriptSpan：上标
- SubscriptSpan：下标
- ImageSpan：图片

这些Span能够很好地帮助我们润色文字，以非常简单地方式获得复杂和绚丽的文字效果，着实是开发中的一大利器，喜欢的朋友收藏备用吧

### **感谢阅读！**

### **欢迎关注个人微信公众号：Charming写字的地方**
