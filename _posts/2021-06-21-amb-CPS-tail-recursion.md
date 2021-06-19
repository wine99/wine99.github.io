---
title: amb & CPS & tail recursion
categories: [CS]
comment: false
---

## amb

教材中 amb evaluator 的实现是 CPS，虽然教材中没有明确提出 CPS 这一概念。由于第一次接触 CPS，要理解 amb 的实现还是比较困难的。

在 [这个讨论帖](https://comp.lang.scheme.narkive.com/3gHLvdQj/sicp-why-is-idea-of-the-amb-evaluator-hard#post5) 中，一位用户给出了在最初的 evaluator 基础上实现的 amb，而不是在 analyzing evaluator 的基础上。事实上，这位用户正是某一年 Berkeley 的 SICP (CS61A) 讲师 Brian Harvey（应该是 2004 年，当时的 CS61A 还没有转向使用 Python 教学）。

Harvey 说，他曾与人争论，使用 analyzing evaluator 而不是最简单的 evaluator 增加了理解教材上这一 amb 实现的难度，但是他的争论失败了。

由于帖子中的答案格式不利于阅读，我在 Berkeley 的网站上发现了原文 pdf：[https://inst.eecs.berkeley.edu/~cs61a/su10/lectures/lecture24.pdf](https://inst.eecs.berkeley.edu/~cs61a/su10/lectures/lecture24.pdf)。但帖子中给出的基于原始 evaluator 的 amb 实现的代码并没有在网上找到，因此我将它搬运到了我的 github 上：[https://github.com/wine99/learn-sicp/blob/master/chap4/sec3/amb-evaluator.scm](https://github.com/wine99/learn-sicp/blob/master/chap4/sec3/amb-evaluator.scm)。

最简单有效的理解方法也是最笨的方法，手动模拟：

![amb_CPS_tail_recursion_1.jpg](https://gitee.com/wine99/pics/raw/master/2021/06/amb_CPS_tail_recursion_1.jpg)

## CPS

CPS的另一例子是教材 5.2 中 extract-labels 的实现（优雅的返回多个值）以及 exercise 5.17。

![amb_CPS_tail_recursion_2.png](https://gitee.com/wine99/pics/raw/master/2021/06/amb_CPS_tail_recursion_2.png)

分析：在 `(cdr text)` 上递归调用的结果会变成 `(lambda (insts labels) ...)` 的实参，在这个 lambda body 中，将 `(car text)` 的分析结果 cons 到在 `(cdr text)` 上的调用结果上去，将 cons 的结果告诉调用外层 extract-labels 的那个 caller，也就是调用外层 extract-labels 的 receive(continuation)。

这是我的习题 5.17 的 solution: [https://github.com/wine99/learn-sicp/blob/master/chap5/sec2/17-trace.scm](https://github.com/wine99/learn-sicp/blob/master/chap5/sec2/17-trace.scm)。

其实，JavaScript 里一直在用的回调，本质也就是 CPS，只不过绝大部分情况下，我们的回调只是进行简单的后续工作的处理，而这里用 callback/continuation 来优雅地实现需要返回多个值的递归。

## 尾递归优化

教材 5.4 中给出了给出了尾递归优化和没有尾递归优化两种“汇编”代码。

![amb_CPS_tail_recursion_3.png](https://gitee.com/wine99/pics/raw/master/2021/06/amb_CPS_tail_recursion_3.png)

![amb_CPS_tail_recursion_4.png](https://gitee.com/wine99/pics/raw/master/2021/06/amb_CPS_tail_recursion_4.png)

对于一个 exp sequence，eval 最后一个 exp 时，不需要再将 unev 和 env save 到栈里，并且由 ev-application 或 ev-begin 保存到栈里的continue 也可以重新 restore 回 continue 寄存器，然后直接继续调用 eval-dispatch，如果这最后一个 exp 是 application，ev-application又会把 continue 重新 save 到栈里，如果是其他的 `ev-<special-form>` ，eval 完后便直接 `(goto (reg continue))`。

PS：对于“汇编”代码理解有困难的话，可以阅读教材给出的斐波那契数列的汇编代码，就能明白如何利用 continue 寄存器来实现递归。

而如果没有做尾递归优化，最后一个 exp 将会同前面的 exp 一样，eval 前仍会将 unev 和 env save 到栈里，并且再把 continue 设为 ev-sequence-continue（即使将来继续这个 continue 时，也没有更多 exp 要 eval 了）。然后，如果这最后一个 exp 是 application，同样地再次 `(save continue)`；和优化版本相比，这里栈里就有两个 continue 了；将来随着递归的深入，还会有更多的 continue（以及 unev 和 env）被保存到栈里，但这些新的 continue 没有什么用：它们在 ev-sequence-end 中被 restore 然后 goto，层层往上，最终目的只是为了回到最初的 ev-application 或 ev-begin 保存的那个 continue。并且，由于 unev 和 env 都被保存在了栈里，这些 env 里的数据将不再被 GC 回收。（这样看来，尾递归优化必须搭配垃圾回收。）

注意，观察上面的 extract-labels 过程，很容易发现，尾递归调用的第二个参数是一个 lambda，而这个 lambda 里面用到了 caller 的参数 text，那么 text 必然需要被保存下来。这是怎么做到的呢，不是说尾递归优化吗，不是说“it was unnecessary to save information in this case”吗？虽然 env 和 unev 确实没有被存到栈中，但当这个 lambda 被 eval 时，构造的 procedure 的 procedure-environment 指向了当前这个 env，而这个 procedure 是 argl 的一部分，是内存中“可达”的数据，自然就不会被 GC 回收。因此，如果尾递归调用里的参数存在在当前 env 里构造出来 procedure，这个 env 就会被保存下来，这样的尾递归仍然会造成内存空间的不断增长。但这是必要的，毕竟我们确实需要 caller 里的信息。当然，也有可能这个作为参数的过程里并没有用到 caller 的信息，或许更强大/聪明的编译器和 GC 能对 procedure body 做分析从而进行更激进/彻底的垃圾回收。

Again，如果理解有问题，那就手动模拟：

![amb_CPS_tail_recursion_5.jpg](https://gitee.com/wine99/pics/raw/master/2021/06/amb_CPS_tail_recursion_5.jpg)
