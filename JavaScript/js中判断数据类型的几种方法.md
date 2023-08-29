# JS中判断数据类型的几种方法
[引用自知乎](https://zhuanlan.zhihu.com/p/453520879?utm_id=0)  

说说JavaScript中判断数据类型的几种方法typeoftypeof一般用来判断基本数据类型，除了判断null会输出"object"，其它都是正确的typeof判断引用数据类型时，除了判断函数会输出"function",其它都是输出"object"

console.log(typeof 6); // 'number' console.log(typeof true); // 'boolean' console.log(typeof 'nanjiu'); // 'string' console.log(typeof []); // 'object' []数组的数据类型在 typeof 中被解释为 object console.log(typeof function(){}); // 'function' console.log(typeof {}); // 'object' console.log(typeof undefined); // 'undefined' console.log(typeof null); // 'object' null 的数据类型被 typeof 解释为 object 

对于引用数据类型的判断，使用typeof并不准确，所以可以使用instanceof来判断引用数据类型

instanceof

Instanceof 可以准确的判断引用数据类型，它的原理是检测构造函数的prototype属性是否在某个实例对象的原型链上

原型知识点具体可以看我之前的文章：你一定要懂的JavaScript之原型与原型链

语法：

object instanceof constructor console.log(6 instanceof Number); // false console.log(true instanceof Boolean); // false console.log('nanjiu' instanceof String); // false console.log([] instanceof Array); // true console.log(function(){} instanceof Function); // true console.log({} instanceof Object); // true 

constructor(构造函数)

当一个函数被定义时，JS引擎会为函数添加prototype属性，然后在prototype属性上添加一个constructor属性，并让其指向该函数。



当执行 let f = new F()时，F被当成了构造函数，f是F的实例对象，此时F原型上的constructor属性传递到了f上，所以f.constructor===F

function F(){} let f = new F() f.constructor === F // true new Number(1).constructor === Number //true new Function().constructor === Function // true true.constructor === Boolean //true ''.constructor === String // true new Date().constructor === Date // true [].constructor === Array 

⚠️注意：

null和undefined是无效的对象，所以他们不会有constructor属性函数的construct是不稳定的，主要是因为开发者可以重写prototype，原有的construction引用会丢失，constructor会默认为Object

function F(){} F.prototype = {} let f = new F() f.constructor === F // false console.log(f.constructor) //function Object(){..} 

为什么会变成Object？

因为prototype被重新赋值的是一个{}，{}是new Object()的字面量，因此 new Object()会将 Object 原型上的constructor 传递给 { }，也就是 Object 本身。

因此，为了规范开发，在重写对象原型时一般都需要重新给 constructor 赋值，以保证对象实例的类型不被篡改。

Object.prototype.toString.call()

toString() 是 Object 的原型方法，调用该方法，默认返回当前对象的 [[Class]] 。这是一个内部属性，其格式为 [object Xxx] ，其中 Xxx 就是对象的类型。

对于 Object 对象，直接调用 toString() 就能返回 [object Object] 。而对于其他对象，则需要通过 call / apply 来调用才能返回正确的类型信息。

Object.prototype.toString.call('') ; // [object String] Object.prototype.toString.call(1) ; // [object Number] Object.prototype.toString.call(true) ; // [object Boolean] Object.prototype.toString.call(Symbol()); //[object Symbol] Object.prototype.toString.call(undefined) ; // [object Undefined] Object.prototype.toString.call(null) ; // [object Null] Object.prototype.toString.call(new Function()) ; // [object Function] Object.prototype.toString.call(new Date()) ; // [object Date] Object.prototype.toString.call([]) ; // [object Array] Object.prototype.toString.call(new RegExp()) ; // [object RegExp] Object.prototype.toString.call(new Error()) ; // [object Error] Object.prototype.toString.call(document) ; // [object HTMLDocument] Object.prototype.toString.call(window) ; //[object global] window 是全局对象 global 的引用

