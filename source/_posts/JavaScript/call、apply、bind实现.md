# call模拟实现
> 大致思路还是依托this在对象中作为方法属性的特性。
```
Function.prototype.myCall = function(context) {
  const ctx = context || window;
  ctx.fn = this;
  const args = [];
  for (let i = 1; i <= arguments.length; i++) {
    args.push("arguments[" + i + "]");
  }
  const res = eval("context.fn(" + args + ")");
  delete ctx.fn;
  return res;
  };
}
```

# apply模拟实现
> 跟call差不多，需要注意下二者的参数区别。
```
Function.prototype.myApply = function(context, arr) {
  const ctx = context || window;
  ctx.fn = this;
  const args = [];
  let res;
  if (!arr) {
    res = ctx.fn();
  } else {
    for (let i = 0; i < arr.length; i++) {
      args.push("arr[" + i + "]");
    }
    res = eval("context.fn(" + args + ")");
  }
  delete ctx.fn;
  return res;
};
```

# bind模拟实现
> 与apply和call不同的是，该方法并不会执行被绑定函数，而是返回一个改变this指向后的函数。
```
Function.prototype.myBind = function(context) {
  // 调用myBind的函数
  const fn = this;
  // myBind接收的剩余参数
  const args = [...arguments].slice(1);
  function resFn() {
    const otherArgs = [...arguments];
    // 如果是通过 new 调用的，绑定 this 为实例对象
    if (this instanceof resFn) {
      fn.apply(this, otherArgs.concat(args));
    } else {
      fn.apply(context, otherArgs.concat(args));
    }
  }
  resFn.prototype = Object.create(fn.prototype);
  return resFn;
};

```




