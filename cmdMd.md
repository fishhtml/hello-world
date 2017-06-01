#方法、this的讲解、apply与call的理解
标签（空格分隔）： javascript

---
#方法与函数
**在一个对象中绑定函数，称为这个对象的方法。**


----------


#this的各种坑！！！
## 1. 先来个正常的,**this指向执行环境的对象**
```
var xiaoming = {
    name: '小明',
    birth: 1990,
    age: function () {
        var y = new Date().getFullYear();
        return y - this.birth;
    }
};
```
```
xiaoming.age; // function xiaoming.age()
xiaoming.age(); // 今年调用是25,明年调用就变成26了
```
## 2. “**this指向执行环境的对象**”


```
 function getAge() {
    var y = new Date().getFullYear();
    return y - this.birth;
}
var xiaoming = {
    name: '小明',
    birth: 1990,
    age: getAge
};
```
```
xiaoming.age(); // 25, 正常结果
getAge(); // NaN
```
**`function getAge`是绑定在`window`下的函数，就相当于`window`的方法，`this`指向`window`理所当然。**

--------------

`getAge`此时是**回调函数**。等同于如下写法：(**与1中一样**)
```
var xiaoming = {
    name: '小明',
    birth: 1990,
    age:  function getAge() {
    var y = new Date().getFullYear();
    return y - this.birth;
	
};
```

-------------------------------------------------------------------

与此类似：
```
var fn = xiaoming.age; // 先拿到xiaoming的age函数
fn(); // NaN
```
在**非严格模式**下，变量未被定义就被调用，返回`NaN`。(没毛病)

------------------------------------------

```
'use strict';

var xiaoming = {
    name: '小明',
    birth: 1990,
    age: function () {
        var y = new Date().getFullYear();
        return y - this.birth;
    }
};

var fn = xiaoming.age;
fn(); // Uncaught TypeError: Cannot read property 'birth' of undefined
```
在**严格模式**下，变量未被定义就被调用，返回`undefined`(**指向Undefined对象**)。(没毛病)

-----------------------------------------

总之：正确的姿势是**`object.xxx()`的调用指向正确**
```
xiaoming.age()//this指向xiaoming
```
## 3. 修复方法。用一个`that`变量首先捕获`this`：
```
 'use strict';

var xiaoming = {
    name: '小明',
    birth: 1990,
    age: function () {
        var that = this; // 在方法内部一开始就捕获this
        function getAgeFromBirth() {
            var y = new Date().getFullYear();
            return y - that.birth; // 用that而不是this
        }
        //定义其他方法,而不是把所有语句都堆到一个方法中
        function getAgeFromBirth(){
        	var return that.birth;
    	}
        return getAgeFromBirth();
    }
};
```
**用`var that = this`;，你就可以放心地在方法内部定义其他函数，而不是把所有语句都堆到一个方法中。**
```
xiaoming.age(); // 25
```

--------------------------------------------

## 4. 用`arguments.callee()`代替`this`也是**不可以**的。
`arguments.callee()`指向**当前调用的函数**，常用在匿名函数里指代该匿名函数。


----------


----------


----------


#`apply()`、`call()`、`bind()`理解
## 1. **要指定函数的this指向哪个对象，可以用`函数本身`的`apply()`方法**

> `apply` `vt.` “应用”的意思。`apply the Function to Object.`**(将函数应用于某对象)** 
> `call` `vt.vi.`"需要"的意思。`call for the Object.`**(需要某对象来运行这个函数)**

简单粗暴的这样理解，便于记忆。

> `apply(thisObj,args)`
> `thisObj`表示函数内`this`所指代的对象(Obj)（是谁），备选值：`this` `null` `object实例`

> - `thisObj`为`this`时，调用`apply()`的函数中**`this`所指代的对象**

> - `thisObj`为`null`时，`thisObj`使用**全局对象**

> - `thisObj`为`object`实例时，很常见

> `args`表示传入的参数，可以是`argument`和`Array实例`
> 返回`function`执行的值

例如：
```
//将默认的Object.toString()应用到对象o上，以覆盖对象o中自定义的toString()方法。
Object.prototype.toString().apply(o);
```
```
//调用Math.max()方法来查找数组中的最大元素
var data = [1,2,3,4,5,6];
Max.max.apply(null,data);
```
```
window.color = "red";
var o = {
    color:"blue";
    }
function sayColor(){
    alert(this.color);
    }
sayColor(); //red
sayColor().apply(this);//red
sayColor().apply(window);//red
sayColor().apply(o);;//blue
```
> `call()`与`apply()`类似
> `call(thisObj,arg1,arg2,......)`

> - `arg1,arg2,......`表示按照顺序传入参数。 
## 2. `bind()` `ES5新增`
> `bind` vi. “捆绑”的意思。`bind the Function to the Object.`**(将函数绑入对象里面，函数此时成了该对象的方法,也就是说，此时函数内的`this`指向绑定的对象`o`）**

> `bind(o,arg1,arg2,....)`

例如：
```
function getName() {
    console.log(this.name);
    }
var o = {
    name:xiaoxiao;
    age:18;
    };
var result=getName().bind(o);
result();//xiaoxiao
getName();//undefined or NaN
```
参数可以分两次传递：
```
//假设f为一个函数
var g = f.bind(0,1,2);
//此时g为一个新函数,g(3)等价于：
f.call(0,1,2,3);

```
```
var o1={
    x:1,
    getplus:function(y,z){
      return this.x+y+z;
    }
  }
  var o2={
    x:11,
    getplus:function(y,z){
      return this.x*y;
    }
  }
  //o1.getplus需要2个参数，分两次传入
  var result=o1.getplus.bind(o2,1);
  //得到的result是函数，函数result需要另一个参数才能计算x+y+z的值
  result(3);//得出计算结果
  var resultapply=o1.getplus.call(o2,1,2);//与上面两步得出的结果一样
```
使`ES3`也兼容`bind()`的方法：
```
//如果方法不存在，就在原型里添加
//提示：bind()返回的是函数
if(!Function.prototype.bind){
    Function.prototype.bind=function(o /*,args*/){
        var self=this,boundArgs=arguments;
            //**此时Function是对象，this指向Funtion本身**，并把它保存起来
            //arguments也保存起来，这里的arguments是第一次传入的参数，所以下面for循环从第二个参数开始
        return function(){
            var args=[],i;
            for(i=1;i<boundArgs.length;i++) args.push(boundArgs[i]);
                //遍历第一次传入的参数，从第二位开始（第一位是对象）
            for(i=0;i<arguments.length;i++) args.push(arguments[i]);
                //遍历第二次传入的参数，全部遍历
            return self.apply(o,args);
        }
    }
}
```
解释：self的值是指向function的指针。
```
var result=o1.getplus.bind(o2,1);
```
其中`o1.getplus`方法（函数）是图中红色的`Function对象`，而self指针是图中的小`this`。
