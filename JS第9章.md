## 代理与反射

### 9.1代理基础

#### 创建空代理

代理是使用proxy构造函数创建的。这个构造函数接收两个参数：目标对象和处理程序对象。缺少其中一个参数都会抛出typeError。

```js
const target = {
    id:'target'
}

const handler = {};
const proxy = new Proxy(target,handler);
//id会访问同一个值
console.log(target.id,proxy.id)//target target

//给目标属性赋值会反映在两个对象上
//因为两个对象访问的是同一个值
target.id = 'foo';
console.log(target.id,proxy.id)//foo foo

//给代理属性赋值会反映在两个对象上
//因为这个赋值会转移到目标对象
proxy.id = 'bar';
console.log(target.id,proxy.id)//bar bar

//hasOwnProperty()方法在两个地方都会应用到目标对象
console.log(target.hasOwnProperty('id'))//true
console.log(proxy.hasOwnProperty('id'))//true

//Proxy.prototype是undefiend
//因此不能使用instanceof操作符
//这里存疑，文章说这里报错，实际上有输出false
console.log(target instanceof Proxy)//false
console.log(proxy instanceof Proxy)//false

//严格相等可以用来区分代理和目标
console.log(target === proxy)//false
```

#### 捕获器

```js
//定义一个get捕获器
const target = {
    foo:'bar'
}

const handler = {
    get(){
        return 'handler override'
    }
}

const proxy = new Proxy(target,handler)

//在代理对象上执行这些操作才会触发捕获器，在目标对象上执行这些操作仍然会产生正常的行为
console.log(target.foo);//bar
console.log(proxy.foo);//handler override

console.log(target[foo]);//bar
console.log(proxy[foo]);//handler override
```

#### 捕获器参数和反射API

```js
//get()捕获器会接收到目标对象、要查询的属性和代理对象三个参数
const target = {
    foo:'bar'
}
const handler = {
    get(trapTarget,property,receiver){
        console.log(trapTarget === target);
        console.log(property);
        console.log(receiver === proxy);
    }
}
const proxy = new Proxy(target,handler);
proxy.foo;
//true
//foo
//true

const target = {
    foo:'bar'
}
const handler = {
    get(trapTarget,property,receiver){
        return trapTarget[property]
    }
}
const proxy = new Proxy(target,handler);
console.log(target.foo);
console.log(proxy.foo);
```

```js
//并不需要手动重建原始行为，可以通过调用全局Relect对象（封装了原始行为）的同名方法
//处理程序对象中所有可以捕获的方法都有对应的Relect(反射)API方法。

const target = {
    foo:'bar'
}
const handler = {
    get(){
        return Reflect.get(...arguments);
    }
    //简写
    //get:Reflect.get
}
const proxy = new Proxy(target,handler);
console.log(target.foo);//bar
console.log(proxy.foo);//bar


//简写,不需要定义处理程序对象
const target = {
    foo:'bar'
}
const proxy = new Proxy(target,Reflect);
console.log(target.foo);//bar
console.log(proxy.foo);//bar

//在某个指定属性被访问时，会对返回的值进行修饰
const target = {
    foo:'bar',
    baz:'qux'
}
const handler = {
    get(trapTarget,property,receiver){
        let decoration = '';
        if(property === 'foo'){
            decoration = '!!!';
        }
        return Reflect.get(...arguments)+decoration;
    }
}
const proxy = new Proxy(target,Reflect);
console.log(target.foo);//bar
console.log(proxy.foo);//bar!!!

console.log(target.baz);//qux
console.log(proxy.baz);//qux

```

#### 捕获器不变式

```js
//目标对象上有一个不可配置的且不可写的数据属性，那么捕获器返回一个与该属性不同的值时，会抛出TypeError
const target = {};
Object.defineProperty(target,'foo',{
    configurable:false,
    writable:false,
    value:'bar'
});
const handler = {
    get(){
        return 'qux'
    }
};
const proxy = new Proxy(target,handler);
console.log(proxy.foo);//TypeError
```

#### 可撤销代理

```js
//有时候需要中代理对象和目标对象之间的联系使用new Proxy()创建的普通代理会在代理对象的声明周期内一直持续存在
//Proxy暴露了revocable()方法，这个方法支持撤销代理对象与目标对象的关联。
//撤销操作是不可逆的。
//撤销函数调用多少次结果都一样
//撤销代理后再次调用代理会抛出TypeError

const target = {
    foo:'bar'
}
const handler = {
    get(){
        return 'inter';
    }
}

const {proxy,revoke} = Proxy.revocable(target,handler);
console.log(proxy.foo);//inter
console.log(target.foo);//bar

revoke();
console.log(proxy.foo);//TypeError
```

#### 使用反射API

很多反射方法返回“状态标记”的布尔值，表示意图执行的操作是否成功

```js
//初始代码
const o = {};
try{
    Object.defineProperty(o,'foo','bar');
    console.log('success');
}catch(e){
    console.log('failure');
}
//重构代码
const o = {};
if(Reflect.defineProperty(o,'foo',{value:'bar'})){
    console.log('success');
}else{
    console.log('failure');
}
//以下的反射方法都提供状态标记
Reflect.defineProperty()
Reflect.preventExtensions()
Reflect.setPrototypeOf()
Reflect.set()
Reflect.deleteProperty()

//用一等函数代替操作符
Reflect.get()//代替对象属性访问操作
Reflect.set()//代替=赋值操作符
Reflect.has()//代替in或者with操作符
Reflect.deleteProperty()//代替delete操作符
Reflect.construct()//代替new操作符
```

#### 代理另一个代理

```js
const target = {
    foo:'bar'
};
const firstProxy = new Proxy(target,{
    get(){
        console.log('first proxy');
        return Reflect.get(...arguments)
    }
});
const secondProxy = new Proxy(firstProxy,{
    get(){
        console.log('second proxy');
        return Reflect.get(...arguments)
    }
});
console.log(secondProxy.foo);
//second proxy
//first proxy
//bar
```

#### 代理的不足

```js
//代理中的this
const target = {
    thisValEqualProxy(){
        return this === proxy;
    }
}
const proxy = new Proxy(target,{});

console.log(proxy.thisValEqualProxy);//true
console.log(target.thisValEqualProxy);//false

//如果目标对象依赖于对象标识，那就可能遇到问题
const wm = new WerkMap();
class User{
    constructor(userid){
        wm.set(this,userid)
    }
    set id(userid){
        wm.set(this,userid)
    }
    get(userid){
        return wm.get(this)
    }
}
//由于这个实现依赖于User实例的对象标识，在这个实例被代理的情况下就会出现问题
const user = new User(123);
console.log(user.id);

const UserInstanceofProxy = new Proxy(user,{});
console.log(UserInstanceofProxy.id);//undefined
```

```js
//一些内置类型可以能会依赖代理无法控制的机制，导致在代理上调用某些方法会出错
//典型例子Date，根据ES规范，Date类型方法的执行依赖this值上的内部槽位[NumberDate],代理对象上不存在这个内部槽位，这个槽位也不能通过普通的get和set操作访问到。
const target = new Date();
const proxy = new Proxy(target,{});
console.log(proxy instanceof Date);
proxy.getDate();//TypeError:'this' is not a Date object
```



### 9.2代理捕获器与反射方法

#### get()

```js
//get()捕获器会在获取属性值的操作中被调用，对应的反射API方法为Reflect.get()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    get(target,property,reciver){
        console.log('get()');
        return Reflect.get(...arguments)
    }
})
prxoy.foo;//'get()'

/*
1、返回值：返回值无限制
2、拦截的操作
	proxy.property
	proxy[property]
	Object.create(proxy)[property]
	Reflect.get(proxy,property,receiver)
3、捕获器处理程序参数
	target：目标对象
	property：引用的目标对象上的字符串键属性
	receiver：代理对象或集成代理对象的对象
4、捕获器不变式
	如果target.property不可写且不可配置，则处理程序返回的值必须与target.property匹配
	如果target.property不可配置且[Get]特性为undefined，处理程序返回的值也必须是undefined
*/
```

#### set()

```js
//set()捕获器会在设置属性值的操作中被调用，对应的反射APId的方法为：Reflect.set()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    set(target,property,value,reciver){
        console.log('set()');
        return Reflect.set(...arguments)
    }
})
prxoy.foo;//'set()'

/*
1、返回值：返回true表示成功；返回false表示失败，严格模式下会抛出TypeError
2、拦截的操作
	proxy.property  = value
	proxy[property] = value
	Object.create(proxy)[property] = value
	Reflect.set(proxy,property,value,receiver)
3、捕获器处理程序参数
	target：目标对象
	property：引用的目标对象上的字符串键属性
	value：要赋给属性的值
	receiver：接收最初赋值的对象
4、捕获器不变式
	如果target.property不可写且不可配置，则不能修改目标属性的值
	如果target.property不可配置且[Set]特性为undefined，则不能修改目标属性的值。
	在严格模式下，处理程序返回false会抛出TypeError
*/
```

#### has()

```js
//has()捕获器在in操作符中被调用。对应的反射API方法为Reflect.has()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    has(target,property){
        console.log('has()');
        return Reflect.has(...arguments)
    }
})
'foo' in proxy //has()

/*
1、返回值：必须返回布尔值，表示属性是否存在。返回非布尔值会被转型为布尔值
2、拦截的操作
	property in proxy.property
	property in Object.create(proxy)
	with(proxy){(property);}
	Reflect.has(proxy,property)
3、捕获器处理程序参数
	target：目标对象
	property：引用的目标对象上的字符串键属性
4、捕获器不变式
	如果target.property存在且不可配置，则处理程序必须返回true
	如果target.property存在且目标对象不可扩展，则处理程序必须返回true
*/
```

#### defineProperty()

```js
//defineProperty()捕获器会在Object.defineProperty()中被调用。
//对应的反射API方法为Reflect.defineProperty()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    defineProperty(target,property,descripttor){
        console.log('defineProperty');
        return Reflect.defineProperty(...arguments)
    }
})
Object.defineProperty(proxy,'foo',{value:'bar'});//defineProperty()

/*
1、返回值：必须返回布尔值，表示属性是否定义成功。返回非布尔值会被转型为布尔值
2、拦截的操作
	Object.defineProperty(proxy,property,descripttor)
	Reflect.defineProperty(proxy,property,descripttor)
3、捕获器处理程序参数
	target：目标对象
	property：引用的目标对象上的字符串键属性alue
	descripttor：包含可选的enumerable，configuable，writable，value，get和set定义的对象
4、捕获器不变式
	如果目标对象不可扩展，则无法定义属性
	如果目标对象有一个可配置的属性，则不能添加同名的不可配置属性
	如果目标对象有一个不可配置的属性，则不能添加同名的可配置属性
*/
```

#### getOwnPropertyDescriptor()

```js
//getOwnPropertyDescriptor()捕获器会在Object.getOwnPropertyDescriptor()中被调用。
//对应的反射API方法为Reflect.getOwnPropertyDescriptor()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    getOwnPropertyDescriptor(target,property){
        console.log('getOwnPropertyDescriptor');
        return Reflect.getOwnPropertyDescriptor(...arguments)
    }
})
Object.getOwnPropertyDescriptor(proxy,'foo');//getOwnPropertyDescriptor()

/*
1、返回值：必须返回对象，或者在属性不存在时返回undefined
2、拦截的操作
	Object.getOwnPropertyDescriptor(proxy,property)
	Reflect.getOwnPropertyDescriptor(proxy,property)
3、捕获器处理程序参数
	target：目标对象
	property：引用的目标对象上的字符串键属性alue
4、捕获器不变式
	如果自有的target.property存在且不可配置，则处理程序必须返回一个表示该属性存在的对象
	如果自有的target.property存在且可配置，则处理程序必须返回表示该属性可配置的对象
	如果自有的target.property存在且target不可扩展，则处理程序必须返回一个表示该属性存在的对象
	如果自有的target.property不存在且target不可扩展，则处理程序必须返回undefined表示该属性不存在
	如果自有的target.property不存在，则处理程序不能返回表示该属性可配置的对象
	
*/
```

#### deleteProperty()

```js
//deleteProperty()捕获器会在delete操作符中被调用。对应的反射API方法为Reflect.deleteProperty()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    deleteProperty(target,property){
        console.log('deleteProperty');
        return Reflect.deleteProperty(...arguments)
    }
})
delete proxy.foo;//deleteProperty()

/*
1、返回值：必须返回布尔值，表示删除属性是否成功。返回field布尔值会被转型为布尔值
2、拦截的操作
	delete proxy.property
	delete proxy[property]
	Reflect.deleteProperty(proxy,property)
3、捕获器处理程序参数
	target：目标对象
	property：引用的目标对象上的字符串键属性alue
4、捕获器不变式
	如果自有的target.property存在且不可配置，则处理程序不能删除这个属性
*/
```

#### ownKeys()

```js
//ownKeys()捕获器会在Object.keys()及类似的方法中被调用。对应的反射API方法为Reflect.ownKeys()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    ownKeys(target){
        console.log('ownKeys');
        return Reflect.ownKeys(...arguments)
    }
})
Object.keys(proxy);//ownKeys()

/*
1、返回值：必须返回包含字符串或符号的可枚举对象
2、拦截的操作
	Object.getOwnPropertyNames(proxy)
	Object.getOwnPropertySymbols(proxy)
	Object.keys(proxy)
	Reflect.ownKeys(proxy)
3、捕获器处理程序参数
	target：目标对象
4、捕获器不变式
	返回的可枚举对象必须包含target的所有不可配置的自有属性
	如果target不可扩展，则返回可枚举对象必须准确地包含自有属性键
*/
```

#### getPropertyOf()

```js
//getPropertyOf()捕获器会在Object.getPropertyOf()方法中被调用。
//对应的反射API方法为Reflect.getPropertyOf()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    getPropertyOf(target){
        console.log('getPropertyOf');
        return Reflect.getPropertyOf(...arguments)
    }
})
Object.getPropertyOf(proxy);//getPropertyOf()

/*
1、返回值：必须返回对象或null
2、拦截的操作
	Object.getPropertyOf(proxy)
	Reflect.getPropertyOf(proxy)
	proxy._proto_
	Object.prototype.isPrototypeOf(proxy)
	proxy instanceof Object
3、捕获器处理程序参数
	target：目标对象
4、捕获器不变式
	如果target不可扩展，则Object.getPropertyOf(proxy)唯一有效的返回值就是Object。getPropertyOf(target)的返回的值
*/
```

#### setPropertyOf()

```js
//setPropertyOf()捕获器会在Object.setPropertyOf()中被调用。
//对应的反射API方法为Reflect.setPropertyOf()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    setPropertyOf(target,prototype){
        console.log('setPropertyOf');
        return Reflect.setPropertyOf(...arguments)
    }
})
Object.setPropertyOf(proxy,Object);//setPropertyOf()

/*
1、返回值：必须返回布尔值，表示原型赋值是否成功。返回非布尔值会被转型为布尔值
2、拦截的操作
	Object.setPropertyOf(proxy)
	Reflect.setPropertyOf(proxy)
3、捕获器处理程序参数
	target：目标对象
	prototype:target的替代原型，如果是顶级原型则为null
4、捕获器不变式
	如果target不可扩展，则唯一有效的prototype参数就是Object.getPropertyOf(target)的返回值
*/
```

#### isExtensible()

```js
//isExtensible捕获器咋Object.isExtensible()中被调用。
//对应的反射API方法为：Reflect.isExtensible()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    isExtensible(target){
        console.log('isExtensible');
        return Reflect.isExtensible(...arguments)
    }
})
Object.isExtensible(proxy);//isExtensible()

/*
1、返回值：必须返回布尔值，表示target是否可扩展。返回非布尔值会被转型为布尔值
2、拦截的操作
	Object.isExtensible(proxy)
	Reflect.isExtensible(proxy)
3、捕获器处理程序参数
	target：目标对象
4、捕获器不变式
	如果target可扩展，则处理程序必须返回true
	如果target不可扩展，则护理程序必须返回false
*/
```

#### preventExtensions()

```js
//preventExtensions()捕获器在Object.preventExtensions()中被调用。
//对应的反射API为Reflect.preventExtensions()

const MyTarget = {};
const proxy = new Proxy(myTarget,{
    preventExtensions(target){
        console.log('preventExtensions');
        return Reflect.preventExtensions(...arguments)
    }
})
Object.preventExtensions(proxy);//preventExtensions()

/*
1、返回值：必须返回布尔值，表示target是否已经不可扩展。返回非布尔值会被转型为布尔值
2、拦截的操作
	Object.preventExtensions(proxy)
	Reflect.preventExtensions(proxy)
3、捕获器处理程序参数
	target：目标对象
4、捕获器不变式
	如果Object.preventExtensions(proxy)是false，则处理程序必须返回true
*/
```

#### apply()

```js
//apply()捕获器会在被调用时被调用。对应的反射API方法为Reflect.apply()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    apply(target,thisArg,...argumentsList){
        console.log('apply');
        return Reflect.apply(...arguments)
    }
})
proxy();//apply()

/*
1、返回值：无限制
2、拦截的操作
	proxy(...argumentsList)
	Function.prototype.apply(thisArg,argumentsList)
	Function.prototype.call(thisArg,...argumentsList)
	Reflect.apply(target,thisArg,argumentsList)
3、捕获器处理程序参数
	target：目标对象
	thisArg：调用函数时的this参数
	argumentsList：调用函数时的参数列表
	
4、捕获器不变式
	target必须是一个函数对象
*/
```

#### construct()

```js
//construct()捕获器会在new操作符中被调用。对应的反射API方法为Reflect.construct()
const MyTarget = {};
const proxy = new Proxy(myTarget,{
    construct(target,argumentsList,newTarget){
        console.log('construct');
        return Reflect.construct(...arguments)
    }
})
new proxy;//construct()

/*
1、返回值：必须是一个对象
2、拦截的操作
	new proxy(...argumentsList)
	Reflect.construct(target,argumentsList，newTarget)
3、捕获器处理程序参数
	target：目标对象
	argumentsList：传给目标构造函数的参数列表
	newTarget：最初被调用的构造函数
	
4、捕获器不变式
	target必须可以用作构造函数
*/
```

### 9.3代理模式

#### 跟踪属性访问

通过捕获get、set和has等操作，可以知道对象属性什么时候被访问、被查询。把实现响应捕获器的某个对象代理放到应用中，可以监控这个对象何时在何处被访问过。

```js
const user = {
    name:'Jake'
}
const proxy = new Proxy(user,{
    get(target,property,receiver){
        console.log('getting');
        return Reflect.get(...arguments);
    }
    set(target,property,value,receiver){
        console.log('setting');
        return Reflect.set(...arguments);
    }
})
proxy.name;//gettting
proxy.age = 27;//setting
```

#### 隐藏属性

代理的内部实现对外部代码是不可见的，因此要隐藏目标对象上的属性也是轻而易举

```js
const hiddenProperties = ['foo','bar'];
const targetObject = {
    foo:1,
    bar:2,
    baz:3
};
const proxy = new Proxy(targetObject,{
    get(target,property){
        if(hiddenProperties.includes(property)){
            return undefined
        }else{
            return Reflect.get(...arguments)
        }
    }
    has(target,property){
        if(hiddenProperties.includes(property)){
            return false
        }else{
            return Reflect.get(...arguments)
        }
	}
})
//get()
console.log(proxy.foo);//undefined
console.log(proxy.bar);//undefined
console.log(proxy.baz);//3

//has()
console.log('foo' in proxy);//false
console.log('bar' in proxy);//false
console.log('baz' in proxy);//true
```

#### 属性验证

所有的赋值操作都会触发set()捕获器，所以可以根据所赋的值决定是否允许还是拒绝赋值

```js
const target = {
    onlyNumberGoHere:0
};
const proxy = new Proxy(target,{
    set(target,property,value){
        if(typeof value !=='Number'){
            return false;
        }else{
            return Reflect.set(...arguments);
        }
    }
});

proxy.onlyNumberGoHere = 1;
console.log(proxy.onlyNumberGoHere);//1
proxy.onlyNumberGoHere = '2';
console.log(proxy.onlyNumberGoHere);//1
```

#### 函数与构造函数参数验证

对函数和构造函数参数进行审查

```js
function median(...nums){
    return nums.sort(){Math.floor(nums.length/2)}
}
//所有参数必须是Number类型
const proxy = new Proxy(median,{
    apply(target,thisAarg,...argumentsList){
        for(const arg of argumentsList){
            if(typeof arg !== 'number'){
                throw 'Non-number argument provide'
            }
        }
        return Reflect.apply(...arguments)
    }
})

const.log(proxy(4,7,1));//4
const.log(proxy(4,'7',1));//Error:Non-number argument provide
```

可以要求实例化时必须传给构造函数传参

```js
class User{
    constructor(id){
        this._id = id;
    }
}
const proxy = new Proxy(User,{
    construct(target,argumentsList,newTarget){
        if(argumentsList[0] === undefined){
            throw 'User cannot be instantiated without id'
        }else{
            return Reflect.construct(...arguments)
        }
    }
})

new proxy(1);
new proxy();//Error:User cannot be instantiated without id
```

#### 数据绑定与可观察对象

通过代理可以把运行时原本不相关的部门联系到一起。从而让不同的代码互操作。

```js
//可以将被代理的类绑定到一个全局实例集合，让所有创建的实例都被添加到这个集合中
const userList = [];
class User{
    constructor(id){
        this._id = id;
    }
}

const proxy = new Proxy(User,{
    construct(){
        const newUser = Reflect.construct(...arguments);
        userList.push(newUser);
        return newUser;
    }
})
new proxy('john')
new proxy('jacob')
new proxy('jinglehe')
console.log(userList);//[User{},User{},User{}]

//可以把集合绑定到一个事件分派程序，每次插入新实例时都会发送消息
const userList = [];
function emit(newValue){
    console.log(newValue)
}
const proxy = new Proxy(emit,{
    set(target,property,value,receiver){
        const result = Reflect.set(...arguments);
        if(result){
            emit(Reflect.get(target,property,receiver))
        }
        return result;
    }
})

proxy.push('john')//john
proxy.push('jacb')//jacb
```

