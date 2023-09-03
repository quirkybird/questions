# 手动实现一个 new 操作符命令的功能

## new 操作符做了什么？

1. 创建一个默认的空对象
2. 绑定对象的原型到构造函数的原型上，类似有 obj.**proto** = Constructor.prototype
3. 执行构造函数，并将默认创建的空对象设置为构造函数的执行上下文
4. 判断构造函数是否有返回对象，如果有，就返回设置的对象，如果没有，则返回默认对象

## 代码部分

```javascript
function new(constructor, ...args) {
  // 创建一个对象并绑定原型
  const obj = Object.create(constructor.prototype)

  // 执行构造函数并绑定上下文

  const result = constructor.apply(obj, ...arg)

  // 判断构造函数返回是对象类型
  if(typeof result === 'object' && typeof result !== null) return result


  return obj
}
```

## 测试用例

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.prototype.sayHello = function () {
  console.log("Hello, I am " + this.name);
};
}

// 使用 new 创建 Person 对象
const person = new(Person, "John", 25);

console.log(person.name); // 输出: John
console.log(person.age); // 输出: 25
person.sayHello(); // 输出: Hello, I am John
```
