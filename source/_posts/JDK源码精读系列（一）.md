# 手撕JDK系列（一）

最近有点焦躁，可能是大环境所影响，对未来有点焦虑，不想在未来的浪潮里被淘汰。

从毕业到现在工作了几年，虽然接触的知识及其广泛，变成语言从静态语言java到jvm上的groovy、Kotlin，甚至golang，到流行的动态语言python、javascript都有一定涉猎。从前端vue到数据库mysql、redis等有也用的比较多。可是进来却感觉一阵迷茫，不知道自己的核心竞争力是什么。痛定思痛，从项目中用的最多，却最不起眼的jdk开始从头再来。

## 首先需要认识下jdk中的3个包

rt.jar、tools.jar和dt.jar

dt.jar和tools.jar位于：{Java_Home}/lib/下，而rt.jar位于：{Java_Home}/jre/lib/下,其中：dt.jar是关于运行环境的类库; tools.jar是工具类库,编译和运行需要的都是toos.jar里面的类分别是sun.tools.java.*; sun.tols.javac.*;  在Classpath设置这几个变量，是为了方便在程序中 import；Web系统都用到tool.jar。

1. rt.jar
    rt.jar 默认就在Root Classloader的加载路径里面的，而在Claspath配置该变量是不需要的；同时jre/lib目录下的

    其他jar:jce.jar、jsse.jar、charsets.jar、resources.jar都在Root Classloader中

2. tools.jar

    tools.jar 是系统用来编译一个类的时候用到的，即执行javac的时候用到

    javac XXX.java

    实际上就是运行

    java -classpath=%JAVA_HOME%\lib\tools.jar xx.xxx.Main XXX.java

    javac就是对上面命令的封装 所以tools.jar 也不用加到classpath里面

3. dt.jar
    dt.jar是关于运行环境的类库,主要是swing的包   在用到swing时最好加上。
    
    也即是：
    
    > - rt.jar：Java基础类库，也就是Java doc里面看到的所有的类的class文件。
    > - tools.jar：是系统用来编译一个类的时候用到的，即执行javac的时候用到。
    > - dt.jar：dt.jar是关于运行环境的类库，主要是swing包。

## 2、rt.jar

从上面几个包的介绍来看，跟我们平常开发关系最紧密的就是rd.jar了，下图是在idea下的rt.jar

![InputStream](/img/io/1559031924992.png)

com包没什么可说的，是oracle和sun公司的两个子包，openJDK是不包含着部分的，跟我们这些coder来说，息息相关的就是java包和javax包了，由于javax的使用不是很常见，本文只讨论java包。

## 3、java包解析

以上图为切入点，applet和awt就不必说了，这是做桌面小程序所用到的，现在java的优势在于服务端的应用，我们从beans包开始。

### beans包：

![beans](/img/io/1559033786355.png)

从java doc里的描述来看， BeanContext充当javabean的逻辑层次容器。BeanContext提供与 bean 上下文有关的类和接口。 详细描述可以参考java api：<http://jszx-jxpt.cuit.edu.cn/JavaAPI/java/beans/beancontext/package-use.html/>

这个包下所有的类和接口基本都是围绕javabean和反射的。

举个简单的例子：

如果我们想获取一个类的私有变量，肯定都知道用反射，也通常都会这样：

```
 Object o = new InternetAddress();
        //第一种
        Class<?> aClass = o.getClass();
        Field[] fields = aClass.getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
        }
```

但是这样直接修改访问权限就违背了java封装的初衷了，也有一定的安全隐患。这个时候换下面这种方式看起来就舒服多了。并且在一定程度上解决了安全问题，很多框架也渐渐在采用这种方式来进行反射的调用。

```
 BeanInfo beanInfo = Introspector.getBeanInfo(InternetAddress.class);
        PropertyDescriptor[] descriptors = beanInfo.getPropertyDescriptors();
        for (PropertyDescriptor descriptor : descriptors) {
            Method readMethod = descriptor.getReadMethod();
            Method writeMethod = descriptor.getWriteMethod();
            Object value = readMethod.invoke(o);
            writeMethod.invoke(o, 1);
        }
```



### IO包

io是各个编程语言的核心之一，很多程序的吞吐和性能瓶颈就在于io这块，java的io体系更是及其繁杂。简单来说，可以分为输入I/输出O两部分，从两个基类InputStream、OutputStream。基于字符的Writer和Reader，基于磁盘的File，基于网络的Socket（不在java.io包下）都要基于上面两个基类。

引用网上的一张图：

<https://blog.csdn.net/zhangerqing/article/details/8466532/>

![InputStream](/img/io/1357309985_3442.png)

![OutputStream](img/io/1357310062_5816.png)

![Writer](/img/io/1357310154_6858.png)

![Reader](/img/io/1357310183_6062.png)

上一些代码来辅助使用说明：

附上最常用的IO操作

```
public static String read(String filename) throws Exception {
		BufferedReader br = new BufferedReader(new FileReader(filename));
		String s;
		StringBuffer sb = new StringBuffer();
		while ((s = br.readLine()) != null) {
			sb.append(s + "\n");
		}
		br.close();
		return sb.toString();
	}
```

好了，今天就到这里。明天继续！