title: 泛型函数
id: 158
categories:
  - Blog
date: 2015-01-07 16:26:45
tags:
---

今天看到一黑科技....

泛型类 泛型参数倒是很常用...

用在返回值类型上的泛型函数我还是第一次见到这么玩的...

写起代码来大概是这样

&nbsp;

<pre class="lang:java decode:true">public class MainActivity extends Activity{
    private Button mButton;
    private TextView mTextView;
    private ListView mListView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mButton = $(R.id.button);
        mTextView = $(R.id.textview);
        mListView = $(R.id.listview);

    }

    public &lt;T extends View&gt; T $(int id)
    {
        return (T) super.findViewById(id);
    }
}</pre>

&nbsp;

一种JQuery的感觉。。。主要是这个$伪装成操作符的样子很有迷惑性...当然JQuery里的$本身也是这样实现的.....

&nbsp;

好处大概是代码压缩的厉害。。省掉了强制类型转换。。写起代码来清晰一些。。

使用起来方便。。一共才2行代码。。扔BaseActivity里就好了。。

当然功能很弱逼。。。比注解形的弱多了。。。