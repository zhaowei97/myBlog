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
> 大致思路还是依托this在对象中作为方法属性的特性。
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

