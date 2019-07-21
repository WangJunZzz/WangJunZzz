---
title: Async-Await
date: 2019-07-21 15:21:36
tags:
- 基础
categories: 
- DotNet

---
### Task对象的前世今生
+ Task对象是.Net Framework 4.0之后出现的异步编程的一个重要对象。在一定程度上来说，Task对象可以理解Thread对象的一个升级产品。
- 示例
``` csharp
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("执行方法之前的时间：" + DateTime.Now.ToString("yyyy-MM-dd hh:MM:sss"));
            var result = Task.Run(() =>{return GetResult();});
            Console.WriteLine("主线程Id:" + Thread.CurrentThread.ManagedThreadId);
            Console.WriteLine(result.Result);
            Console.WriteLine("执行方法之后的时间：" + DateTime.Now.ToString("yyyy-MM-dd hh:MM:sss"));
            Console.ReadKey();
        }

        public static string GetResult()
        {
            Thread.Sleep(2000);
            Console.WriteLine("GetResult的线程Id:" + Thread.CurrentThread.ManagedThreadId);
            return "WangJunZzz";
        }
    }
```
- 结果
``` csharp
    #执行方法之前的时间：2019-07-21 01:07:42
    #WangJunZzz
    #执行方法之后的时间：2019-07-21 01:07:44
    #PS E:\Code\async\async-await> dotnet run
    #执行方法之前的时间：2019-07-21 01:07:52
    #主线程Id:1
    #GetResult的线程Id:3
    #WangJunZzz
    #执行方法之后的时间：2019-07-21 01:07:54
```
> 从结果分析得知，var result = Task.Run(() =>{return GetResult();});这一句后，主线程并没有阻塞取执行GetResult()方法，而是开启一个线程取执行GetResult()方法。知道执行result.Result这一句的时候才会等待GetResult()方法执行完毕。由此可知，Task.Run(()=>{}).Reslut是阻塞主线程的，主要因为主线程要得到返回值，必须等方法执行完毕。

### 线程不安全
- 示例
``` csharp
    class Program
    {
        private static bool isDone = false;
        static void Main(string[] args)
        {
            new Thread(Done).Start();
            new Thread(Done).Start();
            Console.ReadKey();
        }

        static void Done()
        {
            if (!isDone)
            {
                Console.WriteLine("Done");
                isDone = true;
            }
        }
    }
```
- 结果
``` csharp
    #Done
    #Done
```
>第一个线程还没有来得及把isDone设置成true，第二个线程就进来了，而这不是我们想要的结果，在多个线程下，结果不是我们的预期结果，这就是线程不安全。

### 锁
- 示例
```csharp
    class Program
    {
        private static bool isDone = false;
        private static object mylock = new object();
        static void Main(string[] args)
        {
            new Thread(Done).Start();
            new Thread(Done).Start();
            Console.ReadKey();
        }

        static void Done()
        {
            lock(mylock)
            {
            if (!isDone)
            {
                Console.WriteLine("Done");
                isDone = true;
            }
            }
        }
    }
```
>加上锁之后，被锁住的代码在同一个时间内只允许一个线程访问，其它的线程会被阻塞，只有等到这个锁被释放之后其它的线程才能执行被锁住的代码。
### Async,Await
1. 使用它们，方法的返回类型应为Task类型。为了使用await关键字，您必须在方法定义中使用async。如果你在方法定义中放入async，你应该在主体方法的某个地方至少有一处await关键字，如果你缺少他，你通常会收到Visual Studio的一个警告。
2. await 之后不会开启新的线程(await 从来不会开启新的线程)。
3. 加上await关键字之后，后面的代码会被挂起等待，直到task执行完毕有返回值的时候才会继续向下执行，这一段时间主线程会处于挂起状态。
4. Task.Run(()=>{}); 将一个任务添加到线程池里，排队执行。
5. async 标识一个方法为异步方法，可以与主线程并行执行，发挥CPU的多核优势。
6. await 在调用一个async方法前可以添加这个修饰符，它意思是等待当前异步方法执行完后，再执行下面的代码。
7.  ConfigureAwait(true)，代码由同步执行进入异步执行时，当前线程上下文信息就会被捕获并保存至 SynchronizationContext中，供异步执行中使用，并且供异步执行完成之后的同步执行中使用。
8. Configurewait(flase)，不进行线程上下文信息的捕获，async方法中与await之后的代码执行时就无法获取await之前的线程的上下文信息，在ASP.NET中最直接的影响就是HttpConext.Current的值为null，但不会出现非空引用的错误。
9. async配合Task是为了简化异步调用，异步调用的目的恰恰就是为了释放线程资源。
10. 在await的时候，线程就被释放了，等到异步操作完成才会回头来继续执行后面的代码。async的用途恰恰就在于避免阻塞，写的代码看起来像是同步的阻塞代码，但编译后不是，这才是async的精髓。(https://www.cnblogs.com/jesse2013/p/async-and-await.html 评论)。
11. 在.NetCore中我们不用继续关心异步同步混用情况下，是否哪里没有设置ConfigureAwait(false) 会导致的死锁问题，因为在.netcore中的async/await 可能在任何线程上执行，并且可能并行运行！。
