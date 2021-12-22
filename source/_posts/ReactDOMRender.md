# ReactDOM.render

> 对于render方法的定义无非就是将我们的创建好的react元素转换成我们最终想要的DOM并进行渲染。

```
  render(
    element: React$Element<any>, // 虚拟dom对象
    container: DOMContainer, // 目标容器
    callback: ?Function,
  ) {
    return legacyRenderSubtreeIntoContainer(
      null,
      element,
      container,
      false,
      callback,
    );
  },
```

> 进入到legacyRenderSubtreeIntoContainer看一下


```
    function legacyRenderSubtreeIntoContainer(
    parentComponent: ?React$Component<any, any>, // 父组件，之前调用时候已经传了null
    children: ReactNodeList, // 可以理解为根组件
    container: DOMContainer, // 目标容器
    forceHydrate: boolean, // 区分客户端和服务端渲染
    callback: ?Function, // 渲染后的回调函数
    ) {

    ...

    // 第一次执行的时候container上是没有_reactRootContainer属性的，所以该处root为undeifined,所以后续会进到!root判断内。
    let root: Root = (container._reactRootContainer: any);
    if (!root) {
        // Initial mount
        // 创建ReactRoot，可以看下文
        root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
        container,
        forceHydrate,
        );
        if (typeof callback === 'function') {
        const originalCallback = callback;
        callback = function() {
            const instance = DOMRenderer.getPublicRootInstance(root._internalRoot);
            originalCallback.call(instance);
        };
        }
        // Initial mount should not be batched.
        // 初次渲染，需要尽快完成
        DOMRenderer.unbatchedUpdates(() => {
        if (parentComponent != null) {
            // parentComponent是写死的null,所以不会走这里
        } else {
            // 可参考下面ReactRoot部分
            root.render(children, callback);
        }
        });
    } else {
        ...
    }
    return DOMRenderer.getPublicRootInstance(root._internalRoot);
    }
```

> 接上文代码注释：legacyCreateRootFromDOMContainer创建ReactRoot部分

```
    function legacyCreateRootFromDOMContainer(
    container: DOMContainer,
    forceHydrate: boolean,
    ): Root {
    const shouldHydrate =
        forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
    // 第一次的话我们需要对容器进行一个子节点的清理，而这边我们不是服务端渲染，所以进到!shouldHydrate的判断.
    if (!shouldHydrate) {
        let warned = false;
        let rootSibling;
        while ((rootSibling = container.lastChild)) {
        ...
        // 清理掉当前容器的子元素，React认为这些节点是不需要复用的
        container.removeChild(rootSibling);
        }
    }
    ...
    // Legacy roots are not async by default.
    const isConcurrent = false;
    // 返回一个ReactRoot实例，构造函数如下
    return new ReactRoot(container, isConcurrent, shouldHydrate);
    }
```


> ReactRoot的构造函数
> 
```
function ReactRoot(
  container: Container,
  isConcurrent: boolean,
  hydrate: boolean,
) {
  // 调用DOMRenderer.createContainer创建FiberRoot，在后期调度更新的过程中这个节点非常重要
  const root = DOMRenderer.createContainer(container, isConcurrent, hydrate);
  this._internalRoot = root;
}

ReactRoot.prototype.render = function (
  children: ReactNodeList,
  callback: ?() => mixed,
): Work {
  const root = this._internalRoot;
  const work = new ReactWork();
  callback = callback === undefined ? null : callback;
  if (__DEV__) {
    warnOnInvalidCallback(callback, 'render');
  }
  if (callback !== null) {
    work.then(callback);
  }
  // DOMRenderer整体是位于react-reconciler/src/ReactFiberReconciler目录下的，可参考下面部分
  DOMRenderer.updateContainer(children, root, null, work._onCommit);
  return work;
};
```

> updateContainer部分
```
export function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function,
): ExpirationTime {
  const current = container.current;
  const currentTime = requestCurrentTime();
  // 计算更新超时间隔时间
  const expirationTime = computeExpirationForFiber(currentTime, current);
  return updateContainerAtExpirationTime(
    element,
    container,
    parentComponent,
    expirationTime,
    callback,
  );
}

function scheduleRootUpdate(
  current: Fiber,
  element: ReactNodeList,
  expirationTime: ExpirationTime,
  callback: ?Function,
) {
  if (__DEV__) {
    if (
      ReactCurrentFiber.phase === 'render' &&
      ReactCurrentFiber.current !== null &&
      !didWarnAboutNestedUpdates
    ) {
      didWarnAboutNestedUpdates = true;
      warningWithoutStack(
        false,
        'Render methods should be a pure function of props and state; ' +
          'triggering nested component updates from render is not allowed. ' +
          'If necessary, trigger nested updates in componentDidUpdate.\n\n' +
          'Check the render method of %s.',
        getComponentName(ReactCurrentFiber.current.type) || 'Unknown',
      );
    }
  }

  const update = createUpdate(expirationTime);
  // Caution: React DevTools currently depends on this property
  // being called "element".
  update.payload = {element};

  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    warningWithoutStack(
      typeof callback === 'function',
      'render(...): Expected the last optional `callback` argument to be a ' +
        'function. Instead received: %s.',
      callback,
    );
    update.callback = callback;
  }
  enqueueUpdate(current, update);

  scheduleWork(current, expirationTime);
  return expirationTime;
}

```