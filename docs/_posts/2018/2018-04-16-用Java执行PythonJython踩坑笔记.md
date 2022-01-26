---
layout: post
title: 用Java执行Python：Jython踩坑笔记
date: 2018-04-16 17:39:00
author: 薛师兄
tags:
  - Java
  - Python
---
### 常见的java调用python脚本方式

**1.通过Jython.jar提供的类库实现** 
**2.通过Runtime.getRuntime()开启进程来执行脚本文件**

---

# 1.Jython

Jpython使用时，版本很重要！大多数坑来源于此。这句话不听的人还得走点弯路

**运行环境：Python2.7 + Jython-standalone-2.7.0**

```xml
<!--Maven依赖，jar包自行前往仓库下载-->
<dependency>
    <groupId>org.python</groupId>
    <artifactId>jython-standalone</artifactId>
    <version>2.7.0</version>
</dependency>
```

**1）Jython执行Python语句**

```java
import org.python.util.PythonInterpreter;

public class HelloPython {
    public static void main(String[] args) {
        PythonInterpreter interpreter = new PythonInterpreter();
        interpreter.exec("print('hello')");
    }
}
```

**2）Jython执行Python脚本**

```java
import org.python.util.PythonInterpreter;

public class HelloPython {
    public static void main(String[] args) {
        PythonInterpreter interpreter = new PythonInterpreter();
        interpreter.execfile("./pythonSrc/time.py");
    }
}
```

**3）Jython执行Python方法获取返回值**

```py
PythonInterpreter interpreter = new PythonInterpreter();
interpreter = new PythonInterpreter(); 
interpreter.execfile("./pythonSrc/fibo.py"); 
PyFunction function = (PyFunction)interpreter.get("fib",PyFunction.class); 
PyObject o = function.__call__(new PyInteger(8));
System.out.println(o.toString());
```

fibo.py

```py
# Fibonacci numbers module  
ef fib(n): # return Fibonacci series up to n  
   result = []  
   a, b = 0, 1  
   while b < n:  
       result.append(b)  
       a, b = b, a+b  &middot;
   return result 
```

---

# 2.Jython的局限

Jython在执行普通py脚本时速度很慢，而且在含有第三方库（requests, jieba&hellip;）时bug很多,不易处理。 原因在于，python执行时的sys.path和Jython的sys.path路径不一致，以及Jython的处理不是很好。

**python执行时：**

```java
<span class="hljs-special">[</span>'F:<span class="hljs-command">\\</span>Eclipse for Java EE<span class="hljs-command">\\</span>workspace<span class="hljs-command">\\</span>Jython<span class="hljs-command">\\</span>pythonSrc', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>DLLs', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>lib-tk', 'F:<span class="hljs-command">\\</span>Python27', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>site-packages', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>site-packages<span class="hljs-command">\\</span>unknown-0.0.0-py2.7.egg', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>site-packages<span class="hljs-command">\\</span>requests-2.18.4-py2.7.egg', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>site-packages<span class="hljs-command">\\</span>certifi-2018.1.18-py2.7.egg', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>site-packages<span class="hljs-command">\\</span>urllib3-1.22-py2.7.egg', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>site-packages<span class="hljs-command">\\</span>idna-2.6-py2.7.egg', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>site-packages<span class="hljs-command">\\</span>chardet-3.0.4-py2.7.egg', 'C:<span class="hljs-command">\\</span>windows<span class="hljs-command">\\</span>system32<span class="hljs-command">\\</span>python27.zip', 'F:<span class="hljs-command">\\</span>Python27<span class="hljs-command">\\</span>lib<span class="hljs-command">\\</span>plat-win'<span class="hljs-special">]</span>
```

**Jython 执行时：**

```java
<span class="hljs-special">[</span>'F:<span class="hljs-command">\\</span>Maven<span class="hljs-command">\\</span>repo<span class="hljs-command">\\</span>org<span class="hljs-command">\\</span>python<span class="hljs-command">\\</span>jython-standalone<span class="hljs-command">\\</span>2.7.0<span class="hljs-command">\\</span>Lib', 'F:<span class="hljs-command">\\</span>Maven<span class="hljs-command">\\</span>repo<span class="hljs-command">\\</span>org<span class="hljs-command">\\</span>python<span class="hljs-command">\\</span>jython-standalone<span class="hljs-command">\\</span>2.7.0<span class="hljs-command">\\</span>jython-standalone-2.7.0.jar<span class="hljs-command">\\</span>Lib', '__classpath__', '__pyclasspath__/'<span class="hljs-special">]</span>
```

关于路径问题，我们有两种解决方法，**一是手动添加第三方库路径**，调用

```java
PySystemState sys = Py.getSystemState(); 
System.out.println(sys.path.toString());
sys.path.add("F:\\Python27\\Lib\\site-packages\\jieba"); 
```

**二是把第三方库文件夹放到执行的.py脚本同级目录**。 
然后新的问题来了，你以为路径是最终的问题吗？不止，也许是Python的版本语法问题，2x，3x，导致你在用Jython执行含第三方库的.py脚本时，各种Module不存在。原本博主走的Jython的路子，还下载了jieba第三方库，后来运行时一大堆错误：jieba库好像对py3有过渡支持，jython不支持这种语法格式，我改了jieba一处又一处，所以，博主在受过摧残之后果断放弃Jython！因为不能使用第三方库的python，然并卵！

## 最终方法来了：模拟控制台执行

```java
public class Cmd {

    public static void main(String[] args) throws IOException, InterruptedException {
        String[] arguments = new String[] { "python", "./pythonSrc/time.py", "huzhiwei", "25" };
        try {
            Process process = Runtime.getRuntime().exec(arguments);
            BufferedReader in = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line = null;
            while ((line = in.readLine()) != null) {
                System.out.println(line);
            }
            in.close();
            int re = process.waitFor();
            System.out.println(re);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

time.py

```java
#!/usr/bin/python
#coding=utf-8

#定义一个方法
def my_test(name, age):
    print("name: "+str(name))
    print(age)  #str()防解码出错
    return "success"
#主程序
#sys.argv[1]获取cmd输入的参数
my_test(sys.argv[1], sys.argv[2])
```

执行结果

```java
name: huzhiwei
25
0
```

再唠两块钱的~ 
这个方法的局限性也来了，对python开发人员很简单，直接打印输出，但一个python模块只能做一件事，这点很像Python端给Java端的一个公开接口，类似Servlet吧？好处也有，不会出错，运行快！还在犹豫的同学赶快转cmd吧~！

---

# 2018-4-19：

博主在模拟cmd调用Python时遇到一些情况，这类问题可以归类为&ldquo;超时，阻塞&rdquo;等 
问题原因：

```java
Process p=Runtime.getRuntime().exec(String[] cmd);
```

Runtime.exec方法将产生一个本地的进程,并返回一个Process子类的实例,该实例可用于控制进程或取得进程的相关信息。 由于调用Runtime.exec方法所创建的子进程没有自己的终端或控制台,因此该子进程的标准IO(如stdin,stdou,stderr)都通过 p.getOutputStream(), p.getInputStream(), p.getErrorStream() 方法重定向给它的父进程了.用户需要用这些stream来向 子进程输入数据或获取子进程的输出。

例如:Runtime.getRuntime().exec("ls") 另外需要关心的是Runtime.getRuntime().exec()中产生停滞（阻塞，blocking）的问题？ 这个是因为Runtime.getRuntime().exec()要自己去处理stdout和stderr的输出， 就是说，执行的结果不知道是现有错误输出（stderr），还是现有标准输出（stdout）。 你无法判断到底那个先输出，所以可能无法读取输出，而一直阻塞。 例如：你先处理标准输出（stdout），但是处理的结果是先有错误输出（stderr）， 一直在等错误输出（stderr）被取走了，才到标准输出（stdout），这样就产生了阻塞。

解决办法：

用两个线程将标准输出（stdout）和错误输出（stderr）。

完整代码：

```java
import java.io.BufferedInputStream;
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;

/**
 * Created by Martin 2018年4月19日 下午1:50:06
 */
public class ExecuteCmd {
    /** 执行外部程序,并获取标准输出 */
    public static String execute(String[] cmd, String... encoding) {
        BufferedReader bReader = null;
        InputStreamReader sReader = null;
        try {
            Process p = Runtime.getRuntime().exec(cmd);

            /* 为"错误输出流"单独开一个线程读取之,否则会造成标准输出流的阻塞 */
            Thread t = new Thread(new InputStreamRunnable(p.getErrorStream(), "ErrorStream"));
            t.start();

            /* "标准输出流"就在当前方法中读取 */
            BufferedInputStream bis = new BufferedInputStream(p.getInputStream());

            if (encoding != null && encoding.length != 0) {
                sReader = new InputStreamReader(bis, encoding[0]);// 设置编码方式
            } else {
                sReader = new InputStreamReader(bis, "utf-8");
            }
            bReader = new BufferedReader(sReader);

            StringBuilder sb = new StringBuilder();
            String line;

            while ((line = bReader.readLine()) != null) {
                sb.append(line);
                sb.append("\n");
            }

            bReader.close();
            p.destroy();
            return sb.toString();
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}

class InputStreamRunnable implements Runnable {
    BufferedReader bReader = null;

    public InputStreamRunnable(InputStream is, String _type) {
        try {
            bReader = new BufferedReader(new InputStreamReader(new BufferedInputStream(is), "UTF-8"));
        } catch (Exception ex) {<br />　　　　　　ex.printStackTrace();
        }
    }

  
    public void run() {
        String line;
        int num = 0;
        try {
            while ((line = bReader.readLine()) != null) {
                System.out.println("---->"+String.format("%02d",num++)+" "+line);
            }
            bReader.close();
        } catch (Exception ex) {<br />　　　　　　ex.printStackTrace();
        }
    }
}
```

使用时直接调用该工具类即可。