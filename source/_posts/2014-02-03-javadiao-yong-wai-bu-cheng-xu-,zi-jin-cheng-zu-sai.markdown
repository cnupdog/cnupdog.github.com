---
layout: post
title: "JAVA调用外部程序，子进程阻塞"
date: 2014-02-03 18:49:23 +0800
comments: true
categories: 
---
    使用JAVA的Runtime对象通过exec(cmd)调用远程脚本，随机出现进程hand住的情况。

标准输入输出
--------------
标准流是当一个程序运行后，在程序和它的环境（典型终端），事先连接的输入和输出管道。当然管道的特点是单向的，管道把一个进程的输出传给另一个进程作为输入。  
> 在unix环境下执行一个shell命令行通常会自动打开三个标准文件：  
>> **标准输入文件```stdin```:**对应终端的键盘终端     
>> **标准输出文件```stdout```:**对应终端的屏幕     
>> **标准错误输出文件```stderr```:**对应终端的屏幕    

同步异步、阻塞非阻塞
--------------
在通信和I/O中常会提到的两组术语，非常容易混淆，认为他们是一个概念。  
同步和异步是针对信息的发送和接收次序而言，和**消息的通知机制**有关。  
阻塞和非阻塞是针对进程、线程进行等待时的一种方式（是否需要挂起，是否影响线程的执行，是否可以干其他事情），和**程序等待消息时的状态**有关。  
所以它们能有如下四种组合：**同步阻塞**、 **同步非阻塞**、 **异步阻塞**、 **异步非阻塞**。  
> **同步```synchronous```:**发送消息，等待消息处理完后，才往下执行       
> **异步```asynchronous```:**发送消息，不等待消息处理完，就往下执行，让后通过特定的接口或者事件，消息通知你事情完成了  
> **阻塞```blocking```:**程序会阻塞在某一个函数，而不往下执行，就像挂在那里一样，所有的其他业务也都不执行，一直等到有消息到来才往下执行    
> **非阻塞```non-blocking```:**程序不会阻塞在某一个函数，不等待消息到来，往下执行

Process阻塞分析
--------------
**[场景描述]**  现在有很多项目均是前后台结合，前台使用JAVA，后台使用perl或者python等编写的脚本。前台触发执行动作，然后将操作发送到远程，即执行远程脚本。脚本执行完之后将操作结果返回JAVA，再由其做处理。
以一个执行perl的伪代码描述如下：
    
    String[] cmd = {"/bin/sh","-c",""};//前两个为执行perl必要参数
    cmd2[] = "some comond line";//命令行内容
    Process p = Runtime.getRuntime.exec(cmd);//调用底层linux脚本
    //将标准io(这里特指标准输出)通过getInputStream流重定向到父亲进程
    BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));
    String line = null;
    while((line = br.readLine())!=null){
        //do something
    }
     p.waitFor();//控制让当前线程等待，直到Proccess对象表示的进程结束

**[原因分析]**  

    BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()));

**p.getInputStream()**是获得子进程**远程Perl程序**的输出流（子进程的输出流即父进程的输入流），这里的输出流是**字节流**  
**InputStreamReader()**是从字节流到字符流的桥梁，是InputStream和Reader的桥梁    
**BufferedReader**：public class BufferedReader extends Reader，是从**字符输入流**读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取。    

上面的概念均是为后面分析做铺垫，如果**远程Perl程序**的输出流很多，把缓冲区的空间都占满了。想写的写不进，想读的读不去，那么程序将由于waitFor被hang住。举例如：
    
    BufferedReader br = new BufferedReader(new InputStreamReader(p.getErrorStream())); 
这里获取子进程的标准错误输出流，但jvm一直没有读取，缓冲区满了后就无法继续写入数据。错误输出流没有数据要输出，标准输出流有数据要输出，但由于标准输出流没有被读取，进程就不会结束，错误输出流也不会关闭，于是在执行到**readLine()**方法，这个程序就会被阻塞。

**[解决方法]**  

1. <s>如果你知道程序的一个执行顺序，清楚输出 标准输出错误流、标准输出流的先后顺序。例如先读取标准输出错误流，在读取标准输出流     
>  缺点：很多时候你不知道输出的先后顺序，该方法相当没用。。</s>  
2. <s>可以在远程程序，及时flush缓冲        
>  缺点: 不一定能更改远程程序，有点另类</s>  
3. <s>使用ProcessBuilder类，利用**redirectErrorStream**将标准输出流和错误输出流合二为一。      
>  缺点: 没有实际使用过</s>  
4. **推荐**并发获取Process的输出流和错误流，即启用两个线程来并发读取和处理输出流和错误流      
>  用一个线程读取process.getInputStream()的输出流，另外一个线程来读取process.getErrorStream()的输出流，这样就能保证缓冲区得到及时的清空而不用担心线程被阻塞了

--EOF--


    

