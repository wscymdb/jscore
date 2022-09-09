# JavaScript高级

# 1.函数

## 1.1. this绑定规则优先级

* 隐式绑定的this    ``大于``    默认的this
* 显式绑定的this   ``大于``   隐式绑定的this
* new绑定this   ``大于``   隐式绑定的this
* new绑定的this   ``大于``   bind绑定的优先级
* bind绑定的this   ``大于``  apply、call绑定的this



``备注：``

* new关键字不可以和call/apply一起使用，但是可以和bind一起使用
  * 因为bind返回的是一个新的函数
* 隐式绑定： 浏览器执行的时候自动绑定，比如之际调用一个函数，this就是window
* 显示绑定： 手动的设置this的指向，比如 foo.call('aa')

```javascript
function foo() {
    console.log(this)
}
// 显示绑定的优先级高于隐式绑定
// 1.1  apply,call 高于默认绑定
let obj = {foo:foo}
foo.call(obj)  // {foo: foo} 

// bind绑定高于默认绑定
let bar = foo.bind('aaa')
let obj = {name:'zd', baz:bar}
obj.baz()  // aaa

// new绑定的优先级高于隐式绑定
  let obj = {
        name: "zs",
        baz: foo,
      };
  obj.baz(); // {name: 'zs', baz: ƒ}
  new obj.baz(); // {}

// new绑定的优先级高于隐式绑定
// bind 会返回一个新的函数
  let bindFn = foo.bind("aaa");
  bindFn(); // aaa
  new bindFn(); // {}

// bind绑定的this 高于 apply、call绑定的this
  let bindFn = foo.bind("aaa");
  bindFn(); // aaa
  bindFn.call("bbb"); // aaa
  bindFn.apply("bbb"); // aaa
```





## 1.2.this绑定之外的情况

* 非严格模式下，显示的将this绑定成null或undefined，this的指向是默认绑定（window）
* 严格模式下，显示的将this指向null或undefined，this指向就是null或者undefined

```javascript

function foo() {
    console.log(this);
}

foo.call(null);  //  window对象
foo.call(undefined);  // window对象
foo.call("q");  //  包装类 String{q}  就是字符串q

// 
```

* 非严格模式下，创建一个函数的间接调用，this的指向是默认指向
* 由以下案列可知，赋值语句的返回值是要赋的值

```javascript
let obj1 = {
    name: "obbj1",
    foo: function () {
        console.log(this);
    },
};

let obj2 = {};
console.log((obj2.foo = obj1.foo)); // foo函数
// 因为赋值过程的返回值是当前函数，直接调用就是全局调用，this指向window
(obj2.foo = obj1.foo)(); // this指向window
```

## 1.3.箭头函数

* 箭头函数不会绑定this和arguments

* 箭头函数不能用作构造函数

* 如果函数体只有一行代码，默认值返回的是一个对象，必须用括号将对象包裹

  * ```java
    // 如果不用括号包裹   （） => {}  这样js引擎解析不出来
    [1,2,3].map(item => ({a:1,b:2})) 
    ```

* 箭头函数中没有this绑定

  * ```javascript
    // 那为什么再箭头函数中可以使用this
    // 因为再当前作用域中没有会向上一级作用域找
    let foo = () => {
        console.log(this)
    }
    // 正常情况下，bind绑定this -> aaa
    // 但是fn的this是window，并没有替换成aaa
    // 由此可见 箭头函数中没有this
    let fn = foo.bind('aaa')
    fn()   // window对象
    
    ```



## 1.4.函数的属性

* name ：表示当前函数的函数名
* length：返回当前函数的参数（形参）
  * rest（剩余参数）参数是不参与参数的个数

## 1.5.arguments

* arguments是一个类数组对象

arguments转换成数组

```javascript
foo(1,2,3,4)
function foo() {
    let arr；
    //方法0 遍历
    // 方法一
    arr = Array.from(arguments)
    // 方法二
    arr = [...arguments]
    // 方法三
    // slice返回的是截取后的数组，没有参数，截取全部
    arr = [].slice.apply(arguments)
}
```

## 1.6.纯函数

函数式编程中有一个非常重要的概念叫``纯函数``，JavaScript符合函数式编程的范式

纯函数的维基百科定义

* 在程序设计中，若有一个函数符合以下的条件，那么这个函数就被称之为``纯函数``
* 此函数在``相同的输入值时``，``产生相同的输出``
* 函数的``输入和输出值意外的其他隐藏信息或状态无关``，也和由I/O设备产生的外部输出无关
* 该函数``不能有语义上可观察的函数副作用``，诸如，‘触发事件’，使输出设备输出，或更改输出值以外的内容等

```javascript
// 调用这个函数，比如输入10，20，结果是30
// 那么在任何地方foo(10,20) 值都是30
function foo(num1,num2) {
    return num1 + num2
}
foo(10,20) //30
foo(10,20) //30

// 以下的情况，如果num发生了变化，那么输出的结果也会变化，不符合纯函数的定义，所以就不是纯函数
var num = 100
function foo(num1, num2) {
    return num1 + num2 + num 
}

foo(10,20) //130
num = 200
foo(10,20) // 230

```

总结纯函数

* ``确定的输入，一定会产生相同的输出``
* ``函数在执行过程中，不能产生副作用``

### 1.6.1.函数副作用

* 副作用（side effect）其实本身是医学的一个概念，比如经常说吃什么要会有副作用
* 在计算机科学中，也引用了副作用的概念，表示在``执行一个函数``时，除了``返回函数值外``，还对调用函数产生了附加影响，比如``修改了全局变量，修改参数挥着改变外部的存储``

## 1.7.柯里化函数

* 柯里化也属于函数式编程里面的一个非常重要的概念
* 不仅被用于JavaScript，还被用于其他编程语言

维基百科解释：

* 在计算机科学中，柯里化（Currying），又被译为卡瑞化、加里化
* 时把接收多个参数的函数，变成接收一个单一参数的函数，并且返回接收余下的参数，而且返回结果的新函数的技术

自己的理解：

* 只``传递给函数一部分参数来调用它``，让``他返回一个函数去处理剩余的参数``,这个过程就称之为柯里化
* 柯里化是一种函数的转换，将一个函数从可调用的f(x,y,z),转换为可调用的f(x)(y)(z)
* 柯里化不会调用函数，他只是对函数进行转换

```javascript
function foo(x,y,x) {
    return ...
}
foo(10,20,30)

// 将上面的函数转化成下面的函数的过程就是柯里化

function foo(x) {
    return function(y) {
        return function(z) {
            return ...
        }
    }
}
foo(10)(20)(30)
    
```

### 1.7.1.使用箭头函数转换柯里化

```javascript
let foo = x => y => z => console.log(x,y,z)
foo(1)(2)(3) // 1,2,3
```



### 1.7.2.自动化柯里化函数封装

```javascript
function autoCurrying(fn) {
        return function currying(...arg) {
          if (arg.length >= fn.length) {
            return fn(...arg);
          } else {
            return function (...newArr) {
              console.log(arg);
              return currying(...arg.concat(newArr));
            };
          }
        };
      }

      function foo(x, y, z) {
        // console.log(x + y + z);
        console.log(x + y + z);
      }

      let autoFoo = autoCurrying(foo);
      autoFoo("aa")("bb")("cc"); // aabbcc
```

## 1.8.组合函数

* 组合函数(Compose Function)是在JavaScript开发中的一种对``函数的使用技巧、模式``

```javascript
// 有以下场景，首先对一个数取平方，再乘以2，我们需要用到两个函数才能完成，显得有些麻烦
function dobule(num) {
    return num * 2
}
function pow(num) {
    return num ** 2
}

console.log(dobule(pow(2))) // 16

// 将其转换成组合函数
function composeFn(num) {
    return dobule(pow(num))
}
console.log(composeFn(2)) // 16
```

### 1.8.1.自动化组合函数的封装

```javascript
function compose(...fns) {
  // 1. 边界判断
  let len = fns.length;
  if (!len) return;
  for (let i = 0; i < len; i++) {
    let fn = fns[i];
    if (typeof fn !== "function")
      throw new Error(`index positon ${i} must be function `);
  }

  //
  return function (...args) {
    // 因为做了边界判断，所以第一项一定存在
    // 如果直接遍历不取第一项的值，那么每次遍历传的值都是一样的，取了第一次的结果存起来，下次的参数就是这个结果

    let result = fns[0].apply(this, args);
    for (let i = 1; i < len; i++) {
      let fn = fns[i];
      // 参数加[]是因为apply调用的，参数必须是数组，使用apply可以指定this 更加灵活
      result = fn.apply(this, [result]);
    }

    return result;
  };
}

function foo(num) {
  return num * 2;
}

function fn(num) {
  return num ** 2;
}

function fn1(num) {
  return num ** 2;
}

let newFn = compose(foo, fn, fn1);
console.log(newFn(2)); // 256

```



# 2.浏览器原理

## 2.1.页面渲染过程

![](https://static.oschina.net/uploads/img/201502/10140935_MoRm.png)

### 2.1.2.回流和重绘

#### 2.1.2.1.回流

介绍：

* 也称为重排
* 浏览器解析HTML文件时，``第一次确定``节点的大小和位置称为``布局``（layout）
* 之后对节点的大小、位置修改重新计算称之为``回流``

引起回流的情况

* 比如DOM结构发生了改变（添加一个新的节点或移除节点）
* 比如改变了布局（修改了width、height、padding、font-size等值）
* 比如窗口resize（修改了窗口的尺寸）
* 比如调用getComputedStyle方法获取尺寸、位置信息

#### 2.1.2.2.重绘

介绍：

* 浏览器解析HTML文件时，第一次渲染内容称之为绘制（paint）
* 之后重新渲染称之为重绘

引起重绘的情况

* 比如修改了背景色、文字颜色、边框颜色、样式等

### 2.1.3.如何避免发生回流

回流一定会引起重绘，所以回流是一件很消耗性能的事情

* 修改样式时，尽量一次性修改
  * 比如使用cssText进行修改，或者使用class进行修改
* 尽量避免频繁的操作DOM
  * 使用DocumentFragment或者将要操作的元素拼装好再一并追加到元素上
* 尽量避免通过getComputedStyle获取尺寸、位置等信息
* 对某些元素使用position的absolute或者fixed
  * 这种情况也会引起回流、但是开销相对较小、不会对其他的元素造成影响
  * 这些属性会让元素脱标，重排的时候不会影响标准流的元素

### 2.1.4.特殊解析-composite

* 绘制过程中，可以将布局后的元素绘制到都多个合成图层中
  * 这是浏览器的一种优化手段
* 默认情况下，标准流的内容都是被绘制在同一个图层（layers）中
* 而一些特殊的属性，会创建一个新的``合成层``（compositeLayers），并且新的图层可以利用 GPU来加速绘制
  * 因为每个合成层都是单独渲染的

常见的可以从形成新的合成层的属性：

* 3D transforms
* video、canvas、iframe
* opacity动画转换时
* position：fixed
* will-change
  * 一个实验的属性，提前告诉浏览器元素可能发生哪些变化
* animation或transition设置了opacity、transform

``分成确实可以提高性能，但是他以内存管理为代价，因此不应该最为web性能优化策略的一部分过度使用``

## 2.2.script元素和页面解析的关系

* 浏览器在解析HTML过程中，遇到了script元素是不能继续构建DOM树的
* 他会停止继续构建，首先下载js代码，并且执行js脚本
* 只有等到js脚本执行结束后，才会继续解析HTML，构建DOM树

上述的原因

* 因为js的作用之一就是操作DOM，并且可以修改DOM
* 如果等到DOM数构建完成并且渲染在执行js，会造成严重的回流和重绘，影响页面的性能
* 使用script元素提供的两个属性defer和async解决

### 2.2.1.defer

* defer属性告诉浏览器不用等待脚本下载，再去解析HTML
  * 脚本会由浏览器进行下载，但是不会阻塞DOM Tree的构建过程
  * 如果脚本提前下载好了，他会等待DOM Tree构建完成，在DOMContentLoaded时间之前执行defer中的代码
* 多个带defer的脚本时可以``保持正确的顺序执行``
* 从某种角度来说，defer可以提高页面的性能，并且推荐放到head元素中
* 注意：defer仅适用于外部脚本

### 2.2.2.async

* async也能够让脚本不阻塞页面
* async是让一个脚本完全独立的
  * 浏览器不会因async脚本而阻塞
  * async脚本``不能保证顺序``，他是独立下载，独立运行，不会等待其他脚本
  * asyn不能保证在DOMContentLoaded之前或者之后执行
* defer通常适用于需要在文档解析后操作DOM的js代码，并且对多个script文件有顺序要求的
* asyn通常用于独立的脚本，对其他脚本，甚至DOM没有依赖的

# 3.严格模式

* 严格模式通过抛出错误来``消除一些原有的静默错误``
* 严格模式``让js引擎再执行代码时可以进行更多的优化``（不需要对一些特殊的语法进行处理）
* 严格模式``禁用了再ECMAScript未来版本中可能定义的一些语法``
* 可以给某个文件开启严格模式，也可以单独给函数开启严格模式

开启严格模式后

* 无法意外的创建全局变量
* 严格模式静默失败也报错
* 不允许函数参数有相同的名称
* 不允许0的八进制语法
* 不允许使用with
* ...查mdn

# 4.数据属性修饰符

## 4.1.Object.defineProperty

* 数据属性
  * configurable
    * 默认值true
    * 表示属性是否可以通过delete删除。
    * 一旦设置后续将不能修改
  * enumerable
    * 默认值true
    * 是否可枚举
  * writable
    * 默认值true
    * 是否可修改
  * value
    * 默认情况下时undefined，（通过definedProperty设置属性的情况是）

* 存储属性
  * configurable
  * enumerable
  * set
    * 设置属性时会执行的函数，默认值时undefined
  * get
    * 获取属性时会执行的函数，默认值时undefined（不写get时默认情况）

* 使用存储属性对对象的属性进行操作

  ```javascript
  //  1. 使用Object.defineProperty给obj添加一个name的属性
  // 2. 外界访问name 其实就是访问_name的值，修改也是_name的值
  // 因为return this.name
  let obj = {
          _name: 'zs'
        }
  
        Object.defineProperties(obj, {
          _name: {
            enumerable: false,
            configurable: false
          },
          name: {
            get() {
              // 不能直接返回name 不然会造成死循环
              return this._name
            },
            set(val) {
              this._name = val
            },
            enumerable: true,
            configurable: false
          }
        })
  ```

  

# 5.原型&原型链

* 原型分为对象原型和函数原型

## 5.1.对象原型（隐式原型）

注意：对象中是不能用``对象.prototype``访问的，对象的原型称为``隐式原型``

* JavaScript中，每一个对象都有一个特殊的内置属性[[prototype]]，这个特殊的对象可以指向另外一个对象
* 通过属性key来获取一个value时，会``触发[[Get]]``的操作
* ``首先会检查该对象中是否有对应的属性``，有的话就使用，``没有就访问[[property]]中的``

获取对象原型的方式

* 通过对象的``__proto__``属性可以获取到（但是这是早期浏览器自己添加的，存在一定的兼容性）
* 通过``Object.getPrototypeOf``方法获取

## 5.2.函数原型（显示原型）

函数中的原型叫做``显示原型``

* 可以通过``.prototype``来获取原型
* 可以通过``Object.getPrototypeOf``方法获取
* 可以通过对象的``__proto__``属性可以获取到（但是这是早期浏览器自己添加的，存在一定的兼容性）
* 作用是再``使用new操作符``的时候，将``函数的显示原型``赋值给对``象的隐式原型``，这样对象就可以使用了

## 5.3.ES5实现继承

寄生组合式继承（思想）

```javascript
      // 函数作用： 创建一个空对象，对象的隐式原型指向o
      function createObject(o) {
        function F() {}
        F.prototype = o;
        return new F();
      }
      // 继承的函数
      function inherit(subType, superType) {
        // 如果适配早期的浏览器，那么就不能使用Object.crate()
        // subTyp.prototype = Object.create(superType.prototype);
        subType.prototype = createObject(superType.prototype);
        Object.defineProperty(subType.prototype, "constructor", {
          enumerable: false,
          configurable: true,
          writable: true,
          value: subType,
        });
      }

      // 实现继承

      function Person(name, age) {
        this.name = name;
        this.age = age;
      }
      Person.prototype.say = () => console.log("say Person");

      function Student(name, age, score) {
        Person.call(this.name, age);
        this.score = score;
      }

      inherit(Student, Person);

      let s1 = new Student("jack", 19, 98);
      console.log(s1);
```

## 5.4.对象的方法（部分）

### hasOwnProperty

* 对象是否有某一个属于``自己的属性``(``不是在原型上的属性``)

### in/for in操作符

* 判断``某个属性``是否在``对象或者对象的原型``上

### instanceof

* 用于``检测构造函数的property``，是否``出现在某个实例对象的原型链``上

### isPropertyOf

* 用于监测某个对象，是否出现在某个实例对象的原型链上

## 5.5.函数也是对象

* 函数也是对象
* 也有隐式原型，他的隐式原型指向Function的显示原型（默认情况下）

# 6.ES6

* 又叫ECMAScript2015或者ES2015，泛指2015及之后发布的版本的统称

## 6.1.class(类)

* 本质上是构造函数和原型链的语法糖

```javascript
      class Person {
        constructor(name, age) {
          this._name = name;
          this.age = age;
        }
// 原型方法
        say() {
          console.log("sss");
        }
      }
```

### 6.1.1.类中的存储器

* set和get除了可以在Object.defineProperty中使用，还可以直接在对象中使用（不常用）

* ```javascript
        let obj = {
          name1: "zd",
          _age: 18,
          get age() {
            return this._age;
          },
          set age(val) {
            this._age = val;
          },
        };
  ```

* 所以在类中也可以使用set和get

* 以下案列的好处是，当要获取坐标，都是成对出现的，我们还需要进行处理（写一个方法...）

* 使用get就不用写方法，直接调用position属性即可

* ```javascript
  class Rectangle {
          constructor(x, y, width, height) {
            this.x = x;
            this.y = y;
            this.width = width;
            this.height = height;
          }
  
          get postion() {
            return { x: this.x, y: this.y };
          }
  
          get size() {
            return { width: this.width, height: this.height };
          }
        }
        let rect1 = new Rectangle(10, 20, 100, 200);
        console.log(rect1.position); // {"x": 10,"y": 20}
  	  console.log(rect1.size); // {"width": 100,"height": 200}
  ```

### 6.1.2.类的静态方法

* 就是ES6之前的``类方法``

* 通常用于定义直接使用类来执行的方法，不需要有类的实例，使用```static关键字```来定义

* ```javascript
  // 静态方法的this也是指向他的调用者，通常就是类本身
  class Foo {
      constructor() {}
      static say() {
          console.log("我睡觉哦");
          console.log(this);
      }
  }
  Foo.say();
  ```


### 6.1.3.继承

* 通过关键字``extends``来实现继承
* class为我们提供了``super``关键字
  * 执行super.method(...)来调用一个父类的方法
  * 执行super(...)类调用父类constructor（只能在当前类的constructor中调用）
* 在当前类的constructor中，``必须要先使用super``才能使用this（将super()放到constructor代码块的首行），否则会报错
* super的使用位置有三个：``子类的构造方法``、``实例方法``、``静态方法``

```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    say() {
        console.log(`${this.name} is say`);
    }
    drink() {
        console.log(`${this.name} is drank`);
    }
}

class Student extends Person {
    constructor(name, age, sno) {
        super(name, age);
        this.sno = sno;
    }
    drink() {
        super.drink();
    }
}

let s1 = new Student("zs", 18, 1001);
s1.say();
s1.drink();
console.log(s1);
```

### 6.1.4.多类继承

* JavaScript是不支持多类继承的
* 使用mixin混入的思想完成

```javascript
// 有running和flayer两个类
// bird类想要继承这两个类

// class Running {
//   run() {
//     console.log('running~')
//   }
// }
// class Flayer {
//   flay() {
//     console.log('flay~')
//   }
// }

// 使用mixin混入的思想
// return class extends basicClass
// 表示 返回一个新的继承basicClass的类
// 外界接收到这个值就是一个新类并且继承了basicClasss类
function mixinRunning(basicClass) {
    return class extends basicClass {
        run() {
            console.log("running~");
        }
    };
}

function mixinFlayer(basicClass) {
    return class extends basicClass {
        flay() {
            console.log("flayer~");
        }
    };
}

class Bird {
    eat() {
        console.log("eating~");
    }
}

// 方案1
// let NewBird = mixinRunning(mixinFlayer(Bird));

// 方案2
class NewBird extends mixinRunning(mixinFlayer(Bird)) {}
let bird1 = new NewBird();
console.log(bird1);
```



## 6.2.new操作符

* 在内存中创建一个新的对象（空对象）
* 将这个对象内部的\__proto__(隐式对象)指向该类的prototype（显示原型）
* 构造函数内部的this，指向这个新的对象
* 执行构造函数的内部代码
* 如果构造函数没有返回空对象，则返回创建的新对象

## 6.3.多态（面向对象的知识，不是es6才出来的）

维基百科定义：

​	``多态``（polymorphism）指为不同数据类型的实体提供统一的接口，或用一个单一的符号来表示多个不同的类型

​	总结：不同数据类型进行同一个操作，表现的行为是不同的，这就是多态

```javascript
// 为不同数据类型的实体提供统一的接口
function sum(a1, a2) {
    return a1 + a2
}
// 调用了同一个接口，但是返回的结果的数据类型却不一样
// 符合多态的定义
sum(10,20)
sum('qq', 'nn')

// 用一个单一的符号来表示多个不同的类型
// foo可以是任意类型的，也符合多态的定义
var foo = 1
foo = 'aa'
foo = []

```

* 从以上的案例和对维基百科的对比来说，Javascript是``一定存在多态的``

## 6.4.字面量的增强

### 6.4.1.属性的简写

```javascript
let name = 'zs'
let obj = {
    name,
    age: 18
}
```



### 6.4.2.方法的简写

* 对象中的函数可以直接简写

```javascript
let obj = {
    drink：function() {}
    say() {}
}
```



### 6.4.3.计算属性的写法

```javascript
//
let key = 'address'

let obj = {
    [key]: '河南'
}

console.log(obj) // {address: '河南'}
```

## 6.5.结构（Destructuring）

### 6.5.1.数组的解构

```javascript
// 采用下标对下标的方式进行解构
// 如果不想声明下标是2的变量，直接空出去，后面的继续写
let arr = [1,2,3,4]
let [c1,c2, ,c3] = arr
console.log(c1,c2,c3) // 1,2,4

// 拓展练习，解构出数组
// 利用了解构和剩余参数的语法
let [c1, c2, ...c3] = arr
console.log(c1,c2,c3) // 1,2,[3,4]

 
// 解构的默认值
// 如果解构对应的值是undefined， 变量的值就是默认值
let [c1='ad',c2,c3='ss',,c4='de'] = arr
// c1 = 1    c4 = de

```



### 6.5.2.对象的解构

```javascript
let obj = {
    name: 'zs',
    age: 18
}

let {name,address:wAddress='hhh'} = obj
// wAddress = hhh

// 对象解构配合剩余参数
let obj = {
    name: 'zs',
    age: 19,
    address: '河南'
}
let {name,...info} = obj
//  name='zs'  info = {age:19, address:'河南'}
```

