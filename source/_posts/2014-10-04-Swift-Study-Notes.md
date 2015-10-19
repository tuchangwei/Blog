title: Swift Study Notes
date: 2014-10-04 21:17:32
tags: Swift
---

# 402-hd-introduction-to-swift.mov

1. This is a general principle in Swift, that we prefer immutability or constants by default and only really opt into mutablility or variables where things actually need to change. Now this makes your code safer in a multi-threaded environment. It also means that Swift can optimize your code more effectively because it knows what isn't going to change and it just generally makes your code more readable, makes your intent clearer that you're saying what is and isn't going to vary.


2. Swift initializers don't return a value. Rather, their main role to make sure that every stored property has a value by the time its initialization completes.

3. Non-Optional Types can't be nil.

#403-hd-intermediate-swift.mov

1. Weak References tell Swift that this object isn't responsible for keeping the object on the other end alive. And weak references in Swift are modeled as optinonal values.

2.  Unowned References tell Swift that although I don't have an owning stake in the object, my lift depents on it, I can't live without my owner, my card holder.

3. Every value must be initialized before it is used.

4. Initializers handle the responsibility of fully initailizing an instance.

5. We always make sure we set our own properties before we call our SuperClass's initializer.

6. Convenience Initializers

```
init (color: Color, turbo: BOOl) {
	......
}
convenience init(color: Color) {
	
	self.init(color: color, turbo: true)
}
```

7. Lazy Properties.

It will only be evaluated the time it's accessed and only when it's accessed and only that one time.

```
@lazy var multiplayerManager  = MultiplayerManager()
```
8. Initialization

Initialize all values before you use them.

Set all stored properties first, then call super.init

Designated initializers only delegate up

Convenience initializers only delegate across

Deinitializers are there...if you need them.