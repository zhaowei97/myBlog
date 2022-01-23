# new模拟实现
```
function myNew(){
    var obj = new Object();

    constructor = [].shift.call(arguments);

    obj._proto_ = constructor.prototype;

    var ret = constructor.apply(obj, arguments);

    // 如果构造函数有返回值且返回值是个对象，那么直接返回这个对象。
    return typeof ret === 'object' ? ret : obj;
}
```
[写法参考]('https://juejin.cn/post/6844903476766441479')
