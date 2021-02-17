---
typora-root-url: typera
---

## 对象、类、面向对象编程

### 8.1理解对象

创建自定义对象的方式通常是床架Object的一个新实例，然后再给他们添加属性和方法

### 8.1.1属性的类型

#### 数据属性

| 名称             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| [[Configurable]] | 属性是否可以通过delete删除冰重新定义，是否可以修改它的特效，以及是否可以把它作为访问器属性。默认直接定义在对象上的属性的这个特性都是true |
| [[Enumberable]]  | 属性是否可以通过for-in循环返回。默认直接定义在对象上的属性的这个特性都是true |
| [[Writable]]     | 表示属性的值是否可以被修改。默认直接定义在对象上的属性的这个特性都是true |
| [[Value]]        | 包含属性实际的值。默认值是undefined                          |

```js
//创建名为name的属性，writable设置false，非严格模式下重新赋值会被忽略，严格模式抛出错误。
let person = {};
Object.defineProperty(person,'name',{
    // configurable:true,
    // enumerable:true,
    writable:false,
    value:'Matt'
});
console.log(person.name);//'Matt'
person.name = 'Anni';
console.log(person.name);//'Matt'
```

```js
//创建名为name的属性，configurable设置为false，非严格模式下删除会被忽略，严格模式抛出错误。
let person = {};
Object.defineProperty(person,'name',{
    configurable:false,
    // enumerable:true,
    // writable:false,
    value:'Matt'
});
console.log(person.name);//'Matt'
delete person.name;
console.log(person.name);//'Matt'
```

```js
//一个属性被定义为不可配置之后，就不能变成可配置的了。再次调用defineProperty()并修改任何非writrable属性都会导致错误
let person = {};
Object.defineProperty(person,'name',{
    configurable:false,
    value:'Matt'
});
Object.defineProperty(person,'name',{
    configurable:true,
    value:'Matt'
});
```

#### 访问器属性
| 名称             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| [[Configurable]] | 属性是否可以通过delete删除冰重新定义，是否可以修改它的特效，以及是否可以把它作为访问器属性。默认直接定义在对象上的属性的这个特性都是true |
| [[Enumberable]]  | 属性是否可以通过for-in循环返回。默认直接定义在对象上的属性的这个特性都是true |
| [[Get]]          | 获取函数，在读取属性时调用。默认值是undefined                |
| [[Set]]          | 设置函数，在读取写入属性时调用。默认值是undefined            |

```js
//year_下划线表示不希望在对象方法的外部被访问（私有属性)。另一属性year被定义成为一个访问器属性，其中获取函数简单地返回year_的值，设置函数会做一些计算已决定正确的版本。
let book = {
    year_:2017,
    edition:1
};
Object.defineProperty(book,'year',{
    get(){
        return this.year_;
    },
    set(newValue){
        if(newValue>this.year_){
            this.year_ = newValue;
            this.edition += newValue - 2017;
        }
    }
});
book.year = 2019;
console.log(book.edition);
```

### 8.1.2定义多个属性

| 方法                      | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Object.defineProperties() | 两个参数，要为之添加或修改属性的对象和另一个描述符对象，其属性与要添加或修改的属性一一对应。 |

```js
//与上个例子效果一致
let book = {};
Object.defineProperties(book,{
    year_:{
        value:2017
    },
    edition:{
        value:1
    },
    year:{
        get(){
            return this.year_
        },
        set(newValue){
            if(newValue>this.year_){
                this.year_ = newValue;
                this.edition += newValue - 2017;
            }
    }
})
```

### 8.1.3读取属性的特性

| 方法                               | 说明                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| Object.getOwnPropertyDescriptor()  | 获取指定属性的属性描述符。接收两个参数：属性所在的对象和要取得其描述的属性名 |
| Object.getOwnPropertyDescriptors() | 在每个属性上都调用bject.getOwnPropertyDescriptor()方法并在一个新对象中返回他们 |

```js

let book = {};
Object.defineProperties(book,{
    year_:{
        value:2017
    },
    edition:{
        value:1
    },
    year:{
        get(){
            return this.year_
        },
        set(newValue){
            if(newValue>this.year_){
                this.year_ = newValue;
                this.edition += newValue - 2017;
            }
        }
    }
});
console.log(book)//{year_: 2017, edition: 1}
let descriptor = Object.getOwnPropertyDescriptor(book,'year_');
console.log(descriptor.value);//2017
console.log(descriptor.configurable);//false
console.log(typeof descriptor.get);//undefined

let descriptor = Object.getOwnPropertyDescriptor(book,'year')
console.log(descriptor.value);//undefined
console.log(descriptor.enumerable);//false
console.log(typeof descriptor.get);//function

//
console.log(Object.getOwnPropertyDescriptor(book));
// {
//     "year_":{
//     	"value":2017,"writable":false,"enumerable":false,"configurable":false
// 	},
//     "edition"{
//         "value":1,"writable":false,"enumerable":false,"configurable":false
//     },
//     "year":{
//         "enumerable":false,"configurable":false
//     }
// }

```

### 8.1.4合并对象

| 方法            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| Object.assign() | 接收一个目标对象和一个或者多个源对象作为参数，然后将每个源对象中可枚举(Object.propertyIsEnumerable()为true)和自有(Object.hasOwnProperty()为true)属性复制到目标对象。对于每个复核条件的属性，这个方法会是用源对象上的[[Get]]取得属性的值，然后是用目标对象上的[[Set]]设置属性的值 |

```js
let dest,src,result;
//简单复制
dest = {};
src = { id: 'src' };

result = Object.assign(dest,src);
//Object.assign修改目标对象
//也会返回修改后的目标对象
console.log(dest === result);//true
console.log(dest !== src);//true
console.log(result);//{id:src}
console.log(dest);//{id:src}

//多个源对象
dest = {};
result = Object.assign(dest,{a:'foo'},{b:'bar'});
console.log(result);//{a: "foo", b: "bar"}

//获取函数与设置函数
dest = {
    set a(val){
        console.log('dest setter');
    }
}
src = {
    get a(){
        console.log('src getter');
        return 'foo';
    }
}
Object.assign(dest,src);
//调用src的获取方法
//调用dest的设置方法并传入参数“foo”
//因为这里的设置函数不执行赋值操作
//所以实际上并没有把值转移过来
console.log(dest);
//src getter
//dest setter
//{ set a(val){...}}
```

Object.assign()实际上对每个源对象执行的是浅复制。如果多个源对象都有相同的属性，则使用最后一个复制的值。此外，从源对象访问器属性取得的值，比如获取函数，会作为一个静态值复制给目标对象。换句话说，不能再两个对象间转移获取函数和设置函数。

```js
let dest,src,result;
//覆盖属性
dest = {id:'dest'};
result = Object.assign(dest,{id:'src1',a:'foo'},{id:'src2',b:'bar'});
//Object.assign();会覆盖重复的属性
console.log(result);//{id:'src2',a:'foo',b:'bar'}

//可以通过目标对象上的设置函数观测到覆盖的过程
dest = {
    set id(x){
        console.log(x);
    }
}
Object.assign(dest,{id:'first'},{id:'second'},{id:'third'});
//first
//second
//third

//对象引用
dest = {};
src = {a:{}};
Object.assign(dest,src);

//浅复制意味着支付至对象的引用
console.log(dest);//{a:{}}
console.log(dest.a === src.a);//true

//如果赋值期间出错，则操作会终止并退出，同时抛出错误。Object.assign()没有“回滚”之前的赋值的概念。
let dest,src,result;
//错误处理
dest = {};
src = {
    a:'foo',
    get b(){
        //Object.assign()在调用这个获取函数时会抛出错误
        throw new Error();
    },
    c:'bar'
};
try{
    Object.assign(dest,src);
}catch(e){}
//Object.assign()没办法回滚已经完成的修改
//因此在抛出错误之前，目标对象上已经完成的修改会继续存在
console.log(dest);//{a:foo}
```

### 8.1.5对象标识及相等判定

在ES6之前，有些特殊情况即使是===操作符也不能判定，要判断NaN的相等性，必须使用isNaN()

ES6新增了Object.is();

| 方法        | 说明         |
| ----------- | ------------ |
| Object.is() | 接收两个参数 |

```js
Object.is(isNaN,isNaN);//true
Object.is({},{});//false
Object.is("2",2);//false
Object.is(true,1);//false
```

### 8.1.6增强的对象语法

```js
//属性值简写
//原始
let name = 'Matt';
let person = {
    name:name
};
console.log(person);//{name:'Matt'}

//简写
let name = 'Matt';
let person = {
    name
};
console.log(person);//{name:'Matt'}

//代码压缩
function makePerson(name){
    return {
        name
    };
}
// 等价于
// function makePerson(a){
//     return {
//         a
//     };
// }
let person = makePerson('Matt');
console.log(person.name);//'Matt'
```

```js
//可计算属性
//原始
const nameKey = 'name';
const ageKey = 'age';
const jobKey = 'job';
let person = {};
person[nameKey] = 'Matt';
person[ageKey] = 27;
person[jobKey] = 'Soft';
console.log(person);//{name:'Matt',age:27,job:'Soft'}

//使用计算属性
const nameKey = 'name';
const ageKey = 'age';
const jobKey = 'job';
let person = {
    [nameKey]:'Matt',
    [ageKey]:27,
    [jobKey]:'Soft'
}
console.log(person);//{name:'Matt',age:27,job:'Soft'}

//在实例化时求值
const nameKey = 'name';
const ageKey = 'age';
const jobKey = 'job';
let num = 0;
function getKey(key){
    return '${key}_${num++}';
}
let person = {
    [getKey(nameKey)]:'Matt',
    [getKey(ageKey)]:27,
    [getKey(jobKey)]:'Soft'
}
console.log(person);//{name_0:'Matt',age_1:27,job_2:'Soft'}
```

```js
//简写方法名
//原始
let person = {
    sayName:function(name){
        console.log('My name is ${name}');
    }
};
person.sayName('Matt');//My name is Matt
//简写
let person = {
    sayName(name){
        console.log('My name is ${name}');
    }
};
//
let methodskey = 'sayName';
let person = {
    [methodskey](name){
        console.log('My name is ${name}');
    }
};
```



### 8.1.7对象解构

```js
let person = {
    name:'Matt',
    age:27
}
//原始
let perosnName = person.name,
    personAge = person.age;
console.log(personName,persoAge);

let {name:personName,age:personAge} = person;
console.log(personName,persoAge);

let {name,age} = person;
console.log(name,age);

let {name,job} = person;
console.log(name,job);

let {name,job='Soft'} = person;
console.log(name,job);
```



## 8.2创建对象

### 8.2.2 工厂模式

```js
function createPerson(name,age){
    let o = new Object();
    o.name = name;
    o.age = age;
    return o;
}
```

### 8.2.3构造函数模式

```js
function Person(name,age){
    this.name = name;
    this.age = age;
    this.sayName = function(){
        console.log(this.name);
    }
}
let person1 = new Person('Matt',27);
let person2 = new Person('Annie',27);

person1.sayName();//'Matt'
person2.sayName();//'Annie'

/*
与上面一个例子区别是：
没有显式的创建对象
属性和方法直接赋值给了this
没有return
*/

/*
new操作符的步骤
1、创建一个新对象
2、Prototype指向构造函数的prototype
3、this指向新对象
4、执行构造函数内部的代码
5、如果构造函数返回非空对象，则返回改对象；否则，返回刚创建的对象
*/

//或者写成
let Person = function(name,age){
    this.name = name;
    this.age = age;
    this.sayName = function(){
        console.log(this.name);
    }
}

//对于做同一个事情的，没必要定义两个不同的妇女刺痛实例。因此可以把函数定义转移到构造函数外部
function Person(name,age){
    this.name = name;
    this.age = age;
    this.sayName = sayName;
}

function sayName(){
    console.log(this.name);
}
```

### 8.2.4原型模式

每个函数都会创建一个prototype属性，这个属性是一个对象，包含应该有特定引用类型的实例共享的属性和方法。

使用原型对象的好处是，在它上面定义的属性和方法可以被对象实例共享。

原来在构造函数中直接赋给对象实例的值，可以直接赋值给它们的原型

```js
function Person(){};
//let Person = function(){};

Person.prototype.name = 'Alle';
Person.prototype.age = 27;
Person.prototype.sayName = function(){
    console.log(this.name);
}

let person1 = new Person();
person1.sayName();//'Alle'

let person2 = new Person();
person2.sayName();//'Alle'

//所有的sayName()方法都直接添加到了Personde prototype属性上，构造函数体重什么也没有。
//这种原型模式定义的属性和方法都是由所有的实例共享的。
//因此person1和person2访问的都是相同的属性和相同的sayName()函数。
```

![](/1.jpg)

Person.prototype指向原型对象，而Person.prototype.constructor指回Person构造函数。

原型对象包含constructor属性和其它后来添加的属性。

Person的两个实例person1和person2都只有一个内部属性指回Person.prototype，且两个构造函数之间没有直接关系。



在通过对象访问属性时，会按照这个属性的名称开始搜索。搜索开始于对象实例本身。如果这个实例上发现了给定的名称，则返回该名称对应的值。如果没有找到这个属性，则会沿着指针进入原型对象，然后再原型对象上找到属性后，再返回对应的值。



如果在实例上添加了一个与原型对象中同名的属性，那就会在实例上创建这个属性，这个属性会遮盖原型对象上的属性。



```js
function Person(){};
Person.prototype.name = 'Nicholas';
Person.prototype.age = 27;
Person.prototype.sayName = function(){
    console.log(this.name);
};
let person1 = new Person();
let person2 = new Person();

person1.name = 'Greg';
console.log(person1.name);//'Greg'--来自实例
console.log(person2.name);//'Nicholas'--来自原型

//只要给对象实例添加一个属性，这个属性会遮蔽原型对象上的同名属性，也就是虽然不会修改它，但会屏蔽对它的访问，即使实例上把这个属性设置为null，也不会恢复它和原型之间的关系。但是使用delete操作符完全可以删除实例上的这个属性，从而让标识符解析过程能够继续搜索原型对象。

delete person1.name;
console.log(person1.name);//'Nicholas'--来自原型
```

```js
//hasOwnProperty()方法用于确定某个属性是在实例上还是原型对象上。这个方法继承自Object的，会在属性存在于调用它的对象实例上时返回true
function Person(){};
Person.prototype.name = 'Nicholas';
Person.prototype.age = 27;
Person.prototype.sayName = function(){
    console.log(this.name);
};
let person1 = new Person();
let person2 = new Person();
console.log(person1.hasOwnProperty('name'));//false

person1.name = 'Greg';
console.log(person1.name);//'Greg'--来自实例
console.log(person1.hasOwnProperty('name'));//true

console.log(person2.name);//'Nicholas'--来自原型
console.log(person2.hasOwnProperty('name'));//false

delete person1.name;
console.log(person1.name);//'Nicholas'--来自原型
console.log(person1.hasOwnProperty('name'));//false
```

![](/2.jpg)



```js
//in操作符会在可以通过对象访问指定属性时返回true，无论该属性在实例上还是原型上
function Person(){};
Person.prototype.name = 'Nicholas';
Person.prototype.age = 27;
Person.prototype.sayName = function(){
    console.log(this.name);
};
let person1 = new Person();
let person2 = new Person();

console.log(person1.hasOwnProperty());//false
console.log('name' in person1);//true

person1.name = 'Greg';
console.log(person1.name);//'Greg'--来自实例
console.log(person1.hasOwnProperty('name'));//true
console.log('name' in person1);//true

console.log(person2.name);//'Nicholas'--来自原型
console.log(person2.hasOwnProperty('name'));//false
console.log('name' in person2);//true

delete person1.name;
console.log(person1.name);//'Nicholas'--来自原型
console.log(person1.hasOwnProperty('name'));//false
console.log('name' in person1);//true
```



```js
//hasPrototypeProperty():只在原型上时返回true，在实例上时返回false
function Person(){};
Person.prototype.name = 'Nicholas';
Person.prototype.age = 27;
Person.prototype.sayName = function(){
    console.log(this.name);
};

let person = new Person();
console.log(hasPrototypeProperty(person,'name'));//true
person.name = 'Greg';
console.log(hasPrototypeProperty(person,'name'));//false
```



```js
//Object.keys():获取所有可枚举的属性名称的字符串数组
//要获取所有实例属性，无论是否可枚举，使用Object.getOwnPropertyNames();
function Person(){};
Person.prototype.name = 'Nicholas';
Person.prototype.age = 27;
Person.prototype.sayName = function(){
    console.log(this.name);
};

console.log(Object.keys(Person.prototype));//['name','age','sayName']

let p = new Person();
p.name = 'MATT';
p.age = 12;
console.log(Object.keys(p));//['name','age']

//返回多一个不可枚举的属性contructor
console.log(Object.getOwnPropertyNames(Person.prototype);//['contructor',name','age','sayName']

```



```js
for-in和Object.keys()枚举顺序因不同浏览器而异。
Object.getOwnPropertyNames()以及Obejct.assign()枚举顺序确定
```

### 8.2.5 对象迭代

```js
//Object.values():返回对象值的数组，执行对象的浅复制
let o = {
    foo:'bar',
    baz:1,
    qux:{}
}
console.log(Object.values(o));//['bae',1,{}]
```

```js
//Object.entries()：返回键值对的数组,非字符串属性会被转为字符串输出，执行对象的浅复制
let o = {
    foo:'bar',
    baz:1,
    qux:{}
}
console.log(Object.entries(o));//[['foo','bar'],['baz',1],['qux',{}]]
```

```js
function Person(){};

let friend = new Person();
Person.prototype = {
    constructor:Person,
    name:'Matt',
    sayName(){
        console.log(this.name)
    }
}

friend.sayName();//抛错
//friend实例是在Person重写原型对象之前创建的。friend指向的原型还是最初的原型。
```

![](/3.jpg)

#### 原型的问题

```js
//多实例共享属性
function Person(){};
Person.prototype = {
    contructor:Person,
    name:'Matt',
    age:27,
    friends:['A','B'],
    sayName(){
        console.log(this.name);
    }
}
let person1 = new Person();
let person2 = new Person();

person1.firends.push('c');

console.log(person1.friends);//['A','B','C']
console.log(person2.friends);//['A','B','C']
console.log(person1.friends == person2.friends);//true
```

## 8.3继承

### 原型链

```js
function Super(){
    this.property = true;
}
Super.prototype.getSuperValue = function(){
    return this.property;
}

function Sub(){
    this.subProperty = false;
}
Sub.prototype = new Super();
Sub.prototype.getSubValue = function(){
    return this.subProperty;
}

let ins = new Sub();
console.log(ins.getSuperValue());//true
```

```js
//instanceof:实例函数是否属于构造函数
console.log(ins instanceof Sub);//true
console.log(ins instanceof Super);//true
console.log(ins instanceof Object);//true

//isPrototypeOf():原型链中是否包含某个原型
console.log(Object.prototype.isPrototypeOf(ins));//true
console.log(Super.prototype.isPrototypeOf(ins));//true
console.log(Sub.prototype.isPrototypeOf(ins));//true
```

```js
//需要覆盖父类的方法或者新增父类没有的方法。这些方法必须在原型赋值之后再添加到原型上。
function Super(){
    this.property = true;
}
Super.prototype.getSuperValue = function(){
    return this.property;
}
function Sub(){
    this.subproperty = true;
}

//继承Super
Sub.prototype = new Super();
//新方法
Sub.prototype.getSubValue = function(){
    return this.subproperty;
}
//覆盖
Sub.prototype.getSuperValue = function(){
    return false;
}
let person = new Sub();
console.log(person.getSuperValue());//false
```

```js
//需要覆盖父类的方法或者新增父类没有的方法。这些方法必须在原型赋值之后再添加到原型上。
function Super(){
    this.property = true;
}
Super.prototype.getSuperValue = function(){
    return this.property;
}
function Sub(){
    this.subproperty = true;
}
//新方法
Sub.prototype.getSubValue = function(){
    return this.subproperty;
}
//覆盖
Sub.prototype.getSuperValue = function(){
    return false;
}
//继承Super,此时会重新指向
Sub.prototype = new Super();

let person = new Sub();
console.log(person.getSuperValue());//true
```

```js
//以对象字面量方式创建原型方法会破坏之前的原型链
function Super(){
    this.property = true;
}
Super.prototype.getSuperValue = function(){
    return this.property;
}
function Sub(){
    this.subproperty = true;
}
//继承
Sub.prototype = new Super();
//字面量添加新方法，导致上一行无效
Sub.prototype = {
    getSubValue(){
        return this.subproperty;
    },
    someOtherMethods(){
        return false;
    }
}
let ins = new Sub();
console.log(ins.getSubValue());//抛错
```

#### 原型链的问题

```js
//属性通常在构造函数中定义，而不在原型中定义。
function Super(){
    this.colors = ['red','blue']
}
function Sub(){};
//继承
Sub.prototype = new Super();

let ins = new Sub();
ins.colors.push('yellow');
console.log(ins.colors);//['red','blue','yellow']

let ins2 = new Sub();
console.log(ins2.colors);//['red','blue','yellow']
```

### 盗用构造函数

```js
function Super(){
    this.colors = ['red','blue']
}
function Sub(){
    Super.call(this);
};

let ins = new Sub();
ins.colors.push('yellow');
console.log(ins.colors);//['red','blue','yellow']

let ins2 = new Sub();
console.log(ins2.colors);//['red','blue']
//call()或者apply()方法，Super在Sub实例的新对象的上下文中执行了，相当于Sub对象上运行了Super函数中的所有初始化代码。每个实例拥有自己的属性。
```

```js
//盗用构造函数可在子类构造函数中向父类构造函数传参
//为确保Super构造函数不会覆盖Sub定义的属性，可以在调用父类构造函数之后再给子类实例添加额外的属性
//子类不能访问父类原型上定义的方法

//构造函数
function Super(name){
    this.name = name;
}
//原型
Super.prototype.sayName = function(){
    console.log(this.name);
}
function Sub(){
    Super.call(this,'Matt');
    this.age = 27;
}

let ins = new Sub();
console.log(ins.name);//'Matt'
console.log(ins.age);//27
console.log(ins.sayName);//undefined
```

### 组合继承

使用原型链继承原型上的属性和方法，通过盗用构造函数继承实例属性。

这样可以把方法定义在原型上实现重用，又可以让每个实例拥有自己的属性。

```js
function Super(name){
    this.name = name;
    this.colors = ['red','blue'];
}
Super.prototype.sayName = function(){
    console.log(this.name)
}

function Sub(name,age){
    //继承属性
    Super.call(this,name);
    this.age = age;
}

//继承方法
Sub.prototype = new Super();

Sub.prototype.sayAge = function(){
    console.log(this.age);
}

let ins1 = new Sub('Matt',18);
ins1.colors.push('yellow');
console.log(ins1.colors);//['red','blue','yellow']
ins1.sayName();//Matt
ins1.sayAge();//18

let ins2 = new Sub('Gre',27);
console.log(ins2.colors);//['red','blue','yellow']
ins1.sayName();//Gre
ins1.sayAge();//27
```

### 原型式继承

```js
//Object.create()
//只传一个参数，作为新对象原型的对象
let person ={
    name:'张三',
    friends:['A']
}

let ap = Object.create(person);
ap.name = '李四';
ap.friends.push('B');
console.log(person.friends,person.name);//['A','B'],‘张三’

let ap2 = Object.create(person);
ap2.name = '王五';
ap2.friends.push('C');
console.log(person.friends,person.name);//['A','B','C'],‘张三’

//第二个参数：每个新增属性都通过各自的描述符来描述。以这种方式添加的属性会屏蔽原型对象上的同名属性
let person ={
    name:'张三',
    friends:['A']
}

let a3 = Object.create(person,{
    name:{
		value:'李四'
	},
	friends:{
		value:['A','B']
	}
});
console.log(person.friends,person.name);//["A"] "张三"
console.log(person.friends,person.name);//["A", "B"] "李四"
```

### 寄生式继承

```js
//新返回的对象具有person的所有属性和方法，还有新方法sayName
function createAnother(ori){
	let clone = Object(ori);
	clone.sayName = function(){
		console.log('hi')
	};
	return clone;
}

let person = {
	name:'Mat',
	age:27
}

let another = createAnother(person);
another.sayName();//'hi'
console.log(another.name,another.age);//Mat 27
```

### 寄生式组合继承

组合继承其实也存在效率问题。最主要的效率问题是父类构造函数始终会被调用两次：一次是在创建子类原型时调用，另一次在子类构造函数中调用。

```js
function Super(name){
	this.name = name;
	this.colors = ['A','B'];
}
Super.prototype.sayName = function(){
	console.log(this.name)
}

function Sub(name,age){
	Super.call(this,name); //第二次调用Super
	this.age = age;
}
Sub.prototype = new Super(); //第一次调用Super

Sub.prototype.constructor = Sub;
Sub.prototype.sayAge = function(){
	console.log(this.age)
}

let per = new Sub('Mat',27);
per.sayAge();//27
per.sayName();//'Mat'

//见下图，有两组name和colors属性：一组在实例上，一组在原型上
```

![](/4.jpg)

![](/5.jpg)

```js
function initPrototype(subType,superType){
	let prototype = Object(superType.prototype);
	prototype.constructor = subType;
	subType.prototype = prototype;
}
//接收两个参数：子类构造函数和父类构造函数。在这个函数内部，第一步是创建父类原型的一个副本。
//然后给返回的prototype对象设置constructor属性，解决由于重写导致默认constructor丢失的问题
//最后将新创建的对象赋值给子类型的原型链。
```

```js
function SuperType(name){
	this.name = name;
	this.colors = ['A','B'];
}
SuperType.prototype.sayName = function(){
	console.log(this.name)
}

function SubType(name,age){
	SuperType.call(this,name); 
	this.age = age;
}
initPrototype(SubType,SuperType);

SubType.prototype.sayAge = function(){
	console.log(this.age)
}

let per = new Sub('Mat',27);
per.sayAge();//27
per.sayName();//'Mat'
```



## 8.4类

###  8.4.1类定义

```js
//类声明和类表达式。
//类声明
class Person{}
//类表达式
const ani = class{}

//与函数表达式相比，类表达式在被求值前不能引用。类定义不能提升。
console.log(F1);//undefined
var F1 = function(){};
console.log(F1);//function(){}

console.log(F2);//function F2(){}
function F2(){}
console.log(F2);//function F2(){}

console.log(cls1);//undefined
var cls1 = class{};
console.log(cls1);//class{}

console.log(cls2);// 抛错 Cannot access 'cls2' before initialization
class cls2{};
console.log(cls2);//class cls2{}--s上面报错，这里执行不到


//与函数声明不同，函数受作用域限制，而类受块作用域限制。
{
	function F1(){};
	class cls1{};
}
console.log(F1);//ƒ F1(){}
console.log(cls1);//cls1 is not defined

```

#### 类的构成

```
//类可以包含构造函数方法、实例方法、获取函数、设置函数和静态类方法。

//空类定义，有效
class Foo{}
//有构造函数的类，有效
class Bar{
	constructor(){}
}
//有获取函数的类，有效
class Baz{
	get MyBaz(){}
}
//有静态方法的类，有效
class myQux(){}
```

```js
//类表达式的名称是可选的。在把类表达式赋值给变量后，可以通过name属性取得类表达式的名称字符串。但不能在类表达式作用域外部访问这个标识符

let Person = class PersonName {
	idei(){
		console.log(Person.name,PersonName.name)
	}
}

let per = new Person();
per.idei();//PersonName PersonName

console.log(Person.name);//PersonName
console.log(PersonName);//PersonName is not defined
```



### 8.4.2类构造函数

constructor关键字 用于在类定义块内部创建类的构造函数。方法名constructor会告诉解释器在使用new操作符创建类的新实例时，应该调用这个函数。构造函数的定义不是必须的，不定义构造函数相当于将构造函数定义为空函数。

```js
//使用new操作符实例化Person的操作等于使用new操作符调用其构造函数。
//使用new调用类的构造函数会执行如下操作
/*
在内存中创建一个新对象
这个新对象内部的Prototype指针被赋值为构造函数的prototype属性
构造函数内部的this被赋值为这个新对象
执行构造函数内部的代码（给新对象添加属性）
如果构造函数返回非空对象，则返回该对象；否则返回刚创建的新对象
*/

class Ani{}

class Person{
	constructor(){
		console.log('person constructor')
	}
}

class Vegetable{
	constructor(){
		this.color = 'red'
	}
}

let a = new Ani()
let b = new Person();//'person constructor'
let c = new Vegetable();
console.log(c.color);//'red'

//类实例化时传入的参数会作为构造函数的参数。如果不需要参数，则类名后面的括号也是可选的
class Person{
	constructor(name){
		console.log(arguments.length)
		this.name = name||null;
	}
}

let p1 = new Person;//0
console.log(p1.name);//null

let p2 = new Person();//0
console.log(p2.name)//null

let p3 = new Person('jake');//1
console.log(p3.name);//'jake'
```

```js
class Person{
	constructor(override){
		this.foo = 'foo';
		if(override){
			return{
				bar:'bar'
			}
		}
	}
}
let p1 = new Person()
let p2 = new Person(true)

console.log(p1)//Person {foo: "foo"}
console.log(p1 instanceof Person)//true

console.log(p2)//{bar: "bar"}
console.log(p2 instanceof Person)//false
```

类构造函数和构造函数的主要区别是，调用构造函数必须使用new操作符。而普通构造函数如果不适用new调用，name就会以全局的thi是（通常是指window）作为内部对象。调用类构造函数时如果忘了使用new则会抛错

```js
function Person(){};
class ani{};

//把window作为this来构建实例
let p = Person();
//
let a = ani{};//Class constructor ani cannot be invoked without 'new'

//类构造函数在实例化后，会变成普通的实例方法
class Person{}
let p1 = new Person();
p1.constructor();//Class constructor Person cannot be invoked without 'new'
//使用new创建一个新实例
let p2 = new p1.constructor();
```

ES中没有类这个类型，类是一个特殊函数。

```js
class Person{}
console.log(typeof Person);//function

//类标签也有prototype属性，这个原型也有一个constructor属性指向类本身。
console.log(Person.prototype.constructor == Person);//true

//与普通构造函数一样，可以使用instanceof检查构造函数原型是否存在于实例的原型链中
let p = new Person();
console.log(p isinstanceof Person);//true

//类可作为参数传递
let classList = [
    class{
        contructor(id){
            this.id_ = id;
            console.log('this id ${this.id_}')
        }
    }
]
```

### 8.4.3实例、原型和类成员

#### 实例成员

每次通过new调用类标识符时，都会执行类构造函数。

每个实例都应对应一个唯一的成员对象，所有成员都不会再原型上共享。

```js
class Person{
    constructor(){
        this.name = new String('Kack');
        this.sayName = ()=>console.log(this.name);
        this.nickNames = ['Jake','Gog']
    }
}
let p1 = new Person();
let p2 = new Person();

p1.sayName();//Jack
p2.sayName();//Jack

console.log(p1.name == p2.name);//false
console.log(p1.sayName == p2.sayName);//false
console.log(p1.nickNames == p2.nickNames);//false

p1.name = p1.nickNames[0];
p2.name = p1.nickNames[1];

p1.sayName();//Jack
p2.sayName();//Gog
```

#### 原型方法与访问器

为了在实例间共享方法，类定义语法把类块中定义的方法作为原型方法。

```js
class Person{
	constructor(){
        //添加到this的所有内容都会存在不同的实例上
		this.locate = ()=>console.log('instance');
        //在类块中定义的所有内容都会在类的原型上
		locate(){
			console.log('prototype')
		}
	}
}

let p = new Person();

p.locate();//instance
Person.prototype.locate();//prototype

//可以把方法定义在类构造函数中或者类块中，不能在类块中给原型添加原始值或对象
class Person{
    name:'Kake'
}
//Unexpected token :

//类方法等同于对象，因此可以使用字符串、符号或者计算的值作为键
const symbolKey = Symbol('synbolKey');

class Person{
    stringKey(){}
    [symbolKey](){}
    ['computed'+'Key'](){}
}

//类定义也支持获取和设置访问器
class Person{
    set name(newName){
        this.name = newName
    }
    get name(){
        return this.name
    }
}

let p = new Person();
p.name = 'Jake';
console.log(p.name);//Jake
```

#### 静态类方法

可以在类上定义静态方法，通常执行不特定于实例的操作，每个类上只能有一个静态成员

```js
class Person{
    constructor(){
        this.locate = ()=>console.log('instance',this)
    }
	locate(){
		console.log('prototype',this)
	}
	//定义在类本身上
	static locate(){
		console.log('class',this)
	}
}

let p = new Person();
p.locate();//instance Person{}
Person.prototype.locate();//prototype {contructor:...}
Person.locate();//class classPerson{}

//静态类方法非常适合作为工厂函数
class Person{
    constructor(age){
		this.age_ = 123;
	}
	sayAge(){
		console.log(this.age_)
	}
	static create(){
		return new Person(Math.floor(Math.random()*100))
	}
}

console.log(Person.create());//Person {age_: 123}
```

#### 非函数原型和类成员

```js
class Person{
    sayName(){
        console.log(`${Person.greating}`,`${this.name}`)
    }
}
Person.greating = 'my name is';
Person.prototype.name = 'Jake';
let p = new person();
p.sayName();//my name is Jake
```



### 8.4.4 继承

#### 继承基础

```js
//使用extends,可以继承任何拥有【Contruct】和原型的对象

//继承类
class Vehicle {};
class Bus extends Vehicle{};

let b = new Bus();
console.log(b instanceof Bus);//true
console.log(b instanceof Vehicle);//true

//继承普通构造函数
function Person(){};
class Engineer extends Person{}

let e = new Engineer();
console.log(e instanceof Engineer);//true
console.log(e instanceof Person);//true

//
class Vehicle{
	identity(id){
		console.log(id,this)
	}
	static identity(id){
		console.log(id,this)
	}
}

class Bus extends Vehicle{}

let v = new Vehicle();
let b = new Bus();

v.identity('v')//v Vehicle{}
b.identity('b')//b Bus{}

Bus.identity('bus')//bus class Bus
Vehicle.identity('vehicle')//vehicle class Vehicle
```

#### 构造函数、super()、HomeObject

es6给类构造函数和静态方法添加了内部特效HomeObject，这个特性是一个指针，指向定义该方法的对象。这个指针是自动赋值，而且只能在JS引擎内部访问，super始终会定义为HomeObject的原型

```js
//派生类的方法可以通过super关键字引用他们的原型，这个关键字只能在派生类中使用，且仅限于类构造函数、实例方法和静态方法内部。
//在类构造函数中使用super可以调用父类构造函数

class Vehicle{
    constructor(){
		this.hisEng = true;
	}
}

class Bus extends Vehicle{
	constructor() {
		//不要在super之前引用this，否则会抛ReferenceError
	    super();//相当于super.constructor()
		
		console.log(this instanceof Vehicle)//true
		console.log(this)//Bus {hisEng:true}
	}
}

new Bus();

//在静态方法中可以通过super调用继承的类上定义的静态方法
class Vehicle{
	static identity(){
		console.log('vehicle')
	}
}
class Bus extends Vehicle{
	static identity(){
		super.identity()
	}
}

Bus.identity()//vehicle
```

使用super时要注意几个问题

```js
//super只能在派生类构造函数和静态方法中使用
class Vehicle{
	constructor(){
		super();//'super' keyword unexpected here
	}
}

//不能单独使用super关键字
class Vehicle{}
class Bus extends Vehicle{
	constructor(){
		console.log(super)//'super' keyword unexpected here
	}
}

//调用super会调用父类构造函数，并将返回的实例赋值给this
class Vehicle{}
class Bus extends Vehicle{
	constructor(){
		super();
		console.log(this instanceof Vehicle)
	}
}
new Bus()//true

//super的行为如同调用构造函数，如果需要给父类构造函数传参，则需要手动传入
class Vehicle{
	constructor(ll){
		this.ll = ll;
	}
}
class Bus extends Vehicle{
	constructor(LL){
		super(LL);
	}
}
console.log(new Bus('123'))//Bus {ll: "123"}

//派生类如果没有定义构造函数，则在实例化时会调用super()
class Vehicle{
	constructor(ll){
		this.ll = ll;
	}
}
class Bus extends Vehicle{}
console.log(new Bus('123'))//Bus {ll: "123"}

//在类构造函数中，不能再调用super()之前引用this
class Vehicle{}
class Bus extends Vehicle{
	constructor() {
	    console.log(this)
	}
}
new Bus();
//Must call super constructor in derived class before accessing 'this' or returning from derived constructor

//如果在派生类中显示定义了构造函数，要么必须在其中调用super()，要么必须在其中返回一个对象
class Vehicle{}
//未定义构造函数
class Car extends Vehicle{}
//定义构造函数，调用super
class Bus extends Vehicle{
	constructor(){
		super()
	}
}
//定义构造函数,返回对象
class Van extends Vehicle{
	constructor(){
		return {}
	}
}
console.log(new Car())//Car {}
console.log(new Bus())//Bus {}
console.log(new Van())//{}
```

#### 抽象基类

定义这样一个类，可供其他类继承、但本身不会被实例化

```js
class Vehicle{
	constructor() {
	    console.log(new.target)
		if(new.target == Vehicle){
			throw new Error('Vehicle can not be instantiated')
		}
	}
}
class Bus extends Vehicle{}

new Bus();
new Vehicle();//Error:Vehicle can not be instantiated

//在抽象基类构造函数中检查，可以要求派生类必须定义某个方法
class Vehicle{
	constructor() {
		if(new.target == Vehicle){
			throw new Error('Vehicle can not be instantiated')
		}
		if(!this.foo){
			throw new Error('派生类必须定义foo函数')
		}
		console.log('成功')
	}
}
class Bus extends Vehicle{
	foo(){}
}
class Van extends Vehicle{}

new Bus();//成功
new Van();//派生类必须定义foo函数
```

#### 继承内置类型

```js
class SuperArr extends Array{}
let a = new SuperArr();

console.log(a instanceof SuperArr)//true
console.log(a instanceof Array)//true
```

#### 类混入

把不同类的行为集中到一个类是一种常见的JS模式。

```js
class Vehiecle {}

let FooMixin = (superClass) => class extends superClass {
	foo(){
		console.log('foo')
	}
}

let BarMixin = (superClass) => class extends superClass {
	bar(){
		console.log('bar')
	}
}

let BazMixin = (superClass) => class extends superClass {
	baz(){
		console.log('baz')
	}
}

class Bus extends FooMixin(BarMixin(BazMixin(Vehiecle))){}

let b = new Bus();
b.foo();//foo
b.bar();//bar
b.baz();//baz
```

