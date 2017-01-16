# Objective-C Associated Objects 的实现原理
[参考](http://blog.leichunfeng.com/blog/2015/06/26/objective-c-associated-objects-implementation-principle/)
## 问题
1. 关联对象被存储在什么地方，是不是存放在被关联对象本身的内存中？
* 关联对象的五种关联策略有什么区别，有什么坑？
* 关联对象的生命周期是怎样的，什么时候被释放，什么时候被移除？

## 相关函数
```
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);
id objc_getAssociatedObject(id object, const void *key);
void objc_removeAssociatedObjects(id object);
```
`objc_setAssociatedObject` 用于给对象添加关联对象，传入 nil 则可以移除已有的关联对象；
`objc_getAssociatedObject` 用于获取关联对象；
`objc_removeAssociatedObjects` 用于移除一个对象的所有关联对象。

## 总结
* 关联对象与被关联对象本身的存储并没有直接的关系，它是存储在单独的哈希表中的；
* 关联对象的五种关联策略与属性的限定符非常类似，在绝大多数情况下，我们都会使用 `OBJC_ASSOCIATION_RETAIN_NONATOMIC` 的关联策略，这可以保证我们持有关联对象；
* 关联对象的释放时机与移除时机并不总是一致，比如实验中用关联策略 OBJC_ASSOCIATION_ASSIGN 进行关联的对象，很早就已经被释放了，但是并没有被移除，而再使用这个关联对象时就会造成 Crash 。
