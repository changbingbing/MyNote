从两个维度来说，默认加锁和手动加锁：

* 默认加锁：对于表锁的使用，一般直接默认由MySQL帮我们去使用，如何使用呢？
  * 如果是DML和DQL语句，则默认加上读锁：`lock table xxx read`；
  * 如果是DDL语句，则默认加上写锁：`lock table xxx write`
* 手动加锁
  * 如果是DQL语句，则加读锁：
  * 如果是DML、DDL，则加写锁；