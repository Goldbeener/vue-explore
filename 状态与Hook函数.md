**状态** 是特殊的javascript变量，它的变化会引起视图的变化 

如果一个变量已经被框架注册为状态，那么这个变量的变化就会引起视图的更新，我们称之为响应式变量 
如果一个函数包含了状态（响应式变量），那么它就是一个Hook函数 

Hook函数与普通函数的本质区别在于是否具备 **状态** ，也就是是否会引起视图的更新 

Hook函数是同步函数，直接返回值 返回的值是响应式数据，以及对数据的操作方法 

Hook函数注意点 
1. 不用async包装，异步操作用then或者在内部声明async函数 
2. 要考虑是否是 **单例模式** ，也就是每次 **调用hook函数都重新初始化函数内部的状态**