# Javascript features

-----------------------------------------------------------------------------------------------

## 优先解析的函数式声明.
- Javascript优先解释函数式声明 function pointY(){},例如:
        
        console.log(pointX,pointY); //undefined [Function: pointY]
        var pointX = function(){};        
        function pointY(){}      

- 对比:
        
        var pointX = function(){};
        console.log(pointX,pointY); //[Function] [Function: pointY]
                     
        function pointY(){}
        
- **不管函数式声明写在哪里javascript都是在其他语句前先解析的. 而其他语句是顺序解析,如何声明前使用会认为是undefined.**
        
---------------------------------------------------------------------------------------------------
        
## 立即执行函数匿名函数.
- 上面的例子稍作修改
        
        console.log(function pointX(){return},function pointY(){return}());  //[Function: pointX] undefined

- 得到`underfined`是因为函数名后加`()`函数中return没有返回任何值. **()意思是立即执行这个函数.**

        console.log(function(){return},function(){return 100}());  //[Function] 100

- 原来的pointX和pointY名字都去掉了,变成匿名声明方式的匿名函数. 

----------------------------------------------------------------------------------------------------------
## 闭包内变量的作用域

-  上面的例子再变一种写法试试

       function pointX(){}();  //SyntaxError: Unexpected token )
       console.log(func);

- 这样写是报语法错误的,但是这样两种写法就没问题.

        (function pointX(){console.log("pointX")})();   //pointX
        (function pointY(){console.log("pointY")}());   //pointY
        //console.log(pointX);                          //ReferenceError: pointX is not defined

- 包起来了能执行了,但是包外面访问不到了(注释的第3句). 要想能就要给它一个能访问的句柄,或者说找到它的指针.给它一个引用标示.反正怎么说都可以就是把它的地址找到.

        var pointX = (function pointX(){
            console.log("pointX");                                   //pointX
            return "object pointX";
        })();   //pointX
        var pointY = (function pointY(){console.log("pointY")}());   //pointY
        
        console.log(pointX,pointY);                                  //object pointX undefined
        
- **对于pointX包里的变量包外根本访问不到除了return的值.** 所以变量名和包内函数同名根本不会报错 对于pointY没有return值所有返回的是`undefined`
  如果去掉两个包内或者外面的`()`会怎么样呢? 应该又回到第一部分概念两个`[Function]`.
  
- **没有`()`仅仅是函数定义的引用,不执行函数.得到[Function], 有`()`是执行函数返回`return`值**

- 如果闭包没有传入外部参数又没有return返回任何数据或者有return但是没有任何变量接受存储return的数据,那包就毫无意义**

        (function pointX(){
            console.log("pointX");                                   //pointX
            return "object pointX";
        })();

- 对于这段代码外的代码来说,这段代码没用.(其实它引用外部的方法`console.log()`,是全局方法)

## 
http://mp.weixin.qq.com/s?__biz=MjM5MzMyNzg0MA==&mid=400197911&idx=2&sn=25ceb819a04d99a1eea27eeea94ae547&scene=5&srcid=1103KpQHnnBen7rqN25dTb71#wechat_redirect

## 不是new不是点(.)调用的函数中this全部都是指向全局变量.

参考文章: http://www.cnblogs.com/isaboy/archive/2015/10/29/javascript_this.html

- 直接运行函数,无new和(.)

        function pointX(){
            console.log(this===global);
        }
        pointX(); //true

- new运行函数

        function pointX(){
            console.log(this===global);
        }
        
        new pointX();  //false

- 点(.)运行函数

        function pointX(){
            console.log(this===global);
        }
        var obj = {};
        obj.pointX = pointX;
        obj.pointX();   //false

- 匿名立即执行函数

        function(){
           return console.log(this===global);  //true
        }());


## **关于JavaScript函数执行环境的过程，IBM developerworks文档库中的一段描述感觉很不错，摘抄如下：**
  
  　　“JavaScript 中的函数既可以被当作普通函数执行，也可以作为对象的方法执行，这是导致 this 
  含义如此丰富的主要原因。
  
- 一个函数被执行时，会创建一个执行环境（ExecutionContext），函数的所有的行为均发生在此执行环境中，
- 构建该执行环境时，JavaScript 首先会创建arguments变量，其中包含调用函数时传入的参数。
- 接下来创建作用域链。然后初始化变量，
- 首先初始化函数的形参表，值为 arguments变量中对应的值,如果 arguments变量中没有对应值，则该形参初始化为 undefined。
- 如果该函数中含有内部函数，则初始化这些内部函数。
- 如果没有，继续初始化该函数内定义的局部变量，需要注意的是此时这些变量初始化为 undefined，
- 其赋值操作在执行环境（ExecutionContext）创建成功后，函数执行时才会执行，
  
  这点对于我们理解 JavaScript 中的变量作用域非常重要，鉴于篇幅，我们先不在这里讨论这个话题。
  最后为 this变量赋值，如前所述，会根据函数调用方式的不同，赋给 this全局对象，当前对象等。
  至此函数的执行环境（ExecutionContext）创建成功，函数开始逐行执行，所需变量均从之前构建好的执行环境（ExecutionContext）中读取。”
  
  　　理解这段话对于理解Javascript函数将大有好处。　**