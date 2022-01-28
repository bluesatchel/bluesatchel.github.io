---
title: java_ui
date: 2021-12-27 23:25:13
tags:
      - java
      - UI
categories: java
typora-root-url: ..
description: java折磨人的swing体验,都是我太菜呜呜
---

#### JFrame

JFrame初体验

```java
import javax.swing.*;
public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setSize(400,250);
        jf.setLocation(400,300);//setLocation设置出现在屏幕中的位置
        //setBounds()可以一次性完成上面两句
        jf.setVisible(true);//设置窗口可见
        jf.setTitle("hello");//设置标题
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);//设置点击关闭后关闭
        /*关闭方式参数
        DO_NOTHING_ON_CLOSE//什么也不做
        HIDE_ON_CLOSE//隐藏当前窗口
        DISPOSE_ON_CLOSE//隐藏当前窗口,并释放窗体占有的其他资源
        EXIT_ON_CLOSE//结束窗口缩在的应用程序 ...一般选这个...
    }
}

```

<img src="/images/java-ui/image-20211227233144855.png" alt="image-20211227233144855" style="zoom: 67%;" />

#### JDialog

继承自java.awt.Dialog类,他是从一个窗体弹出来的另外一个窗体,他和JFrame类似

`JDialog:可当成JFrame使用,但必须从属于JFrame`

构造函数 **一般用第三种**

```java
JDialog();
JDialog(Frame f);//指定父窗口
JDialog(Frame f,String title);//指定父窗口+标题
```

关闭方式只有三种,一般选第二种

DO_NOTHING_ON_CLOSE//什么也不做
HIDE_ON_CLOSE//隐藏当前窗口
DISPOSE_ON_CLOSE//隐藏当前窗口,并释放窗体占有的其他资源

例子

```java
import javax.swing.*;
public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setSize(400,250);
        //setLocation设置出现在屏幕中的位置
        jf.setLocation(400,300);
        jf.setVisible(true);//设置窗口可见
        jf.setTitle("hello");//设置标题
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);

        JDialog jd=new JDialog(jf,"jdialog");
        jd.setBounds(500,400,300,300);
        jd.setVisible(true);
        jd.setDefaultCloseOperation(WindowConstants.DISPOSE_ON_CLOSE);
    }
}

```



<img src="/images/java-ui/image-20211227234154224.png" alt="image-20211227234154224" style="zoom:67%;" />

关闭JFrame所有窗口都会关闭,关闭JDialogJFrame不受影响



#### 组件和面板

例子

组件一般添加到Jpanel和JScrollPane再添加到JFrame

JScrollPane是带滚动条的面板,只能添加一个组件,添加多个组件可以先添加到Jpanel再添加到JScrollPane

```java
import javax.swing.*;
import java.awt.*;

public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setBounds(400,300,400,250);
        jf.setVisible(true);//设置窗口可见
        jf.setTitle("hello");//设置标题
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        JButton jb1=new JButton("btn1");
        JButton jb2=new JButton("btn2");
        JPanel jp=new JPanel(new FlowLayout());
        jp.add(jb1);
        jp.add(jb2);
        jf.add(jp);

    }
}

```

<img src="/images/java-ui/image-20211227235300734.png" alt="image-20211227235300734" style="zoom:67%;" />

##### 标签组件JLabel

和C#中的label一样,类似于只读的文本框

JLabel构造方法

```java
以下参数都可以省略
JLabel(String str,Icon icon,int aligment);//设置文本,图标,水平对齐方式
```

##### 按钮组件JButton

JButton构造方法

```java
以下参数都可以省略
JButton(String text,Icon icon)//设置文本,图标
```

其他JButton类内自带方法

```java
setTooltipText(String text)//设置提示文本
setBorderPainted();//设置边界是否显示
setEnabled();//设置按钮知否可用
```

##### 单选按钮组件JRadioButton

JRadioButton构造方法

```java
以下参数都可以省略,一般只用指定文字即可
JRadioButton(String text,Icon icon,boolean selected)//指定文字,图标,是否选中
```

例子

```java
import javax.swing.*;
import java.awt.*;

public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setBounds(400,300,400,250);
        jf.setLayout(new FlowLayout());
        jf.setTitle("hello");//设置标题
        JRadioButton jrb1=new JRadioButton("男");
        JRadioButton jrb2=new JRadioButton("女");
        ButtonGroup group=new ButtonGroup();
        group.add(jrb1);
        group.add(jrb2);
        jf.add(jrb1);
        jf.add(jrb2);
        jf.setVisible(true);
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```

<img src="/images/java-ui/image-20211228001003287.png" alt="image-20211228001003287" style="zoom:67%;" />

##### 复选框组件JCheckBox

JCheckBox构造方法

```java
JCheckBox(Icon icon,boolean checked);//指定图标,是否被选中
JCheckBox(String text,boolean checked);//指定文字,是否被选中
```

例子

```java
import javax.swing.*;
import java.awt.*;

public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setBounds(400,300,400,250);
        jf.setLayout(new FlowLayout());//设置流式布局
        jf.setTitle("hello");//设置标题
        JCheckBox jcb1=new JCheckBox("摆烂",true);
        JCheckBox jcb2=new JCheckBox("足球");
        JCheckBox jcb3=new JCheckBox("音乐");
        JCheckBox jcb4=new JCheckBox("睡觉");
        jf.add(jcb1);
        jf.add(jcb2);
        jf.add(jcb3);
        jf.add(jcb4);
        
        jf.setVisible(true);
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```

<img src="/images/java-ui/image-20211228001621601.png" alt="image-20211228001621601" style="zoom:67%;" />

##### 下拉列表组件JComboBox

JComboBox构造方法

```java
JComboBox();//常用
JComboBox(ComboBoxModel dataModel);//使用listModel建立一个下拉列表
JComBox(Object[] arrayData);
JComboBox(Vector vector);
```

方法

`addItem添加下拉内容`

例子

```java
import javax.swing.*;
import java.awt.*;

public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setBounds(400,300,400,250);
        jf.setLayout(new FlowLayout());//设置流式布局
        jf.setTitle("hello");//设置标题
        JComboBox box=new JComboBox();
        box.addItem("---请选择你的学历---");
        box.addItem("高中");
        box.addItem("大学");
        box.addItem("研究生");
        jf.add(box);
        jf.setVisible(true);
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```

<img src="/images/java-ui/image-20211228131636660.png" alt="image-20211228131636660" style="zoom:67%;" />

##### 菜单栏组件

JmenuBar  Jmenu  JmenuItem

```java
import javax.swing.*;
import java.awt.*;

public class S9_3 {
    public static void main(String[] args) {
        JFrame jFrame=new JFrame("记事本");
        jFrame.setBounds(500,400,700,700);
        jFrame.setLayout(new FlowLayout(FlowLayout.LEFT));
        JMenuBar bar=new JMenuBar();
        JMenu menu1=new JMenu("文件(F)");
        JMenuItem item1=new JMenuItem("新建");
        JMenuItem item2=new JMenuItem("保存");
        JMenuItem item3=new JMenuItem("另存为");
        menu1.add(item1);
        menu1.add(item2);
        menu1.add(item3);
        JMenu menu2=new JMenu("编辑(E)");
        JMenu menu3=new JMenu("格式(O)");
        JMenu menu4=new JMenu("查看(V)");
        JMenu menu5=new JMenu("帮助(H)");
        JMenuItem item4=new JMenuItem("查看帮助(H)");
        JMenuItem item5=new JMenuItem("发送反馈(F)");
        JMenuItem item6=new JMenuItem("关于记事本(A)");
        menu5.add(item4);
        menu5.add(item5);
        menu5.add(item6);
        bar.add(menu1);
        bar.add(menu2);
        bar.add(menu3);
        bar.add(menu4);
        bar.add(menu5);


        jFrame.add(bar);
        JTextArea jTextArea=new JTextArea(40,63);
        jFrame.add(jTextArea);
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }

}
```

<img src="/images/java-ui/image-20211229002557142.png" alt="image-20211229002557142" style="zoom:67%;" />

##### 文本组件JTextField

构造方法

```java
以下参数都可以省略,一般只用指定文字即可
JTextField(Document docModel,String text,int fieldWidth);//指定文本框,文字,文本框长度
```

`getText();//获取输入的内容`

##### 密码框组件JPasswordField

构造方法

```java
以下参数都可以省略,一般只用指定文字即可
JPasswordField(Document docModel,String text,int fieldWidth);//指定文本框,文字,文本框长度
```

常用方法

```java
setEchoChar('*');//设置回显字符
getText();//获取输入的内容
```

例子

```java
import javax.swing.*;
import java.awt.*;

public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setBounds(400,300,400,250);
        jf.setLayout(new FlowLayout(FlowLayout.LEFT));//设置流式布局
        jf.setTitle("hello");//设置标题
        JLabel jlabel1=new JLabel("账号:");
        JTextField username=new JTextField(10);
        jf.add(jlabel1);
        jf.add(username);
        JLabel jlabel2=new JLabel("密码:");
        JPasswordField password=new JPasswordField(10);
        password.setEchoChar('*');//设置回显字符

        jf.add(jlabel2);

        jf.add(password);

        jf.setVisible(true);
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```

<img src="/images/java-ui/image-20211228133823317.png" alt="image-20211228133823317" style="zoom:67%;" />

##### 文本域组件JTextArea

构造函数

```java
JTextArea(Document doc,String text,int rows,int cols);//指定文本模型,默认文字,长,宽
```

常用方法

```java
setLineWrap();//设置文本域是否自动换行true or false
getText();//获取输入的内容
```

#### 常用布局

##### 流布局FlowLayout

所有组件像流一样,一个一个排放,排满一行之后排下一行,默认情况下,每个组件是居中排列的,但是也可以设置

构造方法

```java
//参数都可省略,一般只写对齐方式
FlowLayout(int aligment,int horizGap,int vertGap);//设置对齐方式,上下偏移
aligment取值:
FlowLayout.LEFT=0;
FlowLayout.CENTER=1;
FlowLayout.RIGHT=2;
```

##### 边界布局BorderLayout

边界布局是默认的布局管理方式,边界布局将容器分成了东`BorderLayout.NORTH`,南,西,北,中5个区域

在add的时候指定边界

`jf.add(button,BorderLayout.NORTH);`

<img src="/images/java-ui/image-20211228135034235.png" alt="image-20211228135034235" style="zoom:67%;" />

##### 网格布局GridLayout

将容器划分为网格,网格个数由行和列决定,每个组件会填满空格,改变容器大小,组件的大小也会随之改变

构造方法

```java
GridLayout(int rows,int columns,int horizGap,int vertGap);//指定行数,列数,水平间隔.垂直间隔
```

例子

```java
import javax.swing.*;
import java.awt.*;

public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setBounds(400,300,400,250);
        jf.setLayout(new GridLayout(3,3,5,5));//设置网格布局
        jf.setTitle("test");//设置标题
        JButton jButton1=new JButton("A");
        JButton jButton2=new JButton("B");
        JButton jButton3=new JButton("C");
        JButton jButton4=new JButton("D");
        JButton jButton5=new JButton("E");
        JButton jButton6=new JButton("F");
        JButton jButton7=new JButton("G");
        JButton jButton8=new JButton("H");
        jf.add(jButton1);
        jf.add(jButton2);
        jf.add(jButton3);
        jf.add(jButton4);
        jf.add(jButton5);
        jf.add(jButton6);
        jf.add(jButton7);
        jf.add(jButton8);
        jf.setVisible(true);
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```

<img src="/images/java-ui/image-20211228135556932.png" alt="image-20211228135556932" style="zoom:67%;" />

#### 常用的事件监听器

一个事件模型中有三个对象:事件源,事件,以及监听程序

##### 事件监听机制

- 事件源    事件发生的地方
- 事件        要发生的事情
- 事件处理 针对发生的事情做出的处理方案
- 事件监听 把事件源和事件关联起来

##### 两种监听器

|                | 事件        | 事件源                     | 监听接口       | 方法                                       |
| -------------- | ----------- | -------------------------- | -------------- | ------------------------------------------ |
| 动作事件监听器 | ActionEvent | JButton,JlistmJTextField等 | ActionListener | addActionListener(),removeActionListener() |
| 焦点事件监听器 | FocusEvent  | Component及其派生          | FocusListener  | addFocusListener(),removeFocusListener()   |

使用步骤

1. 新建一个组件(如Button)
2. 将该组件添加到相应的面板(如JFrame)
3. 注册监听器以监听事件源产生的事件(如通过ActionListener来响应用户点击按钮)
4. 定义处理事件的方法(如在ActionListener中的actionPerformed中定义响应的方法)

**一般用匿名内部类方式实现**

例子

```java
jla.addFocusListener(new FocusListener(){
    public void focusLost(FocusEvent e){
        //失去焦点的时候做的事情
    }
    public void focusGained(FocusEvent e){
        //获取焦点的时候做的事情
    }  
});
```

```java
jb.addActionListener(new ActionListener(){
	@Override
    public void actionPerformed(ActionEvent e){
     //单击鼠标时执行   
    }
});
```

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;

public class jFrameTest {
    public static void main(String[] args) {
        JFrame jf=new JFrame();
        jf.setBounds(400,300,400,250);
        jf.setLayout(new FlowLayout());//设置网格布局
        jf.setTitle("test");//设置标题
        JTextArea jTextArea=new JTextArea();
        jTextArea.setLineWrap(true);
        JButton jButton=new JButton("希望你对你对人生也是这个态度");
        jf.add(jTextArea);
        jf.add(jButton);
        jButton.addActionListener(new AbstractAction() {
            @Override
            public void actionPerformed(ActionEvent e) {
                jTextArea.append("啊对对对");
            }
        });

        jf.setVisible(true);
        jf.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```

<img src="/images/java-ui/image-20211228141558056.png" alt="image-20211228141558056" style="zoom:67%;" />

##### 综合实践

```java
import javax.swing.*;
import javax.swing.filechooser.FileNameExtensionFilter;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileReader;
import java.io.FileWriter;

public class S9_2 {
    static String path;
    public static void readText(String path,Object obj) throws Exception{
        JTextArea jTextArea=(JTextArea) obj;
        FileReader fileReader=new FileReader(path);
        BufferedReader bufferedReader=new BufferedReader(fileReader);

        String line;
        while((line=bufferedReader.readLine())!=null){
            jTextArea.append(line);
        }
        bufferedReader.close();
        fileReader.close();
    }
    public static void saveText(String path,Object obj)throws Exception{
        JTextArea jTextArea=(JTextArea) obj;
        BufferedWriter bufferedWriter=new BufferedWriter(new FileWriter(path));
        bufferedWriter.write(jTextArea.getText());
        bufferedWriter.flush();
        bufferedWriter.close();
    }
    public static void main(String[] args) {
        JFrame jFrame=new JFrame("S9_2");
        jFrame.setLayout(new FlowLayout(FlowLayout.CENTER));
        jFrame.setBounds(500,500,600,500);
        JTextArea jTextArea=new JTextArea(20,50);
        JTextField jTextField=new JTextField(35);
        //把文本域放进jScrollPane让其拥有滚动条
        JScrollPane jScrollPane=new JScrollPane(jTextArea);
        JLabel jLabel=new JLabel("File:");
        JButton Browse=new JButton("Browse");
        Browse.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                
             	//文件选择组件
                JFileChooser chooser =new JFileChooser("C:/");
                FileNameExtensionFilter filter = new FileNameExtensionFilter(
                        "文本文档", "txt");
                chooser.setFileFilter(filter);
                int returnVal = chooser.showOpenDialog(jFrame);
                if(returnVal == JFileChooser.APPROVE_OPTION) {
                    path=chooser.getSelectedFile().getAbsolutePath();
                    jTextField.setText(path);
                }
            }
        });
        JButton Clear=new JButton("Clear");
        Clear.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                jTextArea.setText("");
            }
        });
        JButton Read=new JButton("Read");
        Read.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {

                try{
                    readText(path,jTextArea);
                }catch (Exception ex){

                }
            }
        });
        JButton Save=new JButton("Save");
        Save.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {

                try{
                    saveText(path,jTextArea);
                }catch (Exception ex){

                }
            }
        });
        JButton Exit=new JButton("Exit");
        Exit.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.exit(0);
            }
        });
        //设置滚动条一直显示
        jScrollPane.setVerticalScrollBarPolicy(ScrollPaneConstants.VERTICAL_SCROLLBAR_ALWAYS);
        jFrame.add(jLabel);
        jFrame.add(jTextField);
        jFrame.add(Browse);
        jFrame.add(jScrollPane);
        jFrame.add(Clear);
        jFrame.add(Read);
        jFrame.add(Save);
        jFrame.add(Exit);
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
    }
}

```

<img src="/images/java-ui/image-20211229002852293.png" alt="image-20211229002852293" style="zoom:67%;" />
