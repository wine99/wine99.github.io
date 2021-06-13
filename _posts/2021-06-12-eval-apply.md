---
title: 初识 `eval-apply` / A First Look into `eval-apply`
categories: [CS]
comment: false
---

## 起因

在 **SICP** 第四章的 **4.1.6 Internal Definitions** 小节的开头给出了如下的例子。

```scheme
(define (f x)
  (define (even? n)
    (if (= n 0)
        true
        (odd? (- n 1))))
  (define (odd? n)
    (if (= n 0)
        false
        (even? (- n 1))))
  ⟨rest of body of f⟩)
```

显然，这个例子能正常运行，无论是使用现有的 Scheme 解释器，还是在这一章我们自己编写的元循环求值器。但这是因为这两个互相递归的定义恰巧在代码段的开头，如果在 `odd?` 定义之前调用了 `even?`，我们的元循环求值器便无法工作。

一个简单的解决方法是将代码进行如下的转换。

```scheme
(lambda ⟨vars⟩
  (define u ⟨e1⟩)
  (define v ⟨e2⟩)
  ⟨e3⟩)

; would be transformed into

(lambda ⟨vars⟩
  (let ((u '*unassigned*)
        (v '*unassigned*))
    (set! u ⟨e1⟩)
    (set! v ⟨e2⟩)
    ⟨e3⟩))
```

在 **Exercise 4.20** 中，给出了如下的代码。

```scheme
(define (f x)
  (letrec
      ((even?
        (lambda (n)
          (if (= n 0)
              true
              (odd? (- n 1)))))
       (odd?
        (lambda (n)
          (if (= n 0)
              false
              (even? (- n 1))))))
    ⟨rest of body of f⟩))
```

`letrec` 表达式的语法规则是这样的。

```scheme
(letrec ((⟨var₁⟩ ⟨exp₁⟩) … (⟨varₙ⟩ ⟨expₙ⟩))
  ⟨body⟩)
```

**Exercise 4.20** 要求用 derived expression 的方式，在我们的求值器中实现 `letrec`。

显然，解决方法类似于实现 `let`，在 `eval` 中添加一个情况，把 `letrec` 转换成上面的 `let ... '*unassigned* ...` 即可。

但是，等一下，为什么我们需要 `letrec`，直接用 `let` 不行吗？

## Odd MIT-Scheme

于是我尝试了这样的代码。

```scheme
(eval '(let ((a 1)
             (b (lambda () (+ a 1))))
         (+ a (b)))
      the-global-environment)
```

结果居然是 `Unbound variable b`？

这时我发现我运行的求值器是还没有实现 `let` 的版本，但这样的话不应该是报 `Unbound variable let` 吗？为什么会是 b 呢？

把 `let` 转换成对应的 `lambda` 形式，再次运行。

```scheme
(eval '((lambda (a b) (+ a (b)))
        1
        (lambda () (+ a 1)))
      the-global-environment)
```

结果变成了 `Unbound variable a`。

为了弄清发生了什么，我给 `eval` 和 `apply` 加上了打印语句，再次运行。

[![25Y4MQ.png](https://z3.ax1x.com/2021/06/12/25Y4MQ.png)](https://imgtu.com/i/25Y4MQ)

第二个被 `eval` 的表达式居然是 `(lambda () (+ a 1))`？难道说 MIT-Scheme 对 operands 的求值顺序是从右向左的吗？如果是的话，那么对于原始的 `let` 形式的代码，`eval` 的顺序的确是 `(let ...)`，`(+ a (b))`，`(b)`，`b`。

继续试验，我分别用 MIT-Scheme 和 DrRacket 运行了如下代码。

[![25Y5rj.png](https://z3.ax1x.com/2021/06/12/25Y5rj.png)](https://imgtu.com/i/25Y5rj)

[![25Yfxg.png](https://z3.ax1x.com/2021/06/12/25Yfxg.png)](https://imgtu.com/i/25Yfxg)

到这里可以确定 MIT-Scheme 对 operands 的求值顺序是从右向左的了。Hmm，奇怪的 MIT-Scheme。不过，R5RS 的确没有规定不能这样做。想想也确实不应该对此作出规定，毕竟如果没有副作用，**"Substitution Model"** 的代换顺序并不会影响结果。

## 初步理解 `eval-apply`

```scheme
(eval '((lambda (a b) (+ a (b)))
        1
        (lambda () (+ a 1)))
      the-global-environment)
```

这是一个 application，将会调用 apply。

```scheme
(apply (eval '(lambda (a b) (+ a (b))) global-env)     ; 1.1
       (list (eval 1 global-env)                       ; 1.2
             (eval '(lambda () (+ a 1)) global-env)))  ; 1.3
```

三个 eval 将分别返回一个过程对象，1 和另一个过程对象。

进入 apply，apply 将调用 eval-sequence，参数分别为过程 1.1 的 body 和一个新的环境对象（记作 E1），这个环境对象继承自 the-global-environment，并且将数字 1 和过程对象 1.3 分别绑定到符号 a 和 b 上。

```scheme
(eval '(+ a (b)) <E1>)
```

同样地，这是一个 application，继续调用 apply。

```scheme
(apply (eval '+ <E1>)            ; 2.1
       (list (eval 'a <E1>)      ; 2.2
             (eval '(b) <E1>)))  ; 2.3
```

第一个 eval 调用 lookup-variable-value，在 the-global-environment 中找到 + 对应的 primitive 过程对象并返回，第二个 eval 将在 E1 中找到 a 对应的值 1 并返回。

第三个 eval 的表达式还是一个 application，再次调用 apply。

```scheme
(apply (eval 'b <E1>)  ; 3.1
       '())
```

这个 eval 将会在 E1 中找到 b 对应的过程对象：`('procedure () ((+ a 1)) the-global-environment)`，即 1.3。

然后进入这个 apply（上面的第二个 apply 还没有进入），apply 将调用 eval-sequence，参数分别为过程 1.3 的 body，即 `(+ a 1)` 和一个新的环境对象 E2。而这个新的环境对象，仍然继承自 the-global-environment，而不是继承自 E1，因为过程对象 1.3 中记录的环境正是 the-global-environment，即使我们现在（这个新环境被创建时）处于环境 E1 中。

因此，接下来调用 `(eval (+ a 1) <E2>)`，将会告诉我们 `Unbound variable a`。

事实上，这个陷阱在当初学习第三章，教材第一次介绍 **"Environment Model"** 时就已经掉进去过！上面的描述对应的 environment diagram 如下图。

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" width="596px" height="331px" viewBox="-0.5 -0.5 596 331" style="background-color: rgb(255, 255, 255);"><defs/><g><rect x="80" y="20" width="480" height="60" rx="9" ry="9" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 478px; height: 1px; padding-top: 50px; margin-left: 81px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; "><font face="Courier New">'+: ('primitive +)</font></div></div></div></foreignObject><text x="320" y="54" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">'+: ('primitive +)</text></switch></g><path d="M 30 60 L 73.63 60" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 78.88 60 L 71.88 63.5 L 73.63 60 L 71.88 56.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="20" y="20" width="60" height="40" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 40px; margin-left: 50px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: nowrap; "><font face="Courier New">global<br />env</font></div></div></div></foreignObject><text x="50" y="44" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">global...</text></switch></g><path d="M 230 120 L 229.4 89.19" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 229.3 83.94 L 232.94 90.87 L 229.4 89.19 L 225.94 91 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="80" y="120" width="300" height="60" rx="9" ry="9" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 298px; height: 1px; padding-top: 150px; margin-left: 81px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: normal; word-wrap: normal; "><font face="Courier New">'a: 1<br />'b: ('procedure () ... &lt;global-env&gt;)</font></div></div></div></foreignObject><text x="230" y="154" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">'a: 1...</text></switch></g><path d="M 445 150 L 445 170 L 445 150 L 445 163.63" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 445 168.88 L 441.5 161.88 L 445 163.63 L 448.5 161.88 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><ellipse cx="445" cy="135" rx="15" ry="15" fill="#ffffff" stroke="#000000" pointer-events="all"/><ellipse cx="445" cy="135" rx="11" ry="11" fill="none" stroke="#000000" pointer-events="all"/><path d="M 475 120 L 474.63 86.79" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 474.57 81.54 L 478.15 88.5 L 474.63 86.79 L 471.15 88.58 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><ellipse cx="475" cy="135" rx="15" ry="15" fill="#ffffff" stroke="#000000" pointer-events="all"/><ellipse cx="475" cy="135" rx="11" ry="11" fill="none" stroke="#000000" pointer-events="all"/><path d="M 360 158 L 385 158 Q 395 158 395 148 L 395 141.5 Q 395 135 405 135 L 423.63 135" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 428.88 135 L 421.88 138.5 L 423.63 135 L 421.88 131.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="385" y="170" width="120" height="40" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 190px; margin-left: 445px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: nowrap; "><font face="Courier New">parameters: ()<br />body: (+ a 1)<br /></font></div></div></div></foreignObject><text x="445" y="194" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">parameters: ()...</text></switch></g><path d="M 30 150 L 73.63 150" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 78.88 150 L 71.88 153.5 L 73.63 150 L 71.88 146.5 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="35" y="130" width="30" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 140px; margin-left: 50px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: nowrap; "><font face="Courier New">E1</font></div></div></div></foreignObject><text x="50" y="144" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">E1</text></switch></g><path d="M 510 230 L 510 155 L 510.51 87.99" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 510.55 82.74 L 514 89.76 L 510.51 87.99 L 507 89.71 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="445" y="230" width="130" height="50" rx="7.5" ry="7.5" fill="#ffffff" stroke="#000000" pointer-events="all"/><rect x="185" y="190" width="90" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 200px; margin-left: 230px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: nowrap; "><font face="Courier New" style="font-size: 14px">(+ a (b))</font></div></div></div></foreignObject><text x="230" y="204" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">(+ a (b))</text></switch></g><rect x="475" y="290" width="70" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 300px; margin-left: 510px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: nowrap; "><font face="Courier New" style="font-size: 14px">(+ a 1)</font></div></div></div></foreignObject><text x="510" y="304" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">(+ a 1)</text></switch></g><path d="M 395 259.5 L 438.63 259.5" fill="none" stroke="#000000" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 443.88 259.5 L 436.88 263 L 438.63 259.5 L 436.88 256 Z" fill="#000000" stroke="#000000" stroke-miterlimit="10" pointer-events="all"/><rect x="400" y="240" width="30" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject style="overflow: visible; text-align: left;" pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 250px; margin-left: 415px;"><div style="box-sizing: border-box; font-size: 0; text-align: center; "><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: #000000; line-height: 1.2; pointer-events: all; white-space: nowrap; "><font face="Courier New">E2</font></div></div></div></foreignObject><text x="415" y="254" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">E2</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://www.diagrams.net/doc/faq/svg-export-text-problems" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Viewer does not support full SVG 1.1</text></a></switch></svg>

而如果是使用 letrec：

```scheme
(letrec ((a 1)
         (b (lambda () (+ a 1))))
  (+ a b))
```

亦即转换为：

```scheme
(let ((a '*unassigned*)
      (b '*unassigned*))
  (set! a 1)
  (set! b (lambda (+ a 1)))
  (+ a b))
```

亦即：

```scheme
((lambda (a b)
   (set! a 1)
   (set! b (lambda () (+ a 1)))
   (+ a b))
 '*unassigned*
 '*unassigned*)
```

则上图中的过程对象记录的环境将指向 E1，而 E2 的父环境也将指向 E1，代码便能正常工作。

回顾上面的 eval-apply 的代换过程。eval 的工作即为对表达式求值。当表达式是过程调用（application）时，eval 分别对 operator 和 operands 求值，前者的结果是创建或查找到一个 procedure，后者的结果将作为这个 procedure 的 arguments；eval 接下来调用 apply，而 apply 的工作即为 Environment Model 中画新的环境框的动作（创建新环境，绑定 arguments 到 procedure 的 parameter 上），画完框之后工作又交回给 eval（在新环境中调用 eval 求值 procedure 的 body）。

虽然人不能踏进同一条河流两次，却能掉进同一个坑两次，为了避免自己第三次落入这个 Environment Model 的小陷阱，我写下了这篇笔记。其实理解这个最简单的 eval-apply 的过程并不困难，但是我知道，我还只是和这个计算机中的精灵见了个面，并没有真正地认识她。继续学习吧，为了成为 one of "The Grand Wizard"！

