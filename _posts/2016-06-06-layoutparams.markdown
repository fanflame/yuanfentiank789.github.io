<div id="article_content" class="article_content">
        <div class="markdown_views">

在上一篇文章里，我总结了一下自定义控件需要了解的基础知识：View的绘制流程——[《自定义控件知识储备-View的绘制流程》](http://blog.csdn.net/yisizhu/article/details/51527557)。其中，在View的测量流程里，View的测量宽高是由父控件的MeasureSpec和View自身的LayoutParams共同决定的。MeasureSpec是什么，上一篇文章里已经说得很清楚了（啥，没看过？快去路克路克，(๑•̀ㅂ•́)و✧）。而LayoutParams呢？是时候在这里做个了断了。

## LayoutParams是什么？

LayoutParams，顾名思义，就是Layout Parameters :布局参数。 

很久很久以前，我就知道LayoutParams了，并且几乎天天见面。那时候在布局文件XML里，写的最多的肯定是`android:layout_width = "match_parent"`之类的了。比如：

![常见布局文件的栗子](http://img.blog.csdn.net/20160602155119095)

    &lt;TextView
        style=<span class="hljs-string">"<span class="hljs-variable">@style</span>/text_flag_01"</span>
        android:layout_width=<span class="hljs-string">"match_parent"</span>
        android:layout_height=<span class="hljs-string">"100dp"</span>
        android:layout_marginLeft=<span class="hljs-string">"10dp"</span>
        android:layout_gravity=<span class="hljs-string">"center"</span>
        android:gravity=<span class="hljs-string">"center"</span>
        android:text=<span class="hljs-string">"英明神武蘑菇君"</span>
        android:textColor=<span class="hljs-string">"<span class="hljs-variable">@color</span>/white"</span>
        android:background=<span class="hljs-string">"<span class="hljs-variable">@color</span>/colorAccent"</span>/&gt;`</pre>

    我们都知道`layout_width`和`layout_height`这两个属性是为View指定宽度的。不过，当时年轻的我心里一直有个疑问：为什么要加上”layout_”前缀修饰呢？其它的描述属性，如`textColor`和`background`，都很正常啊！讲道理，应该用`width`和`height`描述宽高才对啊？

    后来呀，我遇到了LayoutParams，它说`layout_width`是它的属性而非View的，并且不只是针对这一个，而是所有以”layout_”开头的属性都与它有关！所以，它的东西当然要打上自己的标识”layout_”。（呵呵，嚣张个啥，到头来你自己还不是属于View的一部分(￣┰￣*)）

    既然`layout_width`这样的属性是LayoutParams定义的，那为何会出现在描述View的xml属性里呢？View和LayoutParams之间有什么恩怨纠缠呢？

    不吹不黑，咱们来看看官方文档是怎么说的：

    > 1.  LayoutParams are used by views to tell their parents how they want to be laid out.      – LayoutParams是View用来告诉它的父控件如何放置自己的。2.  The base LayoutParams class just describes how big the view wants to be for both width and height.      – 基类LayoutParams（也就是ViewGroup.LayoutParams）仅仅描述了这个View想要的宽度和高度。
> 3.  There are subclasses of LayoutParams for different subclasses of ViewGroup.      – 不同ViewGroup的继承类对应着不同的ViewGroup.LayoutParams的子类。

    看着我妙到巅峰的翻译，想必大家都看懂了&lt;(_￣▽￣_)/。看不懂？那我再来画蛇添足稍微解释一下：

1.  上面我们提到过，描述View直接用它们自己的属性就好了，如`textColor`和`background`等等，为什么还需要引入LayoutParams呢？在我看来，`textColor`和`background`这样的属性都是只与TextView自身有关的，无论这个TextView处于什么环境，这些属性都是不变的。而`layout_width`与`layout_marginLeft`这样的属性是与它的父控件息息相关的，是父控件通过LayoutParams提供这些”layout_”属性给孩子们用的；是父控件根据孩子们的要求（LayoutParams）来决定怎么测量，怎么安放孩子们的；是父控件……（写不下去了，我都快被父控件感动了，不得不再感慨一句，当父母的都不容易啊(′⌒`))  ）。所以，View的LayoutParams离开了父控件，就没有意义了。
2.  基类LayoutParams是ViewGroup类里的一个静态内部类（看吧，这就证明了LayoutParams是与父控件直接相关的），它的功能很简单，只提供了`width`和`height`两个属性，对应于xml里的`layout_width`和`layout_height`。所以，对任意系统提供的容器控件或者是自定义的ViewGroup，其chid view总是能写`layout_width`和`layout_height`属性的。
3.  自从有了`ViewGroup.LayoutParams`后，我们就可以在自定义ViewGroup时，根据自己的逻辑实现自己的LayoutParams，为孩子们提供更多的布局属性。不用说，系统里提供给我们的容器控件辣么多，肯定也有很多LayoutParams的子类啦。let us see see:

    ![ViewGroup.LayoutParams的截图](http://img.blog.csdn.net/20160602205759638)

    果然，我们看到了很多`ViewGroup.LayoutParams`的子类，里面大部分我们应该都比较熟悉。如果你觉得和它们不熟，那就是你一厢情愿啦，你早就“偷偷摸摸”的用过它们好多次了→_→

    ### ViewGroup.MarginLayoutParams

    我们首先来看看`ViewGroup.MarginLayoutParams`，看名字我们也能猜到，它是用来提供margin属性滴。margin属性也是我们在布局时经常用到的。看看这个类里面的属性：

    <pre class="prettyprint">`<span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MarginLayoutParams</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ViewGroup</span>.<span class="hljs-title">LayoutParams</span> {</span>

            <span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> leftMargin;

            <span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> topMargin;

            <span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> rightMargin;

            <span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> bottomMargin;

            <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> startMargin = DEFAULT_MARGIN_RELATIVE;

            <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> endMargin = DEFAULT_MARGIN_RELATIVE;

            ...
        }    `</pre>

    前面4个属性是我们以前在布局文件里常用的，而后面的`startMargin`和`endMargin`是为了支持RTL设计出来代替`leftMargin`和`rightMargin`的。

    > 一般情况下，View开始部分就是左边，但是有的语言目前为止还是按照从右往左的顺序来书写的，例如阿拉伯语。在Android  4.2系统之后，Google在Android中引入了RTL布局，更好的支持了从右往左文字布局的显示。为了更好的兼容RTL布局，google推荐使用MarginStart和MarginEnd来替代MarginLeft和MarginRight，这样应用可以在正常的屏幕和从右往左显示文字的屏幕上都保持一致的用户体验。

    我们除了在布局文件里用`layout_marginLeft`和`layout_marginTop`这样的属性来指定单个方向的间距以外，还会用`layout_margin`来表示四个方向的统一间距。我们来通过源码看看这一过程：

    <pre class="prettyprint">` public MarginLayoutParams(Context c, AttributeSet attrs) {
                super();

                TypedArray a = c.obtainStyledAttributes(attrs, R.styleable.ViewGroup_MarginLayout);
                setBaseAttributes(a,
                        R.styleable.ViewGroup_MarginLayout_layout_width,
                        R.styleable.ViewGroup_MarginLayout_layout_height);

                int margin = a.getDimensionPixelSize(
                        com.android.internal.R.styleable.ViewGroup_MarginLayout_layout_margin, -<span class="hljs-number">1</span>);
                <span class="hljs-keyword">if</span> (margin &gt;= <span class="hljs-number">0</span>) {
                    leftMargin = margin;
                    topMargin = margin;
                    rightMargin= margin;
                    bottomMargin = margin;
                } <span class="hljs-keyword">else</span> {
                    leftMargin = a.getDimensionPixelSize(
                            R.styleable.ViewGroup_MarginLayout_layout_marginLeft,
                            UNDEFINED_MARGIN);
                    <span class="hljs-keyword">if</span> (leftMargin == UNDEFINED_MARGIN) {
                        mMarginFlags |= LEFT_MARGIN_UNDEFINED_MASK;
                        leftMargin = DEFAULT_MARGIN_RESOLVED;
                    }
                    <span class="hljs-keyword">...</span> 
                }
                <span class="hljs-keyword">...</span>
        }        
    `</pre>

    在这个`MarginLayoutParams`的构造函数里，将获取到的xml布局文件里的属性转化成了`leftMagrin`与`rightMagrin`等值。先获取xml里的`layout_margin`值，如果未设置，则再去获取`layout_marginLeft`与`layout_marginRight`等值。所以从这里可以得出一个小结论：

    > 在xml布局里，`layout_margin`属性的值会覆盖`layout_marginLeft`与`layout_marginRight`等属性的值。

    以前我还很傻很天真的猜测，属性写在后面，就会覆盖前面的属性。虽然经过实践，也能发现上述的结论，但是自己了解了背后的原理，再去看看源码实现，自然就有更深刻的印象了。&lt;(￣ˇ￣)/

    ## 揭开隐藏的LayoutParams

    在上文中提到，我们初学Android的时候经常在“偷偷摸摸”的使用着LayoutParams，而自己却还 

    ![一脸懵逼](http://img.blog.csdn.net/20160603082647190)。 

    因为我们常用它的方式是在XML布局文件里，使用容器控件的LayoutParams里的各种属性来给孩子们布局。这种方式直观方便，直接就能在预览界面看到效果，但是同时布局也被我们写死了，无法动态改变。想要动态变化，那还是得不怕麻烦，使用代码来写。（实际上，我们写的XML布局最终也是通过代码来解析滴）

    好的，那还是让我们通过源码来揭开隐藏在`ViewGroup`里的`LayoutParams`吧！&lt;(￣︶￣)↗[GO!]……等会，我们该从哪里开始看源码呢？我认为有句名言说的在理：

    > 脱离场景谈源码，都是在耍流氓         ——英明神武蘑菇君

    上文提到，`LayoutParams`其实是父控件提供给child view的，好让child view选择如何测量和放置自己。所以肯定在child view添加到父控件的那一刻，child view就应该有`LayoutParams`了。我们来看看几个常见的添加View的方式：   

    <pre class="prettyprint">`
    LinearLayout parent = (LinearLayout) findViewById(R<span class="hljs-preprocessor">.id</span><span class="hljs-preprocessor">.parent</span>)<span class="hljs-comment">;</span>
    // <span class="hljs-number">1.</span>直接添加一个“裸”的TextView，不主动指定LayoutParams
    TextView textView = new TextView(this)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setText</span>(<span class="hljs-string">"红色蘑菇君"</span>)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setTextColor</span>(Color<span class="hljs-preprocessor">.RED</span>)<span class="hljs-comment">;</span>
    parent<span class="hljs-preprocessor">.addView</span>(textView)<span class="hljs-comment">;</span>

    // <span class="hljs-number">2.</span>先手动给TextView设定好LayoutParams，再添加
    textView = new TextView(this)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setText</span>(<span class="hljs-string">"绿色蘑菇君"</span>)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setTextColor</span>(Color<span class="hljs-preprocessor">.GREEN</span>)<span class="hljs-comment">;</span>
    LinearLayout<span class="hljs-preprocessor">.LayoutParams</span> lp = new LinearLayout<span class="hljs-preprocessor">.LayoutParams</span>(<span class="hljs-number">300</span>,<span class="hljs-number">300</span>)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setLayoutParams</span>(lp)<span class="hljs-comment">;</span>
    parent<span class="hljs-preprocessor">.addView</span>(textView)<span class="hljs-comment">;</span>

    // <span class="hljs-number">3.</span>在添加的时候传递一个创建好的LayoutParams
    textView = new TextView(this)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setText</span>(<span class="hljs-string">"蓝色蘑菇君"</span>)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setTextColor</span>(Color<span class="hljs-preprocessor">.BLUE</span>)<span class="hljs-comment">;</span>
    LinearLayout<span class="hljs-preprocessor">.LayoutParams</span> lp2 = new LinearLayout<span class="hljs-preprocessor">.LayoutParams</span>(ViewGroup<span class="hljs-preprocessor">.LayoutParams</span><span class="hljs-preprocessor">.WRAP</span>_CONTENT,<span class="hljs-number">300</span>)<span class="hljs-comment">;</span>
    parent<span class="hljs-preprocessor">.addView</span>(textView, lp2)<span class="hljs-comment">;</span>
    `</pre>

    上面代码展示的是3种往LinearLayout里动态添加TextView的方式，其中都涉及到了`addView`这个方法。我们来看看`addView`的几个重载方法：

    <pre class="prettyprint">`<span class="hljs-comment">//这3个方法都来自于基类ViewGroup</span>

     <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> addView(View child) {
            addView(child, -<span class="hljs-number">1</span>);
        }

     /*
      * @param child the child view <span class="hljs-keyword">to</span> add
      * @param index the position at which <span class="hljs-keyword">to</span> add the child    
      /
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> addView(View child, int index) {
            <span class="hljs-keyword">if</span> (child == <span class="hljs-keyword">null</span>) {
                throw <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"Cannot add a null child view to a ViewGroup"</span>);
            }
            LayoutParams params = child.getLayoutParams();
            <span class="hljs-keyword">if</span> (params == <span class="hljs-keyword">null</span>) {
                params = generateDefaultLayoutParams();
                <span class="hljs-keyword">if</span> (params == <span class="hljs-keyword">null</span>) {
                    throw <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"generateDefaultLayoutParams() cannot return null"</span>);
                }
            }
            addView(child, index, params);
        }

    <span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> addView(View child, LayoutParams params) {
            addView(child, -<span class="hljs-number">1</span>, params);
        }    
    `</pre>

    可以看出`addView(View child)`是调用了`addView(View child, int index)`方法的，在这个里面对child的LayoutParams做了判断，如果为null的话，则调用了`generateDefaultLayoutParams`方法为child生成一个默认的LayoutParams。这也合情合理，毕竟现在这个社会呀，像蘑菇君我这么懒的人太多，你要是不给个默认的选项，那别说友谊的小船了，就算泰坦尼克，那也说翻就翻！&lt;(￣︶￣)&gt;……好的，那让我们看看LinearLayout为我们这群懒人生成了怎样的默认LayoutParams：

    <pre class="prettyprint">`<span class="hljs-annotation">@Override</span>
    <span class="hljs-keyword">protected</span> LayoutParams <span class="hljs-title">generateDefaultLayoutParams</span>() {
            <span class="hljs-keyword">if</span> (mOrientation == HORIZONTAL) {
                <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
            } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (mOrientation == VERTICAL) {
                <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.WRAP_CONTENT);
            }
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
    }`</pre>

    显然，LinearLayout是重写了基类ViewGroup里的`generateDefaultLayoutParams`方法的：如果布局是水平方向，则孩子们的宽高都是`WRAP_CONTENT`，而如果是垂直方向，高仍然是`WRAP_CONTENT`，但宽却变成了`MATCH_PARENT`。所以，这一点大家得注意，因为很有可能因为我们的懒，导致布局效果和我们理想中的不一样。因此呢，第1种添加View的方式是不推荐滴，像第2或第3种方式，添加的时候指定了LayoutParams，不仅明确，而且易修改。（果然还是勤劳致富呀…）

    上面三个重载的`addView`方法最终都调用了`addView(View child, int index, LayoutParams params)`这个参数最多的方法:

    <pre class="prettyprint">`public void addView(View child, int index, LayoutParams params) {
            <span class="hljs-keyword">...</span>
            requestLayout();
            invalidate(true);
            addViewInner(child, index, params, false);
        }

    private void addViewInner(View child, int index, LayoutParams params,
                boolean preventRequestLayout) {
            <span class="hljs-keyword">...</span>

            <span class="hljs-keyword">if</span> (!checkLayoutParams(params)) {
                params = generateLayoutParams(params);
            }

            <span class="hljs-keyword">if</span> (preventRequestLayout) {
                child.mLayoutParams = params;
            } <span class="hljs-keyword">else</span> {
                child.setLayoutParams(params);
            }

            <span class="hljs-keyword">...</span>
        }    

    `</pre>

    `addView`方法又调用了方法`addViewInner`，在这个私有方法里，又干了哪些偷偷摸摸的事呢？接着来看看：

    <pre class="prettyprint">`<span class="hljs-comment">//这两个方法都重写了基类ViewGroup里的方法</span>
    <span class="hljs-comment">// Override to allow type-checking of LayoutParams.</span>
    <span class="hljs-annotation">@Override</span>
    <span class="hljs-keyword">protected</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">checkLayoutParams</span>(ViewGroup.LayoutParams p) {
        <span class="hljs-keyword">return</span> p <span class="hljs-keyword">instanceof</span> LinearLayout.LayoutParams;
    }

    <span class="hljs-annotation">@Override</span>
      <span class="hljs-keyword">protected</span> LayoutParams <span class="hljs-title">generateLayoutParams</span>(ViewGroup.LayoutParams p) {
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(p);
    }`</pre>

    `checkLayoutParams`方法的作用是检查传递进来的LayoutParams是不是LinearLayout的LayoutParam。如果不是呢？再通过`generateLayoutParams`方法根据你传递的LayoutParams的属性构造一个LinearLayout的LayoutParams。不得不再次感慨父容器控件的不容易：我们懒得设置child view的LayoutParams，甚至是设置了错误的LayoutParams，父控件都在竭尽所能的纠正我们的错误，只为了给孩子提供一个舒适的环境。(╥╯^╰╥)

    不过呀，虽然父控件可以在添加View时帮我们纠正部分错误，但我们在其他情况下错误的修改child View的LayoutParams，那父控件也爱莫能助了。比如下面这种情况：

    <pre class="prettyprint">`LinearLayout parent = (LinearLayout) findViewById(R<span class="hljs-preprocessor">.id</span><span class="hljs-preprocessor">.parent</span>)<span class="hljs-comment">;</span>
    textView = new TextView(this)<span class="hljs-comment">;</span>
    textView<span class="hljs-preprocessor">.setText</span>(<span class="hljs-string">"此处有BUG"</span>)<span class="hljs-comment">;</span>
    LinearLayout<span class="hljs-preprocessor">.LayoutParams</span> lp = new LinearLayout<span class="hljs-preprocessor">.LayoutParams</span>(<span class="hljs-number">200</span>,<span class="hljs-number">200</span>)<span class="hljs-comment">;</span>
    parent<span class="hljs-preprocessor">.addView</span>(textView, lp)<span class="hljs-comment">;</span>

    textView<span class="hljs-preprocessor">.setLayoutParams</span>(new ViewGroup<span class="hljs-preprocessor">.LayoutParams</span>(<span class="hljs-number">100</span>,<span class="hljs-number">100</span>))<span class="hljs-comment">;</span>`</pre>

    会直接报`ClassCastException`：

    <pre class="prettyprint">`<span class="hljs-label">java.lang.ClassCastException:</span> android<span class="hljs-preprocessor">.view</span><span class="hljs-preprocessor">.ViewGroup</span>$LayoutParams cannot be cast to android<span class="hljs-preprocessor">.widget</span><span class="hljs-preprocessor">.LinearLayout</span>$LayoutParams`</pre>

    上面这种异常熟悉么？反正我是相当熟悉的〒▽〒……原因就是上面代码里的textView是LinearLayout的孩子，而我们调用textView的`setLayoutParams`方法强行给它设置了一个ViewGroup的LayoutParams，所有在LinearLayout重新进行绘制流程的时候，在`onMeasure`方法里，会进行强制类型转换操作：

    <pre class="prettyprint">`LinearLayout<span class="hljs-preprocessor">.LayoutParams</span> lp = (LinearLayout<span class="hljs-preprocessor">.LayoutParams</span>) child<span class="hljs-preprocessor">.getLayoutParams</span>()<span class="hljs-comment">;</span>`</pre>

    所以App斯巴达了。也许你会说，我才不会这么傻，我知道textView的父控件是LinearLayout了，我肯定会给它设置相应的LayoutParams的！这是当然的啦，在这种明确的情况下，我们当然不会这么傻。但是，很不幸的是，有很多时候我们并不能一眼就看出来一个View的LayoutParams是什么类型的LayoutParams，这就需要动用你的智慧去分析分析啦，希望这篇文章能给你一些帮助。♪(＾∀＾●)ﾉ

    ## 自定义LayoutParams

    在本文的开头就提到过：每个容器控件几乎都会有自己的LayoutParams实现，像LinearLayout、FrameLayout和RelativeLayout等等。所以，我们在自定义ViewGroup时，几乎都要自定义相应的LayoutParams。这一节呢，就是对如何自定义LayoutParams进行一个总结。

    我以一个简单的流布局FlowLayout为例，流布局的简单定义如下：

    > FlowLayout：添加到此容器的控件自左往右依次排列，如果当前行的宽度不足以容纳下一个控件，就会将此控件放置到下一行。

    假设这个FlowLayout可以给它的孩子们提供一个gravity属性，效果就是让孩子能在某一行的垂直方向上选择三个位置：top(处于顶部)、center(居中)、bottom(处于底部)。咦？这个效果是不是和LinearLayout提供给孩子的`layout_gravity`属性很像？那好，我们来参考一下LinearLayout里的LayoutParams源码：

    <pre class="prettyprint">`<span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LayoutParams</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ViewGroup</span>.<span class="hljs-title">MarginLayoutParams</span> {</span>

            <span class="hljs-keyword">public</span> <span class="hljs-keyword">float</span> weight;

            <span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> gravity = -<span class="hljs-number">1</span>;

            <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(Context c, AttributeSet attrs) {

                <span class="hljs-keyword">super</span>(c, attrs);
                TypedArray a =
                c.obtainStyledAttributes(attrs, com.android.internal.R.styleable.LinearLayout_Layout);
                weight = a.getFloat(com.android.internal.R.styleable.LinearLayout_Layout_layout_weight, <span class="hljs-number">0</span>);
                gravity = a.getInt(com.android.internal.R.styleable.LinearLayout_Layout_layout_gravity, -<span class="hljs-number">1</span>);

                a.recycle();
            }

            <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(ViewGroup.LayoutParams p) {
                <span class="hljs-keyword">super</span>(p);
            }

            <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(ViewGroup.MarginLayoutParams source) {
                <span class="hljs-keyword">super</span>(source);
            }  

             <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(LayoutParams source) {
                <span class="hljs-keyword">super</span>(source);

                <span class="hljs-keyword">this</span>.weight = source.weight;
                <span class="hljs-keyword">this</span>.gravity = source.gravity;
            }
        }    
    `</pre>

    首先，LinearLayout里的静态内部类LayoutParams是继承`ViewGroup.MarginLayoutParams`的，所以它的孩子们都可以用margin属性。事实上，绝大部分容器控件都是直接继承`ViewGroup.MarginLayoutParams`而非`ViewGroup.LayoutParams`。所以我们的FlowLayout也直接继承`ViewGroup.MarginLayoutParams`。

    其次，LinearLayout支持两个属性`weight`和`gravity`，这两个属性在xml对应的就是`layout_weight`和`layout_gravity`。在它的构造函数`LayoutParams(Context c, AttributeSet attrs)`里，将获取到的xml布局文件里的属性转化成了`weight`与`gravity`的值。不过`com.android.internal.R.styleable.LinearLayout_Layout`这个东西是什么鬼？其实这是系统在xml属性文件里配置的`declare-styleable`，好让系统知道LinearLayout能为它的孩子们提供哪些属性支持。我们在布局的时候IDE也会给出这些快捷提示。而对于自定义的FlowLayout来说，模仿LinearLayout的写法，可以在attrs.xml文件里这么写：

    <pre class="prettyprint">`<span class="hljs-tag">&lt;<span class="hljs-title">declare-styleable</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"FlowLayout_Layout"</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-title">attr</span> <span class="hljs-attribute">name</span>=<span class="hljs-value">"android:layout_gravity"</span>/&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-title">declare-styleable</span>&gt;</span>`</pre>

    而剩下的几个构造方法起的作用就是从传递的LayoutParams参数里克隆属性了。

    依葫芦画瓢，FlowLayout的LayoutParams如下：

    <pre class="prettyprint">`<span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LayoutParams</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ViewGroup</span>.<span class="hljs-title">MarginLayoutParams</span> {</span>

            <span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> gravity = -<span class="hljs-number">1</span>;

            <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(Context c, AttributeSet attrs) {

                <span class="hljs-keyword">super</span>(c, attrs);
                TypedArray a =
                c.obtainStyledAttributes(attrs, com.android.internal.R.styleable.LinearLayout_Layout);
                weight = a.getFloat(R.styleable.FlowLayout_Layout, <span class="hljs-number">0</span>);
                gravity = a.getInt(R.styleable.FlowLayout_Layout_android_layout_gravity, -<span class="hljs-number">1</span>);

                a.recycle();
            }

             <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(ViewGroup.LayoutParams p) {
                <span class="hljs-keyword">super</span>(p);
            }

            <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(ViewGroup.MarginLayoutParams source) {
                <span class="hljs-keyword">super</span>(source);
            }  

             <span class="hljs-keyword">public</span> <span class="hljs-title">LayoutParams</span>(LayoutParams source) {
                <span class="hljs-keyword">super</span>(source);
                <span class="hljs-keyword">this</span>.gravity = source.gravity;
            }

        }    `</pre>

    看起来还是挺简单的吧？好，那我们这篇文章到此结束……等一等！好像忘记了点什么……

    ![image](http://img.blog.csdn.net/20160603223615982)

    如果对上面分析ViewGroup的`addView`方法的流程还有印象，可能你会注意ViewGroup里的这几个方法：

    <pre class="prettyprint">` <span class="hljs-keyword">public</span> LayoutParams <span class="hljs-title">generateLayoutParams</span>(AttributeSet attrs) {
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(getContext(), attrs);
        }

    <span class="hljs-keyword">protected</span> LayoutParams <span class="hljs-title">generateLayoutParams</span>(ViewGroup.LayoutParams p) {
            <span class="hljs-keyword">return</span> p;
        }

    <span class="hljs-keyword">protected</span> LayoutParams <span class="hljs-title">generateDefaultLayoutParams</span>() {
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
        }

    <span class="hljs-keyword">protected</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">checkLayoutParams</span>(ViewGroup.LayoutParams p) {
            <span class="hljs-keyword">return</span>  p != <span class="hljs-keyword">null</span>;
        }`</pre>

    为了能在添加child view时给它设置正确的LayoutParams，我们还需要重写上面几个方法（还问为啥要重写？快翻到前面再see see）。同样的，我们还是先来看看LinearLayout是怎么处理的吧：

    <pre class="prettyprint">` <span class="hljs-annotation">@Override</span>
        <span class="hljs-keyword">public</span> LayoutParams <span class="hljs-title">generateLayoutParams</span>(AttributeSet attrs) {
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LinearLayout.LayoutParams(getContext(), attrs);
        }

        <span class="hljs-annotation">@Override</span>
        <span class="hljs-keyword">protected</span> LayoutParams <span class="hljs-title">generateDefaultLayoutParams</span>() {
            <span class="hljs-keyword">if</span> (mOrientation == HORIZONTAL) {
                <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT);
            } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (mOrientation == VERTICAL) {
                <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.WRAP_CONTENT);
            }
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
        }

        <span class="hljs-annotation">@Override</span>
        <span class="hljs-keyword">protected</span> LayoutParams <span class="hljs-title">generateLayoutParams</span>(ViewGroup.LayoutParams p) {
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LayoutParams(p);
        }

        <span class="hljs-comment">// Override to allow type-checking of LayoutParams.</span>
        <span class="hljs-annotation">@Override</span>
        <span class="hljs-keyword">protected</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">checkLayoutParams</span>(ViewGroup.LayoutParams p) {
            <span class="hljs-keyword">return</span> p <span class="hljs-keyword">instanceof</span> LinearLayout.LayoutParams;
        }

那FlowLayout该如何重写上面的几个方法呢？相信聪明的你已经知道了。(๑•̀ㅂ•́)و✧

## 总结

这一篇文章从自定义控件的角度，并结合源码和表情包生动形象的谈了谈我所理解的LayoutParams。（生动，形象？真不要脸…(¯﹃¯)）。不得不说，结合源码来学习某个知识点，的确是能起到事半功倍的作用。蘑菇君初来乍到，文章里如有错误和疏漏之处，欢迎指正和补充。