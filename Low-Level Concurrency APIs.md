# 底层并发 API

## dispatch_once
---
```
+ (UIColor *)boringColor;
{
    static UIColor *color;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        color = [UIColor colorWithRed:0.380f green:0.376f blue:0.376f alpha:1.000f];
    });
    return color;
}
```
>要确保 `onceToken` 被声明为 `static` ，或者有全局作用域。任何其他的情况都会导致无法预知的行为。

## dispatch_after
---
```
- (void)foo
{
    double delayInSeconds = 2.0;
    dispatch_time_t popTime = dispatch_time(DISPATCH_TIME_NOW, (int64_t) (delayInSeconds * NSEC_PER_SEC));
    dispatch_after(popTime, dispatch_get_main_queue(), ^(void){
        [self bar];
    });
}
```

## 队列
---
```
- (id)init;
{
    self = [super init];
    if (self != nil) {
        NSString *label = [NSString stringWithFormat:@"%@.isolation.%p", [self class], self];
        self.isolationQueue = dispatch_queue_create([label UTF8String], 0);

        label = [NSString stringWithFormat:@"%@.work.%p", [self class], self];
        self.workQueue = dispatch_queue_create([label UTF8String], 0);
    }
    return self;
}
```

>需要注意的是：如果你提交了一个 block 给 GCD，但是这段代码阻塞了这个线程，那么这个线程在这段时间内就不能用来完成其他工作——它被阻塞了。为了确保功能点在队列上一直是执行的，GCD 不得不创建一个新的线程，并把它添加到线程池。

>如果你的代码阻塞了许多线程，这会带来很大的问题。首先，线程消耗资源，此外，创建线程会变得代价高昂。创建过程需要一些时间。并且在这段时间中，GCD 无法以全速来完成功能点。有不少能够导致线程阻塞的情况，但是最常见的情况与 I/O 有关，也就是从文件或者网络中读写数据。正是因为这些原因，你不应该在GCD队列中以阻塞的方式来做这些操作。

### Target Queue
你能够为你创建的任何一个队列设置一个目标队列。

>为一个类创建它自己的队列而不是使用全局的队列被普遍认为是一种好的风格。这种方式下，你可以设置队列的名字，这让调试变得轻松许多—— Xcode 可以让你在 Debug Navigator 中看到所有的队列名字，如果你直接使用 lldb。(lldb) thread list 命令将会在控制台打印出所有队列的名字。

>如果你希望多个 block 同时运行，那要确保你自己的队列是并发的。同时需要注意，如果一个队列的目标队列是串行的（也就是非并发），那么实际上这个队列也会转换为一个串行队列。

### Priorities
>你应该克制这么做的冲动。

>使用 `DISPATCH_QUEUE_PRIORITY_BACKGROUND` 队列时，你需要格外小心。除非你理解了 throttled I/O 和 background status as per setpriority(2) 的意义，否则不要使用它。不然，系统可能会以难以忍受的方式终止你的 app 的运行。打算以不干扰系统其他正在做 I/O 操作的方式去做 I/O 操作时，一旦和优先级反转情况结合起来，这会变成一种危险的情况。

## 隔离队列
### 资源保护
```
- (void)setCount:(NSUInteger)count forKey:(NSString *)key
{
    key = [key copy];
    dispatch_async(self.isolationQueue, ^(){
        if (count == 0) {
            [self.counts removeObjectForKey:key];
        } else {
            self.counts[key] = @(count);
        }
    });
}

- (NSUInteger)countForKey:(NSString *)key;
{
    __block NSUInteger count;
    dispatch_sync(self.isolationQueue, ^(){
        NSNumber *n = self.counts[key];
        count = [n unsignedIntegerValue];
    });
    return count;
}
```

注意以下几点：

* 不要使用上面的代码。
* 我们使用 async 方式来保存值，这很重要。我们不想也不必阻塞当前线程只是为了等待写操作完成。当读操作时，我们使用 sync 因为我们需要返回值。
* 从函数接口可以看出，-setCount:forKey: 需要一个 NSString 参数，用来传递给 dispatch_async。函数调用者可以自由传递一个 NSMutableString 值并且能够在函数返回后修改它。因此我们必须对传入的字符串使用 copy 操作以确保函数能够正确地工作。如果传入的字符串不是可变的（也就是正常的 NSString 类型），调用copy基本上是个空操作。
* `isolationQueue` 创建时，参数 `dispatch_queue_attr_t` 的值必须是`DISPATCH_QUEUE_SERIAL`（或者0）。

### 单一资源的多读单写
以如下方式创建队列：
```
self.isolationQueue = dispatch_queue_create([label UTF8String], DISPATCH_QUEUE_CONCURRENT);
```

并且用以下代码来改变setter函数：
```
- (void)setCount:(NSUInteger)count forKey:(NSString *)key
{
    key = [key copy];
    dispatch_barrier_async(self.isolationQueue, ^(){
        if (count == 0) {
            [self.counts removeObjectForKey:key];
        } else {
            self.counts[key] = @(count);
        }
    });
}
```

`dispatch_barrier_async`
Submits a barrier block for asynchronous execution and returns immediately.
>Calls to this function always return immediately after the block has been submitted and never wait for the block to be invoked. When the barrier block reaches the front of a private concurrent queue, it is not executed immediately. Instead, the queue waits until its currently executing blocks finish executing. At that point, the barrier block executes by itself. Any blocks submitted after the barrier block are not executed until the barrier block completes.

### 锁竞争

>这里有两个开销需要你来平衡。第一个是独占临界区资源太久的开销，以至于别的线程都因为进入临界区的操作而阻塞。第二个是太频繁出入临界区的开销。在 GCD 的世界里，第一种开销的情况就是一个 block 在隔离队列中运行，它可能潜在的阻塞了其他将要在这个隔离队列中运行的代码。第二种开销对应的就是调用 dispatch_async 和 dispatch_sync 。无论再怎么优化，这两个操作都不是无代价的。

### 全都使用异步分发
在 GCD 中，以同步分发的方式非常容易出现死锁。见下面的代码：
```
dispatch_queue_t queueA; // assume we have this
dispatch_sync(queueA, ^(){
    dispatch_sync(queueA, ^(){
        foo();
    });
});
```
一旦我们进入到第二个 dispatch_sync 就会发生死锁。我们不能分发到queueA，因为有人（当前线程）正在队列中并且永远不会离开。但是有更隐晦的产生死锁方式：

```
dispatch_queue_t queueA; // assume we have this
dispatch_queue_t queueB; // assume we have this

dispatch_sync(queueA, ^(){
    foo();
});

void foo(void)
{
    dispatch_sync(queueB, ^(){
        bar();
    });
}

void bar(void)
{
    dispatch_sync(queueA, ^(){
        baz();
    });
}
```
单独的每次调用 dispatch_sync() 看起来都没有问题，但是一旦组合起来，就会发生死锁。

### 如何写出好的异步 API
正如我们刚刚提到的，你需要倾向于异步 API。当你创建一个 API，它会在你的控制之外以各种方式调用，如果你的代码能产生死锁，那么死锁就会发生。

如果你需要写的函数或者方法，那么让它们调用 dispatch_async() 。不要让你的函数调用者来这么做，这个调用应该在你的方法或者函数中来做。

如果你的方法或函数有一个返回值，异步地将其传递给一个回调处理程序。这个 API 应该是这样的，你的方法或函数同时持有一个结果 block 和一个将结果传递过去的队列。你函数的调用者不需要自己来做分发。这么做的原因很简单：几乎所有时间，函数调用都应该在一个适当的队列中，而且以这种方式编写的代码是很容易阅读的。总之，你的函数将会（必须）调用 dispatch_async() 去运行回调处理程序，所以它同时也可能在需要调用的队列上做这些工作。

如果你写一个类，让你类的使用者设置一个回调处理队列或许会是一个好的选择。你的代码可能像这样：
```
- (void)processImage:(UIImage *)image completionHandler:(void(^)(BOOL success))handler;
{
    dispatch_async(self.isolationQueue, ^(void){
        // do actual processing here
        dispatch_async(self.resultQueue, ^(void){
            handler(YES);
        });
    });
}
```
如果你以这种方式来写你的类，让类之间协同工作就会变得容易。如果类 A 使用了类 B，它会把自己的隔离队列设置为 B 的回调队列。

## 组
---

### dispatch_group_async
很多时候，你发现需要将异步的 block 组合起来去完成一个给定的任务。这些任务中甚至有些是并行的。现在，如果你想要在这些任务都执行完成后运行一些代码，"groups" 可以完成这项任务。看这里的例子：
```
dispatch_group_t group = dispatch_group_create();

dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_group_async(group, queue, ^(){
    // Do something that takes a while
    [self doSomeFoo];
    dispatch_group_async(group, dispatch_get_main_queue(), ^(){
        self.foo = 42;
    });
});
dispatch_group_async(group, queue, ^(){
    // Do something else that takes a while
    [self doSomeBar];
    dispatch_group_async(group, dispatch_get_main_queue(), ^(){
        self.bar = 1;
    });
});

// This block will run once everything above is done:
dispatch_group_notify(group, dispatch_get_main_queue(), ^(){
    NSLog(@"foo: %d", self.foo);
    NSLog(@"bar: %d", self.bar);
});
```
需要注意的重要事情是，所有的这些都是非阻塞的。我们从未让当前的线程一直等待直到别的任务做完。恰恰相反，我们只是简单的将多个 block 放入队列。由于代码不会阻塞，所以就不会产生死锁。

### dispatch_group_enter & dispatch_group_leave

举例来说，我们可以给 Core Data 的 -performBlock: API 函数添加上 groups，就像这样：

```
- (void)withGroup:(dispatch_group_t)group performBlock:(dispatch_block_t)block
{
    if (group == NULL) {
        [self performBlock:block];
    } else {
        dispatch_group_enter(group);
        [self performBlock:^(){
            block();
            dispatch_group_leave(group);
        }];
    }
}
```
为了能正常工作，你需要确保:

* dispatch_group_enter() 必须要在 dispatch_group_leave()之前运行。
* dispatch_group_enter() 和 dispatch_group_leave() 一直是成对出现的（就算有错误产生时）。
