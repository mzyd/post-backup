---
title: Emacs Lisp Basics
date: 2021-06-05 19:47:26
catalog: true
subtitle: emacs lisp 的基础语法
header-img: 顶部背景图片
top: false
tags:
- Emacs
- elisp
categories:
- Emacs
- elisp
---

## Emacs Lisp Basics

- 整篇文章都是跟着李杀大神的教程来的，我发现他的教程比 `learn x in y minutes` 好多了
- 而且可以继续深入学习
- 原文地址: http://ergoemacs.org/emacs/elisp_basics.html
- 这里使用的是 lisp-interaction-mode，只需要在表达式后面键入 C-j 就可以在下一行输出结果了
- 或者使用 emacs-lisp-mode 在函数尾部输入 C-x C-e 执行函数，可以在下面看到结果
- spacemacs 使用 `, e f` 快捷键和 C-x C-e 是一样的

### Printing
- 这里只有 `%S` 特殊一点
```elisp
(message "h1") ; "h1"

;; printing variable values
(message "Her age is : %d" 16) ; "Her age is : 16"
(message "Her name is: %s" "Vicky") ; "Her name is: Vicky"

;; %S 可以输出任意 lisp 表达式
(message "My list is: %S" (list 8 2 3)) ; "My list is: (8 2 3)"
```

### Arithmetic （计算）
- lisp 语言把符号写在前面的好处之一是符号只用写一遍，后面随便加参数
``` elisp
(+ 4 5 1)  ; 10
(- 9 2)    ; 7
(- 9 2 3)  ; 4
(* 2 3)    ; 6
(* 2 3 2)  ; 12

;; integer part of quotient
(/ 7 2)    ; 3
(/ 7 2.)    ; 3

;; division
(/ 7 2.0)  ; 3.5

;; mod, remainder
(% 7 4)    ; 3

;; power; exponential
(expt 2 3) ; 8

;; warning : 一位十进制数，如 2. 需要在点后加一个零，
;; 像这样：2.0。例如， (/ 7 2.) 返回 3，而不是 3.5。

;; 3. 是整数, 3.0 是一个小数
;; 以 p 结尾的函数名通常会返回 true 或者 false
;; p 代表这是一个 "谓词"（predicate）
;; 在 lisp 里， t 表示真，nil 表示假
(integerp 3.) ;  t
(floatp 3.) ;  nil
(floatp 3.0) ;  t

;; Convert Float/Integer
;; int to float
(float 3) ; 3.0
(truncate 3.3) ; 3
(floor 3.3) ; 3
(ceiling 3.3) ; 4
(round 3.4) ; 3

;; Convert String and Number
(string-to-number "3") ; 3
(number-to-string 3) ; "3"

;; 也可以使用 format 来完成字符串和数字的转换，后面会说
```

### True, False
- 在 elisp里，nil 是 false，其他都是 true
- nil == (), () 是空列表
``` elisp
;; all the following are false. They all evaluate to nil
(if nil "yes" "no") ;  "no"
(if () "yes" "no") ;  "no"
(if '() "yes" "no") ;  "no"
(if (list) "yes" "no") ;  "no", because (list) eval to a empty list, same as ()

;; t 用来表示 真
(if t "yes" "no") ;  "yes"
(if 0 "yes" "no") ;  "yes"
(if "" "yes" "no") ;  "yes"
(if [] "yes" "no") ;  "yes". The [] is vector of 0 elements

;; elisp 中没有“布尔数据类型”。只要记住 nil 和 empty list () 都是假的，其他的都是真的。

;; Boolean Functions

;; Here's and and or.

(and t nil) ;  nil
(or t nil) ;  t

;; can take multiple args
(and t nil t t t t) ;  nil

(< 3 4) ; less than
(> 3 4) ; greater than

(<= 3 4) ; less or equal to
(>= 3 4) ; greater or equal to

(= 3 3)   ;  t
(= 3 3.00000000000000001) ;  t

(/= 3 4) ; not equal. ⇒ t
```

### Comparing string
- 注意 string-equal 是专门对比字符串和符号的函数
```elisp
(equal "abc" "abc") ;  t

;; dedicated function for comparing string
(string-equal "abc" "abc") ;  t

(string-equal "abc" "Abc") ;  nil. Case matters

;; can be used to compare string and symbol
(string-equal "abc" 'abc) ;  t
```

- 对一般的相等测试，使用 equal。它测试两个值是否具有相同的数据类型和值。
``` elisp
(equal 3 3) ;  t
(equal 3.0 3.0) ;  t

(equal 3 3.0) ;  nil. Because datatype doesn't match.

;; test equality of lists
(equal '(3 4 5) '(3 4 5))  ;  t
(equal '(3 4 5) '(3 4 "5")) ;  nil

;; test equality of strings
(equal "e" "e") ;  t

;; test equality of symbols
(equal 'abc 'abc) ;  t

;; 还有函数 eq，如果两个 args 是同一个 Lisp 对象，它会返回 t。
;; 这通常不是您想要的。 (eq "e" "e") 返回 nil。
(eq "e" "e") ; nil

;; To test for inequality, the /= is for numbers only, and doesn't work for strings and other lisp data. Use not to negate your equality test, like this:
;; 为了测试不等式， /= 仅适用于数字，不适用于字符串和其他 lisp 数据。使用 not 否定您的相等性测试，如下所示：
(not (= 3 4)) ; t
(/= 3 4) ; t
;; General way to test inequality. 一般测试不等式的方式:
(not (equal 3 4)) ; t
```

### even, odd
- 可以封装成一个函数
``` elisp
(= (% n 2) 0) ; test even

(= (% n 2) 1) ; test odd

(defun test-even (n)
  (= (% n 2) 0))

(test-even 4) ; t
```

### Variables
- Global Variables
``` elisp
;; setq is used to set variables. Variables need not be declared, and is global.
(setq x 1) ; assign 1 to x
(setq a 3 b 2 c 7) ; multiple assignment
```
- Local Variables
``` elisp
;; To define local variables, use let. The form is:
(let (var1 var2 …) body)

(let (a b)
(setq a 3)
(setq b 4)
  (+ a b)) ; 7

;; 以下代码控制台只会输出 456，因为只返回最后一个值，查看 *Message* buffer 就能看到 123 其实也被输出了
(let (a b)
  (setq a "123")
  (setq b "456")
  (message a)
  (message b)) ; "123"


;; Another form of let is this:
;; This form lets you set values to variable without using many setq in the body. This form is convenient if you just have a few simple local vars with known values.
(let ((var1 val1) (var2 val2) …) body)

;; 在列表内给参数单独赋值
(let ((a 3) (b 4))
  (+ a b)) ;  7
```

### If Then Else
- `when` 就是没有 `else` 的 `if`
``` elisp
;; The form for “if” expression is:
(if test body)
;; or
(if test true_body false_body)

(if (< 3 2) 7 8 ) ; 8

;; no false expression, return nil
(if (< 3 2) (message "yes") ) ; nil

;; If you do not need a “else” part, you should use the function when instead, because it is more clear. The form is:
(when test expr1 expr2 …)

;; Its meaning is the same as
(if test (progn expr1 expr2 …))
```


### Block of Expressions
- let 也有组合表达式的功能
``` lisp
;; Sometimes you need to group several expressions together as one single expression. This can be done with progn.
;; 这句代码在控制台只能输出 "b", 同样是去 message buffer就能看到 a 了, 控制台只输出最后一个返回值
(progn (message "a") (message "b"))
;; is equivalent to
(message "a") (message "b")

;; The purpose of (progn …) is similar to a block of code {…} in C-like languages. It is used to group together a bunch of expressions into one single parenthesized expression. Most of the time it's used inside “if”.
(if something
  (progn ; true
    …
    )
  (progn
    … ; there is else
    ))

;; progn returns the last expression in its body.
(progn 3 4 ) ; 4
```

### Loop
- Most basic loop in elisp is with while.
``` elisp
(setq x 0)
(while (< x 4)
(print (format "number is %d" x))
(setq x (1+ x)))

;; inserts Unicode chars 32 to 126
(let ((x 32))
  (while (< x 127)
  (insert-char x)
  (setq x (+ x 1))))
;; !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~

;; Usually it's better to use dolist or dotimes.
```

### Define a Function
- 当一个函数被调用时，永远都是返回函数体内的最后一句，`lisp` 里面没有 `return statement`
``` elisp
;; Basic function definition is of the form:
(defun function-name (param1 param2 …) doc_string body)

(defun my-function ()
  "testing"
  (message "Yay!"))
```
- 命令是 `emacs` 用户可以通过 `execute-extended-command【Alt+x】`调用的函数。
- 当一个函数也是一个命令时，我们说该函数可用于交互使用。
- 要使函数可用于交互式使用，请在文档字符串之后添加 (interactive)。
``` elisp

;; 以下代码可以使用 execute-extended-command 调用
(defun yay ()
"Insert “Yay!” at cursor position."
(interactive)
(insert "Yay!"))

;; Here is a function definition template that majority of elisp commands follow:
(defun myCommand ()
"One sentence summary of what this command do. More detailed documentation here."
  (interactive)
  (let (localVar1 localVar2 …)
    ; do something here …
    ; …
    ; last expression is returned
  ))
  ```
