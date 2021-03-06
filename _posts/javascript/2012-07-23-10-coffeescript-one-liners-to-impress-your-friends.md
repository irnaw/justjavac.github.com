---
layout: post
title: 10个让朋友对你刮目相看的CoffeeScript单行代码绝技
description: 10个让朋友对你刮目相看的CoffeeScript单行代码绝技
keywords: javascript, CoffeeScript
category : javascript
tags : [javascript, CoffeeScript, 单行代码]
---

或许你已经看过了[Marcus Kazmierczak](http://solog.co/)的这篇在HN上颇受欢迎的“[10个让朋友对你刮目相看的Scala单行代码绝技][2]”了，
尽管我对Scala并不了解（Java也是），但是这看起来还真不错，于是我也有点手痒，
想让我的朋友们也对我刮目相看一小下——不过不是从Java到Scala，我是从Javascript到CoffeeScript，
下面的例子都是基于[node.js](http://justjavac.iteye.com/blog/1485472)环境的。

## 1. 列表中的每项乘2

Marcus的第一个例子演示了map函数，我们可以使用range语法以及一个匿名函数来完成同样的事情：

    [1..10].map (i) -> i*2  

我们还有下面这个更易读的版本：

    i * 2 for i in [1..10]  

## 2. 数列求和

Javascript（以及CoffeeScript扩展）同样有原生的[map][4]以及[reduce][5]函数：

    [1..1000].reduce (t, s) -> t + s  
     
    (reduce == reduceLeft, 还有reduceRight)  

## 3. 检查字符串是否包含某个单词

这实在是再简单不过了，因为我们有some方法，只要数组中的任何元素满足条件它就会返回true：

    wordList = ["coffeescript", "eko", "play framework", "and stuff", "falsy"]  
    tweet = "This is an example tweet talking about javascript and stuff."  
      
    wordList.some (word) -> ~tweet.indexOf word  

下面的语句会返回匹配到的单词：

    wordList.filter (word) -> ~tweet.indexOf word  

～在CoffeeScript中并不是什么特别的操作符，这里我们使用了一个小技巧，它实际上就是按位取反操作符，
会对数值按位[进行取反][6]操作，在上面的例子里它相当于-x-1，这里我们用它来检查数组的下标是否大于-1，
因为-(-1)-1 == 0，会返回false。

## 4. 读取文件

使用客户端Javascript框架的用户会对下面的代码感到很亲切：

    fs.readFile 'data.txt', (err, data) -> fileText = data  

你还可以使用同步版本：

    fileText = fs.readFileSync('data.txt').toString()  

不过在node.js中，你只有在程序启动时才可以使用同步版本，其它时间你都应该使用异步版本。

## 5. 生日快乐

首先，先来一个Scala版本的映射版，不过我对字符串做了一点篡改：

    [1..4].map (i) -> console.log "Happy Birthday " + (if i is 3 then "dear Robert" else "to You")  

下面再来一个优化版，这个读起来更像伪代码了：

    console.log "Happy Birthday #{if i is 3 then "dear Robert" else "to You"}" for i in [1..4]  

## 6. 过滤数列

将一个数字序列过滤为两种类型，这已经很接近了：

    passed = []  
    failed = []  
    (if score > 60 then passed else failed).push score for score in [49, 58, 76, 82, 88, 90]  

(也可以使用filter，但是那样就不是一行了。。。）

## 7. 读取并解析一个XML Web service

XML是个神马东东？从没听过，让我们把它换成json，你可以使用[request](http://github.com/mikeal/request)库：

    request.get { uri:'path/to/api.json', json: true }, (err, r, body) -> results = body  

## 8. 找到一个数列的最小（最大）值

我们有非常棒的apply函数，它可以让你通过一个数组来调用拥有可变参数的函数：Math.max以及Math.min，
这两个函数都接受可变长度参数，比如Math.max 30, 10, 20返回30，让我们试试下面的代码：

    Math.max.apply @, [14, 35, -7, 46, 98] # 98  
    Math.min.apply @, [14, 35, -7, 46, 98] # -7  

## 9. 并行处理

这个还不行，你可以创建[子进程](http://nodejs.org/docs/v0.4.8/api/child_processes.html)并自己和它们进行通讯，
或者使用[WebWorkers API](https://github.com/pgriess/node-webworker)实现，让我们跳过这个。

## 10. 埃拉托斯特尼筛法

（译者注：埃拉托斯特尼筛法是古希腊数学家埃拉托斯特尼所提出的一种简单的判定素数的算法，详细介绍请参见维基百科）

一行可以搞定？

    sieve = (num) ->  
        numbers = [2..num]  
        while ((pos = numbers[0]) * pos) <= num  
            delete numbers[i] for n, i in numbers by pos  
            numbers.shift()  
        numbers.indexOf(num) > -1  

更新 (06/05): @dionyziz 发给了我这个更简洁的版本:

    primes = []  
    primes.push i for i in [2..100] when not (j for j in primes when i % j == 0).length  

现在我们可以像原始版一样用一行来完成判定了：

    (n) -> (p.push i for i in [2..n] when not (j for j in (p or p=[]) when i%j == 0)[0]) and n in p  

或者是下面这样：

    (n) -> (p.push i for i in [2..n] when !(p or p=[]).some((j) -> i%j is 0)) and n in p  

## 11. 奖励

最后，再奖励你们一个你肯定没见过的最易读的[fizzbuzz](http://en.wikipedia.org/wiki/Bizz_buzz)（数3，数5游戏）版本：

    "#{if i%3 is 0 then 'fizz' else ''}#{if i%5 is 0 then 'buzz' else ''}" or i for i in [1..100]  

更新：基于satyr的提示，这里是一个更简单，也更取巧的版本：

    ['fizz' unless i%3] + ['buzz' unless i%5] or i for i in [1..100]  

如果你对数组使用+操作符，它会变成一个字符串，当数组中包含undefined或是null时，[].toString()和[].join(',')是一个效果，这在Javascript中也同样有效（[undefined] + "b" === "b"）

## 结论

我很吃惊这些例子中的一些语法居然和Scala如此接近，而我原以为它应该是属于另外一个星球的编程语言。

你可以在这里了解更多关于[CoffeeScript](http://jashkenas.github.com/coffee-script/)的知识，
以及[代码片段](http://rosettacode.org/wiki/Category:CoffeeScript)，
也欢迎在Twitter上Follow [@ricardobeat](http://twitter.com/ricardobeat).

[2]: http://solog.co/47/10-scala-one-liners-to-impress-your-friends/
[4]: http://en.wikipedia.org/wiki/Map_%28higher-order_function%29
[5]: http://en.wikipedia.org/wiki/Fold_%28higher-order_function%29
[6]: https://developer.mozilla.org/en/JavaScript/Reference/Operators/Bitwise_Operators 
