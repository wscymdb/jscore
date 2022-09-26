# `JavaScript高级

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

* 箭头函数``不会绑定this和arguments和super``

* 箭头函数``没有显式原型``

* 箭头函数不能用作构造函数(因为没有显示原型)

* 如果函数体只有一行代码，默认值返回的是一个对象，必须用括号将对象包裹

  * ```javascript
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
* 把接收多个参数的函数，变成接收一个单一参数的函数，并且返回接收余下的参数，而且返回结果的新函数的技术

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

## 6.5.解构（Destructuring）

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

## 6.6.let&const

### 作用域提升

* let和const声明的变量，没有作用域提升，但是在代码执行的时候还是会被先创建的，只不过只有到变量被赋值的时候才可以被访问

### 暂时性死区

``英文``：TDZ（temporal dead zone）社区给的定义，官方没有这个定义

* 从块级作用域的顶部一直到变量声明完成之前，这个变量处在暂时性死区

  * 在这个区域内，无法访问该变量

* 暂时性死区和定义的位置没有关系，和代码的执行顺序有关系

  * ```javascript
    function foo() {
        console.log(message)
    }
    let message = 'zs'
    foo() // zs
    ```

* 暂时性死区形成后，再该区域内这这个标识符不能够被访问

  * ```javascript
    let message = 'zs'
    function foo() {
        console.log(message)
        let message = 'aa'
    }
    foo()  // 报错
    ```

### 声明的变量不添加window上

### 块级作用域

* 在块级作用域内用``let或const``声明的变量外界无法访问

* 但是在块级作用域内声明的函数在外界允许访问，但是只能``在声明之后``才能访问

  * 因为块级作用域是es6之后才出来的，如果函数写在块级内，那么早期的代码如果在外部访问就会报错，项目中可能会用社区的库，有些库可能不维护了，所以浏览器对块级内声明的函数做了特殊的处理

  ```javascript
  foo() // 报错
  {
      let a = 123
      function foo() {}
  }
  console.log(a) // 报错
  foo()
  ```


## 6.6.模板字符串

### 模板字符串

```javascript
b = 123
a = `${b} + ddd`
//a  123 + ddd
```



### 标签模板字符串

react中会用，暂做了解

```javascript

// 
      function foo(...arg) {
        console.log(arg);
      }

      foo`hahha is ${123} dog ${222}
      .banner {
        color：red;
      }
      `;
// 打印结果
      // [
      //   [
      //     "hahha is ",
      //     " dog ",
      //     "\n      .banner {\n        color：red;\n      }\n      ",
      //   ],
      //   123,
      //   222,
      // ];
```

## 6.7.函数增强

### 6.7.1.默认参数

* 有默认参数的形参尽量写在后面

* 有默认参数的形参，是不会计算在length之内的

* 默认参数之后的所有参数都不参与计算在length之内

  * ```javascript
    function foo(age,name=123,height=2) {}
    foo.length // 1
    ```

* 剩余参数也是放到最后面，但是要放到默认参数之前

```javascript
function foo(name='zs',age=18) {
    console.log(name,age)
}
foo()  // zs,18
```

### 6.7.2.默认参数解构

```javascript
const obj = {name:'zs'}
const {name = 'jack'}  = obj


// 原始写法 
// 当形参是一个对象的时候，设置默认值
function foo(obj = {name:'zs'} ) {
    console.log(obj.name)
}
// 优化
function foo({name} = {name: 'zs'} ) {
    console.log(name)
}
// 进一步优化
function foo({name = 'zs'} = {} ) {
    console.log(name)
}
foo()
```

## 6.8.展开语法（spread syntax）

* 在``函数调用``时使用
* 在``数组构造``时使用
* 在``构建对象字面量``时，也可以使用展开运算符，ES9(ES2018)新添加的
  * 必须是``对象字面量``的形式才可以使用

```javascript
// 基本使用
let obj = {
    name: 'zs'
}
let info = {...obj}

let arr = [1,2,3]
let arr1 = [...arr,9]  // [1,2,3,9]

let hello = 'hello'
let arr2 = [...hello]
```

## 6.9.Symbol

* Symbol是ES6中新增的一个``基本数据类型``，翻译为``符号``。

**Symbol出现的原因：**

* 在ES6之前，对象的key只能是字符串，很容易造成``属性名的冲突``

* 比如传入一个函数的参数是一个对象，函数体中对对象添加一个属性，那么如果传入的对象中已经有函数代码块中同名的key那么就会进行覆盖

  * ```javascript
    function foo(obj) {
        obj.name = 'zs'
    }
    foo({name:'jack'})
    ```

* Symbol就是为了解决这一问题的，用来``生成一个独一无二的值``

  * Symbol值是通过Symbol函数来生成的，生成后可以作为属性名

* ```javascript
  const s1 = Symbol()
  const obj = {
      [s1]: 'li'
  }
  obj[Symbol()] = 'sss'
  ```

* Symbol即使创建多次，他们也是不同的

  * Symbol函数执行后每次创建出来的值都是独一无二的

* 也可以在创建Symbol值的时候传入一个描述description（ES2019新增的特性）

  * ```javascript
     const s1 = Symbol('我是描述哈哈')
    ```

**获取用Symbol属性名的值**

* 不能使用Object.keys()获取key是symbol的键名
* 要通过``Object.getOwnPropertySymbol()``获取

```javascript
const s1 = Symbol()
const obj = {
    [s1]: 'hhh'
}
let symbolKeys = Object.getOwnPropertySymbol(obj)
```

获取两个相同的Symbol

* 通过Symbol.for方法来做到这一点
* 可以通过Symbol.keyFor方法来获取对应的key (当前symbol的描述)

```javascript
const s1 = Symbol()
const s2 = Symbol()
consolel.log(s1 === s2) //true

const s3 = ('sdfs')
console.log(Symbol.keyFor(s3)) // sdfs
```

## 6.10.Set

* 类似数组，但是``数据不能重复``
* new Set(iteration)
  * 入参是``可迭代的元素``

常见的属性和方法

* **size**
  * 获取集合元素中的个数
* **add(value)**
  * 添加某个元素，``返回Set对象本身``
* **delete(value)**
  * 从Set中删除和这个值相等的元素，``返回Boolean类型``
* **has(value)**
  * 判断set中是否存在某个元素，``返回Boolean类型``
* **clear()**
  * 清空set中所有的元素，``没有返回值``
* **forEach(callback [, thisArg])**
  * 通过forEach遍历set

```javascript
const s1 = new Set([1,2,3])
s1     //  {1,2,3}
```

### 6.7.1.WeakSet

**弱引用和强引用**

``强引用（strong reference）：``

​	设有个一变量obj，他的值是{name:'zs'}，当js的垃圾回收（GC）运行时，根据可达性的原理，发现{name:'zs'}这个对象被obj变量所引用着，那么就不会将其清理掉，这就是强引用

``弱引用（weak reference）：``

​	设有一个变量obj，他的值是{name:'zs'},虽然他也被obj引用着，但是通过别的方式将这个引用变成弱引用了，那么当GC运行时，就会忽略这个引用，然后{name:'zs'}这个对象就可能被回收

``WeakSet``就是弱集合的意思，也就是说他的引用的值，就是``弱引用``

**WeakSet和Set的区别**

* WeakSet中``只能存放对象类型，不能存放基本数据类型``
* WeakSet``对元素的引用是弱引用``，如果他引用的值没有被其他变量引用，那么GC可以对该引用的值进行回收
* WeakSet是不能遍历的
  * 因为``WeakSet是弱引用``，如果我们遍历获取其中的元素，那么可能造成对象不能正常的销毁

```javascript
cosnt arr = [{name: 'zs'}]
const weakS = new weakSet(arr)
```

## 6.11.Map

* 也是键值对形式
* 键是``任意类型``

常见的属性和方法

* **size**
  * 返回Map中元素的个数
* **set(key, value)**
  * 在Map添加key、value，``并返回整个Map对象``
* **get(key)**
  * 根据key获取Map中的value
* **has(key)**
  * 判断是否包含某一个key，``返回Boolean``
* **delete(key)**
  * 根据key删除一个键值对，``返回Boolean``
* **clear()**
  * 清空所有元素
* **forEach(callback [, thisArg])**

```javascript
const m1 = new Map([
    ['建','值'],
    [1,2]
])
```



### 6.11.1.WeakMap

* WeakMap的``key只能使用对象``，不接受其他类型作为key
* WeakMap的``key对对象的引用是弱引用``，如果没有其他引用引用这个对象，那么GC可以回收该对象
* ``不支持遍历``

```javascript
const obj1 = {name: 'zs'}
const wm = new WeakMap()
wm.set(obj1, 123)
```

# 7.ES7~ES13新特性

## 7.1.ES7新特性

### 7.1.1.Array.includes

查询mdn

### 7.1.2. 指数运算符

求一个数的多少次方，用``**``表示

```javascript
Math.pow(2,3)
2 ** 3
```

## 7.2.ES8新特性

### 7.2.1.Object.values

* 获取对象中所有对值

* ```javascript
  const obj = {
      name: 'zs',
      age: 19
  }
  const val = Object.values(obj)
  // ['zs', 19]
  ```

### 7.2.2.Object.entries

* 可以获取一个数组，数组中``会存放可枚举属性的键值对的数组``

```javascript
const obj = {
    name: 'zs',
    age: 19
}
const ent = Object.entries(obj)
// ent  [ ["name", "zs"], ["age", 19] ]
for (let item of ent) {
	const [key,value] = item
    console.log(key,value)
}

// 如果是一个数组
console.log(Object.entries([1,2,3])) 
// [['0', 1], ['1', 2], ['2', 3]]

// 如果是字符串
console.log(Object.entries('aa'))
//[['0','a'],['1','a']]
```

### 7.2.3.字符串填充

* **padStart(targetLength[, padString])**
* **padEnd(targetLength[, padString])**
* targetLength:当前字符串需要填充的目标长度，``如果小于这个长度``将用padString进行填充

```javascript
// 格式化日期
const hh = "3".padStart(2, "0");
const mm = "15".padStart(2, "0");
console.log(`${hh}:${mm}`);
// 03:15

// 加密身份证或者银行卡
const cardNum = '411989199901011234'
const lastFour = cardNum.slice(-4)
const resNum = lastFour.padStart(cardNum.length, '*')

```

 

## 7.3.ES10

### 7.3.1.flat

* 将一个数组，按照规定的深度遍历，将遍历到的元素和子数组中的元素组成一个新的数组，然后返回
* **flat(depth)**
  * 按照深度去遍历,默认值是1

```javascript
// 更多案例查询mdn文档
const nums = [1,2,[1,2],[[1,2]]]
const flatNum = nums.flat(1)
console.log(flatNum) // [1,2,1,2,[1,2]]
```

### 7.3.2.flatMap

* 方法首先使用映射函数映射每个元素，然后将结果压缩成一个新数组
* 做了``两步操作``
  * 首先对数组进行``map操作``
  * 然后在对操作之后的数组进行``flat(1)``的操作

```javascript
const msg = ["hello world", "hello zs"];
const newMsg = msg.flatMap((item) => item.split(" "));
console.log(newMsg); // ['hello', 'world', 'hello', 'zs']
```

### 7.3.3.fromEntries

* 将转为entries的对象，变成没转之前的状态

```javascript
const obj = {
    name: 'zs',
    age: 19
}
let objEntries = Object.entries(obj)
// [ ['name', 'zs'],['age', 19] ]
let transObj = Object.fromEntries(objEntries)
// {name: 'zs', age: 19}

```

### 7.3.4.trimStart

* 去除首部的空格

```javascript
const str = ' hello world'
const trimS = str.trimStart()
// 'hello world'
```



### 7.3.5.trimEnd

* 去除尾部的空格

```javascript
const str = ' hello world '
const trimS = str.trimEnd()
// ' hello world'
```

## 7.4.ES11

### 7.4.1.BigInt

* 在早期的JavaScript中，不能正确的表示过大的数字
* 大于``Number.MAX_SAFE_INTEGER``的数值，表示的可能不正确
* 使用``BigInt``即可，在要表示的数后面添加``n``

```javascript
console.log(Number.MAX_SAFE_INTEGER)
// 9007199254740991
console.log(9007199254740991 + 1) // 9007199254740992
console.log(9007199254740991 + 2) // 9007199254740992


// 使用BIGINT
console.log(9007199254740991n + 2n) // 9007199254740993
console.log(9007199254740991n + 5n) // 9007199254740996
```

### 7.4.2.空值合并运算符

* Nullish Coalescing Operator
* 使用``??``，如果??前面的值是``null``或``undefined``,那么就使用``??后面的值``

```javascript
let info = undefined
info = info ?? '默认值'
//  info  默认值  
```

### 7.4.3.可选链操作符

* optional chaining
* 允许读取位于连接对象链深处的属性的值，而不必明确验证链中的每个引用是否有效,如果没有找到，该表达式短路返回值是 `undefined`

```javascript
const obj = {
name:'zs'
}
obj?.asy?.()
```

### 7.4.4.Global this

* 规范了获取全局对象使用``globalThis``

## 7.5.ES12

### 7.5.1.FinalizationRegistry

* 可以让对象在被垃圾回收的时候请求一个回调
* FinalizationRegistry提供一种方法，``当一个在注册表中的对象被垃圾回收时，请求在某个时间点上调用一个清理回调``（清理回调有时被称为finalizer）
* 可以通过``调用register方法，注册热河想要清理回调的对象``，传入该对象和所含的值

```javascript
// register的第二个参数，自定义标识符
// 如果有第二个参数，那么在执行回调的时候，参数就是传入的自定义标识符
let obj = [{ name: "zs" }];
let info = { a: 123 };
const finalization = new FinalizationRegistry((val) => {
    console.log("对象被回收了", val);
});

finalization.register(obj, "obj");
finalization.register(info, "info");

obj = null;

// 检测WeakMap是否会被GC删除
// 结果会调用回调，证明会被GC回收
// 如果将WeakMap换成Map就不会执行回调
let obj = { name: "123" };
let wp = new WeakMap();

wp.set(obj, 123);

const finalization = new FinalizationRegistry(() => {
    console.log("first");
});
finalization.register(obj);
obj = null;
```

### 7.5.2.WeakRefs

* **WeakRefs: 弱引用**

```javascript
let obj = { name: "zs" };
let wr = new WeakRef(obj);

let finalization = new FinalizationRegistry((val) => {
    console.log("对象被回收", val);
});

finalization.register(obj, "obj");

// 此时obj指向的对象已经被回收了，
obj = null;
```

### 7.5.3.逻辑赋值运算符

* ||=
* ??=
* &&=

```javascript
function foo(message) {
    // 1.||逻辑运算符
    // message = message || '默认值'
    // message ||= '默认值'
    
    // 2. ?? 逻辑运算符
    message = message ?? '默认值'
    message ??= '默认值'
    
    console.log(message)
}
foo(0)  // 0
foo()  // 默认值

let obj = {
    name:'zs'
}
obj = obj && obj.name
obj &&= obj.name
```

### 7.5.4.数字分割

* Numeric Separator
* 使用下划线分割较大的数字

```javascript
const num = 123_0000_0000
```

### 7.5.5.字符串替换

* String.prototype.replaceAll()
* 可以替换全部的字符串
* String.prototype.replace()只能替换一次

## 7.5.ES13

### 7.5.1.Object.hasOwn

* Objcet.hasOwn(obj, propKey)
* 用来替代Object.prototype.hasOwnProperty()方法的

### 7.5.2.method.at

* 查mdn

### 7.5.3.class的新成员

* **Instance publish fields**
  * 实例的公共字段，每个实例都有的字段
* **Static public fields**
  * 类的静态字段
* **Instance private fields**
  * 
* **static block**
  * 类的静态代码块，当类加载的时候执行一次

```javascript
class Person {
    // 所有实例都可有这个属性
    height = 1.88
	constructor() {}
	// 类的静态属性
	static address = '河南'
	// 类的静态私有属性, 只有类内部能够访问
	// 用this.访问
	static #number = 12
    // 类的静态代码块
    static {
        console.log('只有类初始化时候执行一次')
    }
}
```



# 8.Proxy

* 对``一个对象进行代理``，之后``对该对象的所有操作``都是``通过代理对象``来完成的
* new Proxy(target, handler)
* target : 要代理的对象
* handler：被代理对象的行为
  * 内置很多个方法（只列出几个，剩余的查mdn）

```javascript
const obj = {
    name: "zs",
    age: 19,
};

// target表示被代理的源对象
// key 被查看、操作的对象的属性
// val 被修改的新值
// receiver 被代理的对象
const proxyObj = new Proxy(obj, {
    set(target, key, val, receiver) {
        console.log(`监听了${key},值是${val}`);
        target[key] = val;
    },

    get(target, key) {
        console.log(`监听了${key}`);
        return target[key];
    },

    deleteProperty(target, key) {
        delete target[key];
    },

    has(target, key) {
        console.log(`${key}被判断是否存在${target}中`);
        return key in target;
    },
});

proxyObj.name = "jack";
proxyObj.address = "河南";
proxyObj.age;
delete proxyObj.address;
console.log("age" in proxyObj);
```

# 9.Reflect

* Reflect是一个对象，字面意思是``反射``
* 主要提供了很多操作JavaScript对象的方法，有点像Object中操作对象的方法
* 将``Object对象中对 对象本身进行操``作的API迁移到了Reflect上，并且又新增了一些别的方法

Reflect和Object的区别

* 早期的ECMA规范中没有考虑这中``对 对象本身的操作如何设计会更加规范``，所以将API放到了Object上
* 但是``Object作为一个构造函数``这些操作实际放到它身上并不合适
* 另外还包含一些``类似于in、delete操作符``让js看起来会有一些奇怪
* 所以在ES6中``新增了Reflect``，让我们这些操作都集中到了Reflect对象上
* 另外在使用proxy时，可以做到``不操作原对象``

```javascript
const obj = {
    name: "zs",
    age: 19,
};
// 返回 对象本身
console.log(
    Object.defineProperty(obj, "name", {
        configurable: false,
    })
);
// 返回Boolean值  false上面设置了configurable：false
console.log(
    Reflect.defineProperty(obj, "name", {
        enumerable: false,
    })
);

// 都返回Boolean 但是后者的书写的语义更加明确
console.log(delete obj.name); // false
console.log(Reflect.deleteProperty(obj, "name"));  // false
```

## Reflect和Proxy的搭配使用

* 好处一：
  * 代理对象的目的：不直接操作原对象
* 好处二：
  * Reflect.set方法返回Boolean值，可以判断本次的操作是否成功
* 好处三：
  * 见下面案例``好处三``

```javascript
const obj = {
    name: "zs",
    age: 19,
};

Reflect.defineProperty(obj, "name", {
    configurable: false,
    writable: false,
});
// 使用Reflect对对象进行操作，可以进行间接操作对象
// 可以返回一个Boolean值
// 如果对象的属性被保护起来了，那么可以判断操作是否成功
// 避免了严格模式下报错
const proxyObj = new Proxy(obj, {
    set(target, key, newVal, receiver) {
        let isConfig = Reflect.set(target, key, newVal, receiver);
        // let isConfig = target[key];
        console.log(isConfig);
        if (!isConfig) {
            console.log(`${key}属性不支持该操作`);
        }
    },
});
proxyObj.name = "河南";
console.log(proxyObj);
```

案例``好处三``

```javascript
const obj = {
    _name: "zs",
    set name(newVal) {
        this._name = newVal;
    },
    get name() {
        return this._name;
    },
};

//  Reflect的receiver参数表示
//  set/get（obj中的set/get） 方法执行的时候的this

// 如果没有传入receiver，那么obj中的set在调用的时候只监听了一次
// 当调用proxyObj.name的时候其实是调用了两次set方法
// 第一次是调用了proxyObj的set，然后执行了Reflect.set（）
// 调用了obj的set方法，但是此时obj的set中的this指向的是obj
// 操作源对象，proxyObj无法监听到，所以只监听了一次

// 如果传入receiver，
// 当执行Reflect.set()，时候，obj的this已经变成了proxyObj这个代理对象
// this._name其实就是proxyObj._name
// 那么又会执行proxyObj的set方法，所以就可以监听两次

// get方法和上述同理

// 如果不使用Reflect的方法操作对象，那么也只会监听一次，因为obj的this就是指向obj本身

// 所以 如果想要监听两次操作，就使用Reflect，如果不在乎的话，也可以使用target[key]

let proxyObj = new Proxy(obj, {
    set(target, key, newVal, receiver) {
        console.log("设置了值");
        Reflect.set(target, key, newVal, receiver);
    },
    get(target, key, receiver) {
        console.log("获取了值");
        return Reflect.get(target, key, receiver);
    },
});

proxyObj.name = "123";
proxyObj.name;
```

# 10.Promise

* promise是一个类
* 在通过new创建Promise对象时，传入一个回调函数，我们称之为executor
  * 这个回调函数会被立即执行 ，并且给传入两个回调函数resolve，reject
  * 当``调用resolve``回调函数时，``会执行Promise对象的then方法``传入的回调函数
  * 当``调用reject``回调函数时，``会执行Promise对象的catch方法``传入的回调函数

## 10.1.promise的状态

* **pending（待定）**
  * 初始状态，既没有被兑现，也没有被拒绝，
* **fulfilled（已对象）**
  * 意味着操作成功，执行了resolve时，处于该状态
* **rejected（已拒绝）**
  * 意味着操作失败，执行reject时，处于该状态

## 10.2.注意

* **Executor**是在创建Promise时需要传入的一个回调函数，``这个回调函数会被立即执行``，并且传入两个参数(Executor是传入的回调的叫法)

* Promise的``状态一旦被确定下来，就不会更改``，也不能在执行某一个回调函数来改变状态
  * 一旦``调用了resolve或者reject``，就``不能在调用这两个中的任意一个``，调用了也没有反应

## 10.3.resolve的值

* 如果传入``一个普通的值或者对象``，那么这个值将``作为then方法的回调的参数``
* 如果resolve中传入的是``另外一个Promise``，那么这个``新Promise会决定原来的Promise的状态``
* 如果resolve中传入的是一个``对象``，并且这个``对象中有then方法``，那么会执行then方法，并根据``then方法的结果来决定promise的状态``

```javascript
const p = new Promise((resolve, reject) => {
    resolve("p的resolve");
});
const promise = new Promise((resolve, reject) => {
    // 1. resolve() 普通的值
    // resolve(123);

    // 2. resolve() 一个promise， 这个新的promise会决定原promise的状态
    // resolve(p);

    // 3. resolve() 一个带有then方法的对象
    resolve({
        name: "22",
        then(resolve, reject) {
            // resolve("hhh");
            reject("ss");
        },
    });
});

promise
    .then((val) => console.log(`结果：${val}`))
    .catch((err) => console.error(err));
```

## 10.4.then方法相关

* 可以调用多次then方法，里面的回调都会被调用
* 可以将catch写在then方法中
* 写then的时候后面最好跟一个catch方法，不然如果promise中拒绝了，那么外面就会报错

```javascript
const promise = new Promise((reslove, reject) => {
    reslove('成功的回调')
    // reject('失败的回调')
})

promise.then(() => {})
promise.then(() => {})

promise.then(res => {}, err => {})

```

### 10.4.1.then方法支持链式调用

* then方法的``返回值也是一个Promise``**,所以可以支持链式调用**

```javascript
// then方法返回一个新的promise，这个新的promise的决议（resolve/reject）是等到then方法传入的回调函数有返回值时进行决议的
const promise = new Promise((resolve, reject) => {
    resolve("aaa");
    // reject("ss");
});

// then方法中的返回值的类型(见上面的10.3)
// 普通值
// 新的Promise
// thenable对象（对象中有then方法）
promise
    .then((res) => {
    console.log(res); // aaa
    return "hhh";
    // return {
    //   then(resolve, reject) {
    //     reject(0.11);
    //   },
    // };
})
    .then((res) => {
    console.log(res); // hhh
});

// then方法相当于做了以下的操作
// 所以如果then方法中没有返回值，那么下个then方法中的结果就是undefined
function then(cb) {
    let res = cb();
    return new Promise((resolve, reject) => {
        resolve(res);
    });
}
```

## 10.5.catch方法相关

* 可以调用多次catch方法，里面的回调都会被调用

```javascript
const promise = new Promise((reslove, reject) => {
    reject('失败的回调')
    // reject('失败的回调')
})

promise.reject(() => {})
promise.reject(() => {})
```

### 10.5.1.catch的链式调用

* then方法的``返回值也是一个Promise``**,所以可以支持链式调用**

```javascript
const promise = new Promise((resolve, reject) => {
    reject("ee");
});

// catch方法也会返回一个新的Promise
// catch方法默认返回undefined
// 如果写了return 或者默认的返回，那么决议就是resolve，链式调用的下一级就会进入到then方法中
// 所以默认情况下不能一直.catch().catch()调用
// 如果想使用.catch.catch，那么可以在每个catch中抛出一个异常，这样就可以一直被catch所捕获
promise
    .catch((err) => console.log(err)) //ee
    .then((res) => console.log(res)) // undefined
    .then((res) => console.log(res)); // undefined

const promise1 = new Promise((resolve, reject) => {
    resolve("ee");
});

// 如果有promise的决议是reject的，那么会找到最近的catch方法进行调用
// 所以下面代码会在执行第一个then后，直接执行catch方法
// 使用throw 或者 return一个新的promise（决议是reject）都会进入catch中
promise1
    .then((res) => {
    console.log(res, "asa");
    throw new Error("报错");
})
    .then((res) => console.log(res))
    .catch((err) => console.log(err));

// catch方法相当于做了以下的操作
function catch1(cb) {
    let res = cb();
    return new Promise((resolve, reject) => {
        resolve(res);
    });
}
```

##  10.6.finally方法相关

* es9新增的新特性
* 表示``无论Promise对象无论变成fulfilled还是rejected状态，最终都会被执行的代码``
* finally方法不接受参数，因为无论前面是fulfilled还是rejected状态，他都会执行

## 10.7.Promise的类方法

### resolve方法

* 就是``new promise(resolve => resolve())``的语法糖

### reject方法

* 就是``new promise((resolve,reject) => reject())``的语法糖

### all方法

* 他的作用是``将多个Promise包裹在一起形成一个新的Promise``
* ``新的Promise状态由包裹的所有Promise共同决定``
  * 当``所有的Promise状态变成fulfilled状态``时，``新的Promise状态为fulfilled``，并且会``将所有的Promise的返回值组成一个新的数组``
  * 当``有一个Promise状态为reject时``，新的``Promise状态为reject``，并且会``将第一个reject的返回值作为参数``，给catch方法

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("11");
    }, 3000);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("33");
    }, 1000);
});

// 如果上面的三个promise中有一个reject，那么不用等到全部promise执行完毕，直接调用catch，剩余没有执行的promise也不会执行了
Promise.all([promise, promise1, promise2])
    .then((res) => {
    console.log(res); //["11", "22", "33"];
})
    .catch((err) => console.log(err));
```

### allSettled

* es11中新增的方法
* 该方法``会在所有的Promise都有结果(settled)``，（无论是fulfilled还是rejected），``才会有最终的状态 ``
* 并且这个Promise的``结果``一定是``fulfilled状态的``

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("11");
    }, 3000);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("33");
    }, 1000);
});
Promise.allSettled([promise, promise1, promise2]).then((res) => {
    console.log(res);
});
// [
//   { status: "rejected", reason: "11" },
//   { status: "fulfilled", value: "22" },
//   { status: "fulfilled", value: "33" },
// ];
```

### race方法

* race是竞赛、竞技的意思
* 表示多个Promise，``谁先有结果那么就使用谁的结果``
* ``无论Promise的状态是fulfilled还是rejected``

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("11");
    }, 300);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reslove("33");
    }, 1000);
});
Promise.race([promise, promise1, promise2])
    .then((res) => {
    console.log(res);
})
    .catch((err) => console.log(err));
// 11
```

### any方法

* es12新增的方法 
* any方法等到一个fulfilled状态，才会决定新Promise的状态
  * 如果``有一个Promise是fulfilled``，那么直接调用then方法，``只会``将当前的内容传进去
* 如果所有的Promise都是rejected，那么也会等到所有的Promise都变成rejected状态，然后会进入catch方法，会报一个``AggregateError``的错误

```javascript
const promise = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("11");
    }, 300);
});
const promise1 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("22");
    }, 2000);
});
const promise2 = new Promise((reslove, reject) => {
    setTimeout(() => {
        reject("33");
    }, 1000);
});
Promise.any([promise, promise1, promise2])
    .then((res) => {
    console.log(res);
})
    .catch((err) => console.log(err));
// AggregateError: All promises were rejected
```

# 11.Iterator(迭代器)

* **在Javascript中，迭代器也是哟个具体的对象，这个对象需要符合迭代器协议**（iterator protocol）
  * 迭代器协议定义了``产生一系列值（无论是有限还是无线个）的标准方式``
  * 在JavaScript中这个标准就是一个``特点的next方法``

**next方法有如下的要求**

* 一个``无参数或者一个参数的函数``，返回一个应当``拥有以下两个属性的对象``
* **done(Boolean)**
  * 如果迭代器``可以产生序列中的下一个值``，则为false
  * 如果迭代器``已将序列迭代完毕``，则为true，这种情况下，value是可选的，如果它依然存在，即为迭代结束之后的默认返回值（一般情况是undefined）
* **value**
  * 迭代器返回的任何JavaScript值。``done为true时可以省略``

```javascript
// namesIterator对象就是names的迭代器
const names = ["a", "b", "c"];
let i = 0;
const namesIterator = {
    next() {
        if (i < names.length) {
            return { done: false, value: names[i++] };
        } else {
            return { done: true, value: names[i++] };
        }
    },
};
console.log(namesIterator.next());
console.log(namesIterator.next());
console.log(namesIterator.next());
console.log(namesIterator.next());
```



## 11.1.可迭代对象

* 当一个对象``实现了iterable protocol协议时``，他就是一个可迭代对象
* 这个对象的要求是``必须实现@@iterator方法``，在代码中``使用Symbol.iterator访问该属性``
* 如果想让一个对象变成可迭代的对象，那么需要在这个``对象中返回一个迭代器``
* 这个对象中必须有``属性名是[Symbol.iterator]``方法
  * 方法名 是 Symbol.iterator  这个是Symbol类的方法所以``要用计算属性名``
  * 这个方法返回一个迭代器

```javascript
const infos = {
    name: "jack",
    age: 19,
    height: 1.88,
    [Symbol.iterator]() {
        // 获取key
        // let keys = Object.keys(this);
        // 获取值
        // let values = Object.values(this);
        // 获取entries(键值对)
        let entries = Object.entries(this);
        let index = 0;

        let iterator = {
            next: () => {
                if (index < entries.length) {
                    return { done: false, value: entries[index++] };
                } else {
                    return { done: true, value: undefined };
                }
            },
        };
        return iterator;
    },
};

for (let info of infos) {
    console.log(info);
}
```



## 11.2.可迭代对象的应用

**JavaScript中语法(常用)**

* for...of
* 展开语法（spread syntax）
* yield*
* 解构赋值（Destructuring_assignment）

**创建一些对象时：**

* new Map([iterator]) / new WeakMap([iterator])
* new Set([iterator]) / new WeakSet([iterator])

一些方法的调用

* Promise.all(iterator)
* Promise.race(iterator)
* Array.from(iterator)

## 11.3.自定义类的迭代

```javascript
class Person {
    constructor(name, age, height, friends) {
        this.name = name;
        this.age = age;
        this.height = height;
        this.friends = friends;
    }

    [Symbol.iterator]() {
        let index = 0;
        let values = Object.values(this);
        return {
            next() {
                if (index < values.length) {
                    return { done: false, value: values[index++] };
                } else {
                    return { done: true, value: values[index++] };
                }
            },
        };
    }
}

const p1 = new Person("jack", 18, 1.88, ["rose", "lily"]);

for (let item of p1) {
    console.log(item);
}
```

## 11.4.终端检测器

* 迭代器在某些情况下没有完全迭代的情况下终端
  * 比如遍历的过程中通过break、return、throw中断了循环操作
  * 比如在解构时候，没有解构所有的值
* 就是迭代器的``return方法``

```javascript
const infos = {
    name: "jack",
    age: 19,
    height: 1.88,
    [Symbol.iterator]() {
        // 获取值
        let values = Object.values(this);
        let index = 0;

        let iterator = {
            next: () => {
                if (index < values.length) {
                    return { done: false, value: values[index++] };
                } else {
                    return { done: true, value: undefined };
                }
            },
            return() {
                console.log("进入终端监测");
                return { done: true };
            },
        };
        return iterator;
    },
};
const [name] = infos;
// console.log(name);
```



# 12.Generator(生成器)

## 12.1.什么是生成器

* 生成器是ES6中新增的``一种函数控制、使用的方案``，它可以让我们更加灵活的``控制函数什么时候继续执行、暂停执行``等

* ``生成器``是``由生成器函数``所创造的
* ``生成器函数也是函数``，但是和普通函数有些区别
  * 首先，生成器函数需要在``function后面``加一个符号`` *``

    * 符号挨着function后面，或者挨着函数名前面都行

    * 如果没有function关键字声明的函数，``*``直接加在``名字前面``

    * ```javascript
      let info = {
          *next() {}
      }
      ```
  * 其次，生成器函数``可以通过yield关键字``来控制函数的执行流程
  * 最后，生成器函数返回值是一个``Gererator(生成器)``：

    * 生成器事实上是一种特殊的迭代器

```javascript
function* foo() {
    console.log("first");
    console.log("first");
    yield；
    console.log("first`q");
    console.log("first`q");
}

const gererator = foo();
```

## 12.2.生成器函数执行

* 生成器函数的代码``直接调用是不会执行``的
  * 因为生成器函数``返回一个生成器``
* 通过``调用``返回的``生成器的next方法``来执行生成器函数中的代码块



* 当执行代码的过程中``遇到yield关键字``的时候会``暂停执行``
  * next函数是有返回值的
* 如果想``继续执行``，需要``再次调用next``函数
* ``yield 后面可以跟值``
  * 这个值就会作为next返回值中对象的value的值
* 调用``next的时候可以传参``，但是这个参数会在yield之后的代码中接收到

```javascript
// next函数传参 第一次执行next无法接收到，一般直接传在函数中
function* foo(res) {
    console.log("111", res);
    console.log("111", res);
    const res1 = yield "萘萘萘";
    // return "萘萘萘";
    console.log("222", res1);
    console.log("222", res1);
    const res2 = yield "哈哈哈";
    console.log("333", res2);
    console.log("333", res2);
}

const gererator = foo("第一个");
console.log(gererator.next());
console.log(gererator.next("第二个"));
console.log(gererator.next("第三个"));

// 执行结果如下
// {value: '萘萘萘', done: false} 这种是next函数的返回值
// 如果遇到return后续代码不会执行，next的返回值是 {value: '萘萘萘', done: true}
// 之后在执行next 返回值都是 {value: undefined, done: true}
/*
        111 第一个
        111 第一个
        {value: '萘萘萘', done: false}
        222 第二个
        222 第二个
        {value: '哈哈哈', done: false}
        333 第三个
        333 第三个
         {value: undefined, done: true}
      */
```

## 12.3.生成器函数提前结束

* 除了在函数中直接使用return，也可以用下面的方式

```javascript
function* foo() {
    try {
        console.log("1111");
        const c1 = yield "aaa";
        console.log("2222", c1);
        const c2 = yield "bbb";
        console.log("3333", c2);
    } catch (error) {
        console.log(error);
    }
}
```

### 使用return函数

* return传值后这个生成器函数就会结束，之后调用next不会继续生成值了

* ```javascript
    const genetarorFoo = foo();
        console.log(genetarorFoo.next("@@@"));
        console.log(genetarorFoo.return('###');
        console.log(genetarorFoo.next("$$$"));
  ```

### 使用throw函数

* 抛出异常后可以在生成器函数中捕获异常

* 但是在catch语句中不能继续yield新的值了，但是可以在catch语句外使用yield继续中断函数的执行

* ```javascript
  const genetarorFoo = foo();
  console.log(genetarorFoo.next("@@@"));
  console.log(genetarorFoo.throw(new Error()));
  console.log(genetarorFoo.next("$$$"));
  ```

# 13.生成器替代迭代器

* 生成器本身就是特殊的迭代器
* 生成器函数返回的是一个生成器，可以调用next方法
* 当遇到yield时候，返回值的done是false
* 没有yield时候，返回值的done是true

## 13.1.生成器代替迭代器应用场景

```javascript
const names = ['jack', 'rose', 'lily']

function* createIterator(iter) {
for (let i = 0; i < iter.length; i++) {
yield iter[i]
}
// yield iter[0]
// yield iter[1]
// yield iter[2]
}

const namesIterator = createIterator(names)

console.log(namesIterator.next()) // {value: 'jack', done: false}
console.log(namesIterator.next()) // {value: 'rose', done: false}
console.log(namesIterator.next()) // {value: 'lily', done: false}
console.log(namesIterator.next()) // {value: undefined, done: true}

```

**可迭代对象的优化版本**

* 可迭代对象的标准
  * [Symbol.iterator]方法中返回一个迭代器
* 生成器本身就是一个特殊的迭代器

```javascript
const infos = {
    firends: ["jack", "rose", "lily"],
    *[Symbol.iterator]() {
       yield* this.firends;
    },
};

for (let info of infos) {
    console.log(info);
}
```

## 13.2.yield* 的使用

* yield* 可以产生一个可迭代对象
* yield* 后面跟的必须是一个``可迭代的对象``
* yield*相当于是一种yield的语法糖，他会``依次迭代后面跟着的可迭代对象的内容``

```javascript
const names = ['jack', 'rose', 'lily']
yield* names

相当于

yield 'jack'
yield 'rose'
yield 'lily'


// 简化代码
function* createIterator(arr) {
    yield* arr
}
```



## 

