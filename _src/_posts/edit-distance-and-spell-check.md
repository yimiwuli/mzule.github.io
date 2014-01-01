title:最小编辑距离，键盘距离与拼写纠正
time:2013-10-27 11:22:12
template:post.html
---
`最小编辑距离`是指将一个错误拼写的单词纠正正确的最小编辑次数，这里的编辑包含插入、删除、修改三种操作，每一次编辑只能改变一个字母。因为这个概念是俄罗斯科学家 Vladimir Levenshtein 在1965年提出来的，所以编辑距离又称为`Levenshtein`距离。

就拿`Levenshtein`这个单词举例说明好了，`Levenshtein`作为一个人名，很容易会被拼写错误。假设现在有一个错误拼写的`Lavensting`. 那么他们的编辑距离是多少呢？

参考正确的单词`Levenshtein`，可以看出，由`Lavensting`纠正为`Levenshtein`的步骤为:
`Lavensting` V.S. `Levenshtein`

1. 将第二个字母`a`修改为`e`；
2. 在第六个字母`s`后面插入`h`；
3. 在第七个字母`t`后面插入`e`；
4. 将最后一个字母`g`删除。

这里我们进行了四次操作，所以`Lavensting`和`Levenstein`的编辑距离是`4`. 而且我们可以目测出这已经是最小编辑距离了。

那么我们怎么使用最小编辑距离为错误拼写的单词进行纠正呢？原理很简单，也很粗暴。用单词库里面的所有单词与错误拼写的单词计算最小编辑距离，最小编辑距离最小的单词，便很可能是正确的单词，也就是纠正的结果。

接下来，接下来便是如何使用计算机求解最小编辑距离。动态规划经常被用来作为这个问题的解决手段之一。

笔者水平有限，动态规划难以描述清楚，这里给一个定义：动态规划是通过把原问题分解为相对简单的子问题的方式求解复杂问题的方法。

下面是java代码：
```
package com.mzule.al;

public class LevenshteinDistance {

    public double distance(String w1,String w2){
        double[][] m = new double[w1.length()+1][w2.length()+1];
        for(int i=0;i<m.length;i++){
            m[i][0]=i;
        }
        for(int i=0;i<m[0].length;i++){
            m[0][i]=i;
        }
        for(int i=1;i<m.length;i++){
            for(int j=1;j<m[0].length;j++){
                m[i][j] = min(m[i][j-1]+1,m[i-1][j]+1,m[i-1][j-1]+cost(w1.charAt(i-1),w2.charAt(j-1)));
            }
        }
        return m[w1.length()][w2.length()];
    }

    protected double cost(char c1,char c2) {
        return c1==c2?0:1;
    }

    protected double min(double i, double j, double k) {
        double t = i<j?i:j;
        return t<k?t:k;
    }
    
}
```
上面的核心代码是两个for循环中的部分：

```
m[i][j] = min(m[i][j-1]+1,m[i-1][j]+1,m[i-1][j-1]+cost(w1.charAt(i-1),w2.charAt(j-1)));
```

这段代码可以这样理解，例如我们已知`ab`与`a`、`a`与`a`、`a`与`ac`的距离，计算字符串 `ab`与`ac`的距离，有三个方式，对应于编辑操作的插入、删除、修改三种操作：

1. 在`ab`与`a`的距离基础上+1，因为`ac`多了一个`c`；
2. 在`a`与`ac`的距离的基础上+1，因为ab多了一个`b`；
3. 在`a`与`a`的距离的基础上加上`b`与`c`的距离，`b`与`c`的距离很简单，因为`b`与`c`不相等，为1

测试上面的代码运行效果：

```
public static void main(String[] args) {
    double d = new LevenshteinDistance().distance("Lavensting", "Levenshtein");
    System.out.println(d);
}
```

输出的结果为：`4.0`，和我们之前目测的结果一样。

好了，现在我们可以用这个算法去做拼写纠错了。

但是这个算法不够完美，因为没有考虑到键盘距离。

在上面程序中的`cost`方法，只是简单的对相同的字符返回`0`，不同的字符返回`1`。这种情况下，`cost(‘a’,‘p’)`和`cost(‘a’,‘s’)`的值是一样的，都是`1`.但是他们应该是不一样的，因为`a`被误输入为`s`的概率比误输入为`p`的概率大得多，因为在键盘上，`a`与`s`是邻居，手指很容易误按，而`p`与`a`距离太远，用户输入`p`基本上都是真实的想法。

所以我们要对上面的算法进行改进，引入新的`cost`计算机制：

```
package com.mzule.al;

import java.util.HashMap;
import java.util.Map;

public class KeyboardLevenshteinDistance extends LevenshteinDistance {

    private static final Map<Character, String> charSiblings;
    private static final double SCORE_MIS_HIT = 0.1;

    static {
        charSiblings = new HashMap<>();
        charSiblings.put('q', "was");
        charSiblings.put('w', "qsead");
        charSiblings.put('e', "wsdfr");
        charSiblings.put('r', "edfgt");
        charSiblings.put('t', "rfghy");
        charSiblings.put('y', "tghju");
        charSiblings.put('u', "yhjki");
        charSiblings.put('i', "ujklo");
        charSiblings.put('o', "ikl;p");
        charSiblings.put('p', "ol;'[");
        charSiblings.put('a', "qwsxz");
        charSiblings.put('s', "qazxcdew");
        charSiblings.put('d', "wsxcvfre");
        charSiblings.put('f', "edcvbgtr");
        charSiblings.put('g', "rfvbnhyt");
        charSiblings.put('h', "tgbnmjuy");
        charSiblings.put('j', "yhnm,kiu");
        charSiblings.put('k', "ujm,.loi");
        charSiblings.put('l', "ik,./;po");
        charSiblings.put('z', "asx");
        charSiblings.put('x', "zasdc");
        charSiblings.put('c', "xsdfv");
        charSiblings.put('v', "cdfgb");
        charSiblings.put('b', "vfghn");
        charSiblings.put('n', "bghjm");
        charSiblings.put('m', "nhjk,");
    }

    @Override
    protected double cost(char c1, char c2) {
        return keyboardDistance(c1, c2);
    }

    private double keyboardDistance(char c1, char c2) {
        if (c1 == c2) {
            return 0;
        }
        String s = charSiblings.get(c1);
        if (s != null && s.indexOf(c2) > -1) {
            return SCORE_MIS_HIT;
        }
        return 1;
    }
    
}
```

上面的类继承自`LevenshteinDistance` ，重写了`cost`方法，根据键盘距离，对相邻的字母返回`0.1`，不相邻的字母返回距离`1`.

```
cost('a','s')=0.1
cost('a','p')=1
```

测试`KeyboardLevenshteinDistance`:

```
public static void main(String[] args) {
    double d = new KeyboardLevenshteinDistance().distance("thanks", "tjsmla");
    System.out.println(d);
}
```

输出：`0.5`，和预期的结果一样。`tjsmla`与`thanks`非常相似，因为在新买的键盘还是不熟悉的情况下，误输入`thanks`为`tjsmla`也很正常。

上面的算法完美了吗？of course not.

还有很多优化空间。比如：除了键盘距离，我们还可以考虑读音距离，对于读音相似的字母，也应该距离更近一些。比如说`a`与`e`，就很容易就混淆。对于结尾的`t`与`te`的距离应该更近一些，而不是`1`。但是这些都还是自己想法，不容易实现。欢迎指点。
