---
layout: post
title: 深入理解 javaScript “this" 
categories:
- Programming
tags:
- javascript
---

在Javascript里有个特殊的值: `this`. 对于前端工程师来说，`this`的名声不是很好，在平常的使用过程中会带来许多的困惑跟烦恼。

在Javascript里每个函数都有自己的属性，当函数执行的时候，`this`的值就是当前调用函数的对象。

###  this定义在函数里


~~~ javascript
function normalFunction(){
console.log( this === window ) // true
}
normalFunction()

~~~

在严格模式下，`this`的值为 undefined

~~~ javascript
function strictFunction(){
'use strict'
console.log( this === undefined ) // true
}

strictFunction()
~~~


一个常见的jQuery代码

~~~ javascript
$('button').click(function(event){
console.log( $(this).prop('name') );
})
~~~

`$(this)` 就是对象`$('button')`的值，因为是 $('button')调用的click方法，关于这个在后面会详细解释。

### this定义在方法里

~~~ javascript
var person = {
  method: function(){
	console.log(this === person)// true
  }
}

person.method();
~~~

### this定义在全局环境里

~~~ javascript
var firstName = `jason`, lastName = 'weng';
function showFullName(){
	console.log( this.firstName + lastName ); 
}
showFullName()   // jasonweng
~~~
showFullName函数里的this只的值是window 对象，因为函数showFullName是定义在全局作用域里的。所以调用showFullName()的window对象，showFullName()相当于window.showFullName()，只是我们平常把window省略掉了。


~~~ javascript
var person = {
  firstName = 'zhang',
  lastName = 'wenguang',
  showFullName: function(){
	console.log( this.firstName + lastName ); 
  }
}

person.showFullName(); // zhangwenguang
~~~

当我们调用person.showFullName()的时候，this的值就是调用showFullName方法的对象，也就是person对象，那么this.firstName相当于person.firstName


### this的一些小陷阱

小提示：平常我们在写代码的时候可以使用 'use strict' 模式，因为在这个模式下 this在函数里值是undefined。有些情况下会报错，会更安全些。

- ### 创建构造函数的时候忘记了 new

~~~ javascript
function Person(firstName,lastName) {
  this.firstName =firstName;
  this.lastName = lastName;
}
var JW = Person('jason','weng');
~~~
这个时候我们同时也创建了2个全局变量firstName跟lastName。
但是如果你在定义函数的时候使用 'use strict' 模式

~~~ javascript
function Person(firstName,lastName) {
 'use strict' 
  this.firstName =firstName;
  this.lastName = lastName;
}
var JW = Person('jason','weng');
~~~

就会抛出`TypeError: Cannot set property 'firstName' of undefined`。

- ### 回调函数

假设我们有一个函数

~~~ javascript
function imCallback(func){
	func();
}

~~~

~~~ javascript
var counter = {
	count: 0,
	add: function(){
		this.count++;
	}
}

imCallback( counter.add );
console.log(counter.count) // 0
console.log(count) // NaN( undefined++ )

~~~

当我们把counter.add当做回调函数来调用的时候，this会指向window，一个全局的count变量将会被创建，值为NaN。
使用[bind()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)可以解决这个问题

~~~ javascript
var counter = {
	count: 0,
	add: function(){
		this.count++;
	}
}

imCallback( counter.add.bind(counter) );
console.log(counter.count) // 1

~~~

- 闭包里的this
当我们创建一个闭包的时候，在闭包里使用this是访问不到外部函数的this的值的。

先看一段代码

~~~ javascript
var obj = {
        name: 'Jason',
        friends: [ 'Aaron', 'Azuma' ],
        loop: function () {
            'use strict';
            // 在这里this还是指向obj对象的
            this.friends.forEach(
                function (friend) {
                	// 在这个匿名函数里，this不在指向obj对象。
                    console.log(this.name+' knows '+friend);
                }
            );
        }
    };
    obj.loop();
    // TypeError: Cannot read property 'name' of undefined
~~~

方法1

在forEach之前把指向obj对象的this赋值给另外一个变量：self

~~~ javascript
loop: function () {
            'use strict';
            
            var self = this;
            
            this.friends.forEach(function (friend) {
                    console.log(self.name+' knows '+friend);
                }
            );
        }

~~~

方法2

使用bind

~~~ javascript
loop: function () {
            'use strict';            
            this.friends.forEach(function (friend) {
                    console.log(self.name+' knows '+friend);
                }.bind(this));
        }

~~~











