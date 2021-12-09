---
title: java正则表达式
date: 2021-12-06 21:43:24
tags:
    - java
    - 正则表达式
categories: java
description: java正则
typora-root-url: ..
---

首先感谢[韩顺平老师的课程](https://www.bilibili.com/video/BV1Eq4y1E79W?p=1&spm_id_from=pageDriver)!!

#### 正则匹配中贪婪模式和非贪婪模式的区别

我个人理解为,贪婪模式情况下,会尽可能多的进行每一次的匹配,非贪婪模式每一次进行匹配都会找尽可能短的满足`?`前条件的字符完成本次匹配

#### java正则简单实现

简单实践,后面记录各个类和方法的信息

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
public class r1 {

    public static void main(String[] args) {
        String content="2000年5月，JDK1.3、JDK1.4和J2SE1.3相继发布，几周后其获得了Apple公司Mac OS X的工业标准的支持。2001年9月24日，J2EE1.3发布。" +
                "2002年2月26日，J2SE1.4发布。自此Java的计算能力有了大幅提升，与J2SE1.3相比，其多了近62%的类和接口。在这些新特性当中，还提供了广泛的XML支持、安全套接字（Socket）支持（通过SSL与TLS协议）、全新的I/OAPI、正则表达式、日志与断言。" +
                "2004年9月30日，J2SE1.5发布，成为Java语言发展史上的又一里程碑。为了表示该版本的重要性，J2SE 1.5更名为Java SE 5.0（内部版本号1.5.0），" +
                "代号为“Tiger”，Tiger包含了从1996年发布1.0版本以来的最重大的更新，其中包括泛型支持、基本类型的自动装箱、改进的循环、枚举类型、" +
                "格式化I/O及可变参数。";
		//匹配数字或者单词
        Pattern pattern = Pattern.compile("\\d\\d\\d\\d");
        //2.创建一个匹配器对象
        Matcher matcher= pattern.matcher(content);
        //可以循环匹配
        while(matcher.find()){
            //匹配文本放到matcher.group(0)
            System.out.println("找到"+matcher.group(0));
        }
    }
}

```

##### `matcher.find()`完成的任务:

1.根据指定的规则定位满足规则的子字符串(比如2000)

2.找到后,将子字符串的开始索引记录到matcher对象的属性int[] groups;

![image-20211207000454930](/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211207000454930.png)

![](/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211207000542796.png)

groups[0]=0,把该子字符串的结束的索引+1的值记录到groups[1]=4

3.同时记录oldLast值为子字符串的结束的索引+1,即下次执行find方法时就从记录的索引位置4开始匹配

group()源码

```java
public String group(int group) {
        if (first < 0)
            throw new IllegalStateException("No match found");
        if (group < 0 || group > groupCount())
            throw new IndexOutOfBoundsException("No group " + group);
        if ((groups[group*2] == -1) || (groups[group*2+1] == -1))
            return null;
        return getSubSequence(groups[group * 2], groups[group * 2 + 1]).toString();//截取字符串
    }
```

1.根据groups[0]和groups[1]的记录位置,从content开始截取,从content开始截取字符串返回就是[0,4),包含0但是不包含索引为4的位置

如果再次执行find方法,仍然按照上面的方法,groups[0]和groups[1]记录本次的开始和结束位置,group[0]记录的是下次!![](/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211207001258292.png)

`Pattern pattern = Pattern.compile("(\\d\\d)(\\d\\d)");`加上括号,分组匹配:

根据指定的规则，定位满足规则的子字符串(比如(20)(00))

1.找到后将 子字符串的开始索引 group[0]=0 记录到 matcher 对象的熟悉 int[] groups数组中；
2.1 groups[0] = 0, 把该子字符串的结束的索引+1的值记录到 groups[1] = 4
2.2 记录1组()匹配到的子字符串 groups[2] = 0 groups[3] = 2
2.3 记录2组()匹配到的子字符串 groups[4] = 2 groups[5] = 4

![](/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211207004146025.png)

2.4 如果有更多的分组，同理

```java
while (matcher.find()) {

            System.out.println("找到：" + matcher.group(0)); // 2000
            System.out.println("第一组匹配到的值: " + matcher.group(1)); // 2
            System.out.println("第二组匹配到的值: " + matcher.group(2)); // 000
            // System.out.println("找到：" + matcher.group(3)); 索引越界
        }
```

<img src="/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211207004834793.png" style="zoom:80%;" />

3.同时记录 oldLast 的值为 子字符串的结束的 索引+1的值即69，即下次执行find时，就从69开始匹配。

**我的理解是,group[0]永远获取的是每次整个表达式匹配的结果,group(1)和group(2)则是每次匹配到的括号内的内容**



##### 转义字符的使用

注:在java的正则表达式中,两个\\\\代表一个\

```java
public class Demo {
    public static void main(String[] args) {
        String content = "abc$(a.bc(123(";

        // 俩个 \\ 字符表示 \
        Pattern compile = Pattern.compile("\\(");
        // 2.创建一个匹配器对象
        Matcher matcher = compile.matcher(content);
        // 3. 可以循环匹配
        while (matcher.find()) {
            // 匹配内容，文本，放到 m.group(0)
            System.out.println("找到：" + matcher.group(0));
        }
    }
}
```

#### 区分大小写

java正则表达式默认是区分大小写的,如何实现不区分大小写

- (?i)abc      表示abc都不区分大小写
- a(?i)bc       表示bc不区分大小写
- a((?i)b)c     表示只有b不区分大小写
- `Pattern pat  = Pattern.compile(regEx,pattern.CASE_INSENSITIVE);`开启不区分大小写的匹配,整个regEx都不区分大小写



#### 关于[ ]的一些注意点

^在[ ]内表示非

[ ]内的任意字符之间都是或的关系

[ ]内的符号就只是一个符号了,不带特殊功能    **注意:在( )内要加转义符**

#### 限定符

| 示例        | 说明                                                        |
| ----------- | ----------------------------------------------------------- |
| (abc)*      | 仅包含任意个abc的字符串                                     |
| m+(abc)*    | 以m开头,后接任意个abc的字符串                               |
| m+abc?      | 以m开头,后接ab或者abc的字符串                               |
| [abcd]{3}   | 由abcd中字母组成的任意长度为3的字符串                       |
| [abcd]{3,}  | 由abcd中字母组成的任意长度不小于3的字符串                   |
| [abcd]{3,5} | 由abcd中字母组成的任意长度不小于3,不大于5的字符串(左闭右开) |

java匹配默认贪婪匹配



#### 定位符

定位符,规定要匹配的字符串出现的位置,比如在字符串的开始还是在结束的位置

| 符号 | 含义                   | 示例             | 说明                                                       |
| ---- | ---------------------- | ---------------- | ---------------------------------------------------------- |
| ^    | 指定起始字符           | ^[0-9]+[a-z]*    | 以至少1个数字开头,后接任意个小写字母的字符串               |
| $    | 指定结束字符           | ^[0-9]\\-[a-z]+$ | 以1个数字开头后接连字符"-",并以至少1个小写字母结尾的字符串 |
| \b   | 匹配目标字符串的边界   | er\\b            | 匹配forev`er`中的er,但不匹配verb中的er                     |
| \B   | 匹配目标字符串的非边界 | er\\B            | 匹配v`er`b中的er,但不匹配forever中的er                     |

#### 分组

##### (pattern)

非命名捕获,捕获匹配的子字符串,编号为0的第一个捕获是由整个正则表达式模式匹配的文本(即group(0)),其它捕获结果则根据左括号的顺序从1开始自动编号

##### (?\<name\>)

命名捕获,将匹配的字符串捕获到一个组名称或编号名称中,用于name的字符串不能包含任何表单符号,并且不能以数字开头,可以使用单引号替代尖括号,例如(?'name')

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
public class r1 {
    public static void main(String[] args) {
        String content="dsfdsfds s7789 nn1189han";
//      String regStr="^[0-9]+[a-z]+\\d+$";
        String regStr="(?<g1>\\d\\d)(?<g2>\\d\\d)";
        Pattern pattern=Pattern.compile(regStr);
        //创建一个匹配器对象
        Matcher matcher= pattern.matcher(content);
        while (matcher.find()) {
            System.out.println("找到：" + matcher.group(0)); // 2000
            System.out.println("分组1(根据组名): "+matcher.group("g1"));
            System.out.println("分组1: "+matcher.group(1));
            System.out.println("分组2(根据组名): "+matcher.group("g2"));
            System.out.println("分组2: "+matcher.group(2));
        }
    }
}
```

![](/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211207234020782.png)

#### 非捕获分组

##### (?:pattern)

非捕获分组不算入分组内,即不能通过group(i)去访问

```java
import java.util.regex.Matcher;
import java.util.regex.Pattern;
public class r1 {
    public static void main(String[] args) {
        String content="124张三男士  hell张三先生   张三同学  ";
//      String regStr="^[0-9]+[a-z]+\\d+$";
        String regStr="张三(?:男士|先生|同学)";//等价于:张三男士|张三先生|张三同学
        Pattern pattern=Pattern.compile(regStr);
        //创建一个匹配器对象
        Matcher matcher= pattern.matcher(content);
        while (matcher.find()) {
            System.out.println("找到：" + matcher.group(0)); // 2000
        }
    }
}

```

![](/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211208235406173.png)

##### (?=pattern)

它是一个非捕获匹配,例如,"windows(?=95|98|NT|2000)"匹配"windows2000"中的"windows",但不匹配"windows10"中的windows

(理解这个等号的含义即可)

##### (?!pattern)

与上一个刚好相反,例如,"windows(?=95|98|NT|2000)"匹配"windows10"中的"windows",但不匹配"windows2000"中的windows

(理解!与=的相反含义)

#### 匹配汉字

`resStr = "^[\u0391-\uffe5]+$"`

上面的regStr匹配纯汉字

\u0391-\uffe5是汉字的编码范围

一些练习:

匹配1-9开头的六位数:  `^[1-9]\\d{5}$`

匹配必须以13,14,15,18开头的11位数: `^1[3458]\\d{9}$`

匹配小数或者整数 `^[-+]?([1-9]\\d*|0)(\\.\\d+)?$`





### 三个常用类

#### Pattern类

Pattern对象是一个正则表达式对象,Pattern类没有公共构造方法,(不需要new)要创建一个Pattern对象,调用其公共静态方法,它返回一个Pattern对象,该方法接受一个正则表达式作为它的第一个参数,比如

```java
Pattern p=Pattern.compile(regStr);
```

##### matches方法

如果只是判断是否满足格式,可以使用Pattern的matches,整体匹配,比较简洁,返回一个bool值,但是只能整体匹配,不能分组

`Pattern.matches(regStr,content)`

### Matcher类

Matcher类是对输入字符进行解释和匹配的引擎,与Pattern类一样,Matcher也没有公共构造方法,需要调用Pattern对象的matcher方法来获得一个Matcher对象,比如

```java
Matcher matcher= pattern.matcher(content);
```

##### matcher.start();获取find每一组开始的索引

##### matcher.end();获取find每一组尾部的索引

##### replaceAll();//替换,注意该函数只是返回替换后的结果,并不修改原来conetent

```
public class r1 {
    public static void main(String[] args) {
        String content="我确实是个傻缺";
//      String regStr="^[0-9]+[a-z]+\\d+$";
        String regStr="确实";
        Pattern pattern=Pattern.compile(regStr);
        //创建一个匹配器对象
        Matcher matcher= pattern.matcher(content);
        while (matcher.find()) {
            String newContent=matcher.replaceAll("雀氏");
            System.out.println(newContent);
        }
    }
}
```



### PatternSyntaxException类

PatternSyntaxException是一个非强制异常类,它表示一个正则表达式中的语法错误





#### 分组,捕获,反向引用

1.分组

我们可以用圆括号组成一个比较复杂的匹配模式,那么一个圆括号的部分我们可以看作是一个子表达式(一个分组)

2.捕获

把正则表达式中子表达式/分组匹配的内容,保存澡内存中以数字编号或显式命名的组里,方便后面引用,从左往右,以分组的左括号为标志,第一个为1,0代表整个表达式匹配内容

3.**反向引用**

圆括号的内容被捕获偶,可以在这个括号后被引用,从而写出一个比较使用的匹配模式,这个称为反向引用,这种引用既可以是在正则表达式内部,<u>也可以是在正则表达式外部,内部反向引用用`\\分组号`,外部反向引用`$分组号`</u>

例子,匹配回文数

```java
String regStr="(\\d)(\\d)\\2\\1";
```

---应用示例:

结巴程序:把 类似: "我....我要....学学学学....编程java!"   通过正则表达式修改为"我要学编程java!"

思路:

1.去掉所有的点

2.去除所有重复字

```java
public static void main(String[] args) {
        String content="我....我要....学学学学....编程java!";
        //1.去掉所有的点
        String regStr="\\.";
        Pattern pattern=Pattern.compile(regStr);
        Matcher matcher= pattern.matcher(content);
        content=matcher.replaceAll("");
        System.out.println(content);
        //2去除所有重复字
        regStr="(.)\\1+";//只会匹配到1-多的字符串
        pattern=Pattern.compile(regStr);
        matcher=pattern.matcher((content));
        content=matcher.replaceAll("$1");//分组的表达式外引用
        System.out.println(content);
    	//第2步的代码也可以用一行代码替换
        /*content=Pattern.compile("(.)\\1+").matcher(content).replaceAll("$1");
        System.out.println(content);
        */
    
    }
```



#### String类中使用正则表达式

String类中可以直接使用

1. replaceAll()方法

2. matches()方法

3. 分割功能    split()方法中可以使用正则表达式

   ```java
   public class r1 {
       public static void main(String[] args) {
           String content="hello#abc-jack12smith~河南";
           //要求按照#或-或~或数字来分割
           String[]spilt = content.split("#|~|-|\\d+");
           for(String s : spilt){//类似于foreach
               System.out.println(s);
           }
       }
   }
   ```

   ![](/images/java%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F/image-20211209124141204.png)

#### 写正则表达式思路

1.先写出一个简单的表达式,接着根据各种情况来逐渐完善

2.如果遇到需要不匹配的内容,可以在()外对其进行匹配,并可以用标志性符号对其设置边界,比如url中的/





