---
title: EventBus
date: 2019-07-09 23:22:26
tags:
- 事件
categories: 
- DotNet
---
### 引言
+ 事件总线这个概念对你来说可能很陌生，但提到观察者（发布-订阅）模式，你也许就很熟悉。事件总线是对发布-订阅模式的一种实现。它是一种集中式事件处理机制，允许不同的组件之间进行彼此通信而又不需要相互依赖，达到一种解耦的目的。
+ 本质：事件是由事件源和事件处理组成。
+ 发布订阅模式：定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并被自动更新。 ——发布订阅模式。

### 基本思路
1. 在事件总线内部维护着一个事件与事件处理程序相映射的字典。
2. 利用反射，事件总线会将实现了IEventHandler的处理程序与相应事件关联到一起，相当于实现了事件处理程序对事件的订阅。
3. 当发布事件时，事件总线会从字典中找出相应的事件处理程序，然后利用反射去调用事件处理程序中的方法。

### 基于内存事件总线实现
> 定义一个事件接口
``` csharp
    public interface IEvent
    {
    }
```
> 定义一个事件处理接口
``` csharp
    public  interface  IEventHandler : IEvent
    {
       Task Handle(IEvent e);
    }
```
> 定义一个发布接口
``` csharp
    public interface IEventSubscriber 
    {    
        Task Subscribe<TEvent, EH>() where TEvent : IEvent where EH : class, IEventHandler, new();
    }
```
> 创建一个类用来存事件
``` csharp
    public static class MemoryMq
    {
        public static ConcurrentDictionary<string, IEvent> eventQueueDict { get; set; }
    }
```
> 实现发布类
``` csharp
	public class InMemoryEventPublisher : IEventPublisher
	{
		public Task Publish<TEvent>(TEvent e) where TEvent : IEvent
		{
			if (e == null) return Task.CompletedTask;
			if (MemoryMq.eventQueueDict == null)
			{
				MemoryMq.eventQueueDict = new ConcurrentDictionary<string, IEvent>();
			}
			MemoryMq.eventQueueDict.GetOrAdd(Guid.NewGuid().ToString(), e);
			return Task.CompletedTask;
		}
	}
```
> 实现订阅类
``` csharp
	public class InMemoryEventSubscriber : IEventSubscriber
	{
		public Task Subscribe<TEvent, EH>() where TEvent : IEvent
			 where EH : class, IEventHandler, new()
		{
			EH state = new EH();
			Task.Run(() =>
			{
				while (true)
				{
					if (MemoryMq.eventQueueDict != null)
					{
						foreach (var item in MemoryMq.eventQueueDict)
						{
							state.Handle(item.Value as IEvent);
							IEvent o;
							MemoryMq.eventQueueDict.TryRemove(item.Key, out o);
						}
					}
				}
			});
			return Task.CompletedTask;
		}
```
> 实现逻辑处理
``` csharp
	public class EventHandler : IEventHandler
	{
		public Task Handle(IEvent e)
		{
			switch (e)
			{
				case Order value:
					Console.WriteLine(value.Name);
					break;
			}
			return Task.CompletedTask;
		}
	}

    public class Order : IEvent
    {
        public string name { get; set; }
    }
```
> 测试
``` csharp
	class Program
	{
		static void Main(string[] args)
		{
			var pub = new InMemoryEventPublisher();
			var sub = new InMemoryEventSubscriber();
			sub.Subscribe<Order, EventHandler>();
			var order = new Order();
			order.Name = "wangjunzzz";
			pub.Publish(order);
			Console.WriteLine("Hello World!");
			Console.ReadKey();
		}
	}
```