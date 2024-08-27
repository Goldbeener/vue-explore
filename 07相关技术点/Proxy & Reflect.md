
# Proxy
为任何目标对象创建一个代理，从而拦截、定义对该对象的基本操作的自定义行为
完全控制对内部对象的访问

# Reflect
内置对象
提供了一组与JS运行时操作对应的方法

也就是使用Reflect提供的方法操作对象
与JS内部运行时操作对象的机制是一样的

也就是说Reflect可以实现类似JS内部函数调用的效果
比如[[Set]] [[Get]]



二者一般配合使用
使用Proxy代理对象、拦截对象行为时，最终还是要符合对象的默认行为；
而使用Reflect可以简单的实现这个

比如访问对象不存在属性，返回undefined
如果不使用Reflect，需要手动处理兜底行为返回undefined
使用`Reflect.get(target, key)`; 就像是JS引擎在访问对象，与默认行为一致


Reflect 一般用在Proxy的trap函数中，用来使得Proxy的拦截，更加简单，并且不影响原函数的设计


# Receiver
receiver 是接受者的意思
表示调用对应属性或方法的主体对象
通常情况下，无需使用，如果发生了继承，为了明确调用主体，就需要使用receiver


**在Proxy trap函数中**

> The receiver can either be the Proxy created or any object that inherits the Proxy.

正常情况下，receiver是proxy对象
如果发生继承，那么receiver是继承proxy的对象


**在Reflect.get 场景下**
receiver可以改变计算属性中this的指向
```js
Reflect.get(target, propertyKey[, receiver])

// 如果`target`对象中指定了`getter`，`receiver`则为`getter`调用时的`this`值

Reflect.set(target, propertyKey, value [, receiver])
// 如果`target`对象中指定了`getter`，`receiver`则为`setter`调用时的`this`值
```

用于调整target对象getter/setter函数内this的指向