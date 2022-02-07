---
layout:      post
title:       "进程与线程"
subtitle:    "  ————进程,线程基础以及同步与互斥"
date:        2021-12-02
author:      "Hywel"
header-img:  "img/post-bg-multithreading.jpg" 
tags:
   -OperatingSystem
   -Process
   -Thread
---
### 进程 ###
  进程拥有一个地址空间和一个控制线程,每个进程的地址空间相对独立.进程还会有一个线程表来调度和管理自己的线程. 

### 线程 ### 
* **用户态线程:** 将整个线程包放在用户空间.内核不知道线程存在
  + 优点:
    - 1.用户级线程可以在不支持线程的 操作系统上实现
    - 2.比陷入内核要快一个数量级以上
    - 3.允许每个进程有自己定制的线程调度算法
  + 问题&解决办法:
    - Q1. 如何解决线程的系统阻塞会阻塞整个进程?
    - a1. 进行select调用查看当前系统调用是否会导致阻塞,若会则调用另一线程,直到检测安全后才进行有关的系统调用.
        这种方法需要重写系统调用库,所以效率不高.进行外部检查的这类代码被称为包装器.
    - Q2. 线程引发的页面故障而不阻塞整个进程.
    - Q3. 在进程内,没有时钟中断,线程如何实现调度避免永久运行?
    - a3. 让运行时系统请求时钟信号(中断)
             开销太大,对程序不友好,会扰乱运行时系统使用的时钟.

* **内核态线程:** 将线程包放在内核空间实现,由内核管理线程
  + 优点:
    - 能够解决用户态线程中的主要问题
  + 缺点:
    - Q1:速度比较慢
    - Q2:创建,撤销线程等会耗费很多的资源等...

* **混合实现:** 在内核空间维持一些可以复用的线程.将进程空间的线程装载到内核线程中运行.内核只调用内核线程.每个内核级线程有一个可以轮流使用的用户级线程集合

<a>  **调度程序激活机制:**</a> 当内核了解到一个线程被阻塞以后,内核通知该进程的运行时系统,并且在堆栈中以参数形式传递被阻塞线程的编号和故障描述(**上行调用**).进程收到消息以后进行线程调度,从线程表中取出另一个已经就绪的线程装入该内核线程中继续执行.等到先前被阻塞的线程又可运行时,内核再次进行上行调用通知进程调整进程表,将该线程标记为可执行.
    _ps:_上行调用:_ 内核通过一个已知起始地址启动运行时系统发出通知 ,向上层传递消息.

<a> **弹出式线程:**</a> 分布式系统中使用线程.一个消息到达导致系统创建一个线程去处理该消息.该类线程没有必须的寄存器,堆栈等,可以快速创建以致消息到达与处理之间时间非常短.

### 进程和线程区别 ###
**线程共享的进程中内容**:  
地址空间,全局变量,打开文件,子进程,信号与信号处理程序,账户信息,即将发生的报警  
**每个线程独有内容:**
1. 程序计数器: _记录线程接下来要执行的指令_ 
2. 寄存器: _保存线程当前的工作变量_
3. 堆栈: _记录执行历史_
4. 状态: _1.new(新生) 2.runnable(可运行) 3.running(正在运行) 4.blocked           (被阻塞)  5.dead(死亡)_

### 进程(线程)间通信 ###

**临界区域(critical region):** 共享内存(临界资源)的程序片段  
**进程同步:** 直接制约关系.为了合作完成某个任务,两个或多个进程在某些地方要求有不同的工作顺序而等待,传递消息所产生的制约关系   
**进程互斥:** 间接制约关系.当一个进程使用临界资源时,另一进程必须等待,被阻塞直至占有临界资源的进程释放资源并退出临界区.

##### 互斥解决办法: #####

###### 1. 信号量: ######
  + **信号量:** 使用一个整型变量来累计唤醒次数,down和up原子操作对信号量进行处理

  + **生产者-消费者 --信号量:**
    
       ```
         /*
         *mutex实现互斥,保证某一时刻只能有一个进程读写缓冲区
         *available和used实现同步,保证事件顺序执行或不发生
         **/   
         #define N 100       //定义缓冲区大小为100
         typedef int semaphore; //定义信号量(实际为int类型)
         semaphore mutex = 1;   //定义对临界区访问控制信号量
         semaphore available = N;   //定义缓冲区可用数目
         semaphore used = 0;    //定义缓冲区已使用的数目
       
         //生产者
         void producer(void)
         {
             int item;
             while(TRUE)
             {
                 item = produce_item; //生产产品
                 down(&available);       //将缓冲区可用数目减一
                 down(&mutex);       //将控制信号量减一
                 insert_item(item);  //将产品放入缓冲区
                 up(&mutex);         //信号量加一
                 up(&used);          //缓冲区已使用数目加一
             }
         }
       
         //不能将mutex与available,used判断交换位置,交换后可能会产生死锁
         //消费者
         void consumer(void)
         {
             int item;
             while(TRUE)
             {
                 down(&used);         
                 down(&mutex);
                 item = remove_item();
                 up(&mutex);
                 up(&available);
                 consume_item(item);
             }
         }
       ```
      
      >   down,up是原子操作  
      >   down操作,检查值是否大于0,大于0则将值减1,若0则进程睡眠  
      >   up操作,对信号量加1操作,如果有一个或多个进程在该信号量上睡眠,则由系统选择其中一个进程唤醒, 并允许该进程完成它的down操作.  
      >   mutex初值为1,保证同时只能有一个进程可以进程临界区,称作**二元信号量**


###### 2.互斥量: ######
  + **互斥量实现原理:** 除去信号量的计数能力,相当于信号量的简化版本
  + **生产者-消费者问题 --互斥量:**

      > 使用Pthread包实现  
      > pthread_mutex_* 互斥量操作
      > pthread_cond_* 条件变量操作


        #include <stdio.h>
        #incldue <pthread.h>
        #define MAX 10000   //最大生产数量
        pthread_mutex_t the_mutex;  //定义一个互斥量
        pthread_cond_t condc,condp;  //定义条件变量
        int buffer = 0;     //数据缓冲区
        
        //生产者
        void *produce(void *ptr)
        {
            int i;
            for(i=1;i<MAX;i++)
            {
                pthread_mutex_lock(&the_mutex);  //加锁,互斥访问缓冲区
                while(buffer!=0)
                    pthread_cond_wait(&condp,&the_mutex); //阻塞线程在条件变量conp上,并释放the_mutex锁(释放缓冲区)
                buffer=i;       //将数据放入缓冲区
                pthread_cond_signal(&condc);    //唤醒消费者
                pthread_mutex_unlock(&the_mutex); //解锁,释放缓冲区
            }
            pthread_exit(0);
        }
        
        //消费者
        void *consumer(void *ptr)
        {
            int i;
            for(i=1;i<=MAX;i++)
            {
                pthread_mutex_lock(&the_mutex); //加锁,互斥访问缓冲区
                while(buffer==0)
                    pthread_cond_wait(&condc,&the_mutex);//阻塞消费者线程,并释放the_mutex互斥量 
                buffer = 0; //从缓冲区取出数据
                pthread_cond_signal(&condp); //唤醒阻塞在条件变量condp上的线程,唤醒生产者
                pthread_mutex_unlock(&the_mutex); //解锁,释放缓冲区
            }
            pthread_exit(0);
        }
    
        int main(int argc,char **argv)
        {
            pthread_t pro,con;   //定义生产者和消费者线程
            pthread_mutex_init(&the_mutex,0); //创建一个互斥量
            pthread_cond_init(&condc,0);      //创建条件变量--消费者
            pthread_cond_init(&condp,0);      //创建条件变量--生产者
            pthread_creat(&con,0,consumer,0); //创建消费者线程执行consumer函数
            pthread_creat(&pro,0,producer,0); //创建生产者线程执行producer函数
            pthread_join(pro,0);  //生产者pro线程运行
            pthread_join(con,0);  //消费者con线程运行
            pthread_cond_destroy(&condc);  //销毁条件变量
            pthread_cond_destroy(&condp);
            pthread_mutex_destroy(&the_mutex); //销毁互斥量
         }


###### 3.管程(monitor): ######

  + **管程:** 一个管程是一个过程,变量及数据结构等组成的一个集合.管程是语言概念仅JAVA和类pascal等一些语言支持,C/C++语言不支持.任意时刻管程中只能有一个活跃进程或线程
  + **管程解决生产者-消费者问题:**
    
          public class ProducerConsumer{
            static final int N = 100; //定义缓冲区大小
            static producer p = new producer(); //新建生产者线程 
            static consumer c = new consumer(); //新建消费者线程
            static our_monitor mon = new our_monitor(); //初始化一个管程
                
           	public static void main(String args[]){
                p.start();
                c.start();
            }
          
            //生产者
            static class producer extends Thread{
                public void run(){
                    int item;
                    while(true){
                        item = produce_item();
                        mon.insert(item);
                    }
                }
                private int produce_item(){
                    System.out.println("实际生产函数");
                    return 1;
                }
            }
          
            //消费者
            static class consumer extends Thread{
                public void run(){
                    int item;
                    while(true){
                        item=mon.remove();
                        consume_item(item);
                    }
                }
                private void consume_item(int item){
                    System.out.println("实际消费操作");
                }
            }
          
            //管程,sychronized同时只能允许一个进程或线程在其中
            static class our_monitor{
                private int buffer[] = new int[N];
                private int count=0, lo=0, hi=0;
            
                //静态同步函数
                //synchronized锁为当前类的字节码对象our_monitor.class
                public synchronized void insert(int val){
                    if(count==N)
                        please_wait();
                    buffer[hi]=val;
                    hi=(hi+1)%N; //设置存放下一数据的位置
                    count=count+1; 
                    if(count==1)
                        notify(); //唤醒wait()中线程
                }
            
                //synchronized锁为our_monitor.class
                public synchronized int remove(){
                    int val;
                    if(count==0)
                        please_wait();
                    val = buffer[lo];
                    lo=(lo+1)%N; //设置下一取出数据的位置
                    count=count-1;
                    if(count==N-1)
                        notify();
                }
                private void please_wait(){
                    try{wait();}
                    catch(InterruptedException exc){};
                }
            	}
            }
      
      
      > 同步代码块:synchronized(锁对象){}
      > 同步函数中synchronized的锁为this,当前对象本身
      > 静态同步函数中synchronized的锁为class,当前类的字节码对象

###### 4.消息传递 ######

+ **消息传递:** 信号量太低级,在分布式系统中具有多个CPU,信号量中down和up原语将失效,而管程又只有少数语言支持.所以分布式系统中需要消息传递来解决同步与互斥问题.
+ **生产者-消费者 --消息传递:**
        
        #define N 100
        
        void producer(void)
        {
            int item;
            message m;   //消息缓冲区
            while(true)
            {
                item = produce_item(); //产生数据
                receive(consumer,&m);  //等待消费者发送空缓冲区格子 
                build_message(&m,item);//将产生的数据放入空盒子
                send(consumer,&m);  //发送数据给消费者
            }
        }
        
        void consumer(void)
        {
            int itme;
            massage m;
            for(i=0;i<N;i++)
                send(producer,&m); //向生产者 发送一百个空盒子
            while(true)
            {
                receive(producer,&m); //从生产者接收带有数据的盒子
                item=extract_item(&m);//从盒子里取出数据
                send(producer,&m);    //将空盒子返回给生产者
                consume_item(item);   //处理数据
            }
        }