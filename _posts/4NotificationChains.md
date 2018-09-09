---
layout: post
title: "CHAPTER4 Notification Chains"
author: "omnihorizon"
tags:
- network
- linux kernel

---
#CHAPTER4 Notification Chains
内核的许多子系统之间相互依赖，一个子系统探测或产生的事件可能会被其他多个子系统需要。内核使用了 notification chains 来满足这种交互性。

在本章，我们将会学到：

* 内核网络中声明和定义 notification chains
* 内核子系统注册到一个 notification chains
* 内核子系统在一个 chain 上生成一个 notification

notification chains 仅用于内核子系统间。内核与用户态的 notification 依赖于其它机制，例如第三章所述的方式。

##Overview
notification chain 是一组函数列表，当某些事件发生时 chain 上的函数会执行。
notification chain有两种，一种是 notified 端，一种是 notifier 端，被称为 publish-and-subscribe 模型：

* notified 是指当某些事件发生时，子系统希望被通知，并提供了一些回调函数被调用的对象
* notifier 是指生成时间并调用回调函数的对象

notification chain 的拥有者定义了一个列表，任意子系统都可以向这个列表注册回调函数

##Defining a Chain
Chain 是一个链表，链上的每个元素定义如下：

```c
struct notifier_block
     {
         int (*notifier_call)(struct notifier_block *self, unsigned long, void *);
         struct notifier_block *next;
         int priority;
};
```
notifier_call ：执行的函数
next：下一个元素的指针
priority：函数执行的优先级，chain 上的函数按照优先级排序。拥有较高优先级的函数会被优先执行。实际上，几乎所有的函数都是默认优先级 0，执行顺序为注册顺序

notification chain 通常的命名为 xxx_chain, xxx_notifier_chain, 和 xxx_notifier_list.

##Registering with a Chain
当一个内核子系统对某一个 notification chain 上的事件感兴趣时，会调用 notifier\_chain_register 注册，notifier\_chain\_unregister 用于取消注册。内核提供了一系列关于注册函数的包装。

```c
int notifier_chain_register(struct notifier_block **list, struct
notifier_block *n)
int notifier_chain_unregister(struct notifier_block **nl, struct
notifier_block *n)
int notifier_call_chain(struct notifier_block **n, unsigned long
val, void *v)
```
|  wrappers |  |  
| ------ | ------ | 
| inetaddr_chain | register\_inetaddr_notifier |
| inet6addr_chain | register\_inet6addr_notifier |
| netdev_chain | register\_netdevice_notifier |
| inetaddr_chain | unregister\_inetaddr_notifier |
| inet6addr_chain | unregister\_inet6addr_notifier |
| netdev_chain | unregister\_netdevice_notifier |

notification chains 使用 notifier_lock 限制访问。子系统通常只在系统启动或者模块加载的时候注册回调函数，之后会以只读的方式访问 notification chains，因此锁的限制性能影响不大。

```c
int notifier_chain_register(struct notifier_block **list, struct notifier_block *n)
     {
         write_lock(&notifier_lock);
         while(*list)
         {
             if(n->priority > (*list)->priority)
                 break;
             list= &((*list)->next);
         }
         n->next = *list;
         *list=n;
         write_unlock(&notifier_lock);
         return 0;
}
```

##Notifying Events on a Chain
链上的注册的回调函数调用 notifier_call_chain 生成 Notifications

```c
int notifier_call_chain(struct notifier_block **n, unsigned long val, void *v)
    {
        int ret = NOTIFY_DONE;
        struct notifier_block *nb = *n;
		  while (nb) {
            ret = nb->notifier_call(nb, val, v);
            if (ret & NOTIFY_STOP_MASK)
            {
			  return ret; }
            nb = nb->next;
        }
		  return ret; }
```
n : Notification chain
val : 事件类型
v ： 回调函数输入参数

回调函数返回的结果定义在 include/linux/notifier.h
NOTIFY_OK ： 回调正确执行
NOTIFY\_DONE ：对事件不感兴趣
NOTIFY\_BAD：出现错误
NOTIFY\_STOP：线程正常调用，该事件没有更多的回调函数需要被调用
NOTIFY\_STOP\_MASK：用于 notifier\_call\_chain 去检验是否终止线程

notifier\_call_chain抓取最后一个被调用的回调函数的返回值。

##Notification Chains for the Networking Subsystems
inetaddr_chain ： 本地接口 ipv4 协议的插入、删除和修改的相关 notification

netdev_chain ： 网络设备注册状况的 notification


