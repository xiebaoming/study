常见问题 

谈谈你理解的 Hashtable，讲讲其中的 get put 过程。ConcurrentHashMap同问。
1.8 做了什么优化？
线程安全怎么做的？
不安全会导致哪些问题？
hashMap和concurrentHashMap区别
保证数据一致性的方法
如何解决？有没有线程安全的并发容器？
ConcurrentHashMap 是如何实现的？ 
ConcurrentHashMap并发度为啥好这么多？
1.7、1.8 实现有何不同？为什么这么做？
CAS是啥？
ABA是啥？场景有哪些，怎么解决？
synchronized底层原理是啥？
synchronized锁升级策略
快速失败（fail—fast）是啥，应用场景有哪些？安全失败（fail—safe）同问。
……

加分项 
在回答Hashtable和ConcurrentHashMap相关的面试题的时候，一定要知道他们是怎么保证线程安全的，那线程不安全一般都是发生在存取的过程中的，那get、put你肯定要知道。
HashMap是必问的那种，这两个经常会作为替补问题，不过也经常问，他们本身的机制其实都比较简单，特别是ConcurrentHashMap跟HashMap是很像的，只是是否线程安全这点不同。
提到线程安全那你就要知道相关的知识点了，比如说到CAS你一定要知道ABA的问题，提到synchronized那你要知道他的原理，他锁对象，方法、代码块，在底层是怎么实现的。
synchronized你还需要知道他的锁升级机制，以及他的兄弟ReentantLock，两者一个是jvm层面的一个是jdk层面的，还是有很大的区别的。
那提到他们两个你是不是又需要知道juc这个包下面的所有的常用类，以及他们的底层原理了？
那提到……
