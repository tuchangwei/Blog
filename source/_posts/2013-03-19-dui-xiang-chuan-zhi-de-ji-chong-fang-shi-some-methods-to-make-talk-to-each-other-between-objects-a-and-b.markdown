---
layout: post
title: "对象传值的几种方式(Some  methods to make talk to each other between objects A and B)"
date: 2013-03-19 21:59
comments: true
categories: Summary
---
1. **NSNotificationCenter**. This is anonymous one-to-many communication. Object A posts a notification to the NSNotificationCenter, which then distributes it to any other objects listening for that notification, including Object B. A and B do not have to know anything about each other, so this is a very loose coupling. Maybe a little too loose…
2. **KVO (Key-Value Observing)**. One object observes the properties of another. This is a very tight coupling, because Object B is now peeking directly into Object A. The advantage of KVO is that Object A doesn’t have to be aware of this at all, and therefore does not need to send out any notifications — the KVO mechanism takes care of this behind the scenes.
3. **Direct pointers**. Object A has a pointer to Object B and directly sends it messages when something of interest happens. This is the tightest coupling possible because A and B cannot function without each other. In the case of view controllers you generally want to avoid this.
4. **Delegates**. Object B is a delegate of Object A. In this scenario, Object A does not know anything about Object B. It just knows that some object performs the role of its delegate and it will happily send messages to that delegate, but it doesn’t know — or care — that this is Object B. The delegate pattern is often the preferred way to communicate between view controllers, but it takes some work to set up.
5. **Blocks**. Essentially the same approach as delegates, except that Object B now gives Object A one or more blocks (closures) to be executed when certain events take place. There is no formal delegate protocol and the only thing that Object A sees of Object B is the blocks it is given.

话说KVO检测对象的属性和Blocks相互传值还没使用过，回头试一下。