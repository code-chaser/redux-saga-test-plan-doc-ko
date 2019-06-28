# Effect Creator Assertions

`expectSaga`는 Redux Saga에 있는 대부분의 이팩트 생성자들에 대해 검증 API를 제공합니다. 이펙트
생성자들에 대해서는 Redux Saga의 [문서](http://redux-saga.github.io/redux-saga/docs/api/index.html#effect-creators)를 참조하세요.
<!-- The `testSaga` API has assertions for all the effect creators available in Redux
Saga. You can reference them in Redux Saga's docs
[here](http://redux-saga.github.io/redux-saga/docs/api/index.html#effect-creators). -->

- `take(pattern)`
- `takeMaybe(pattern)`
- `take(channel)`
- `takeMaybe(channel)`
- `put(action)`
- `putResolve(action)`
- `put(channel, action)`
- `call(fn, ...args)`
- `call([context, fn], ...args)`
- `apply(context, fn, args)`
- `cps(fn, ...args)`
- `cps([context, fn], ...args)`
- `fork(fn, ...args)`
- `fork([context, fn], ...args)`
- `spawn(fn, ...args)`
- `spawn([context, fn], ...args)`
- `join(task)`
- `cancel(task)`
- `select(selector, ...args)`
- `actionChannel(pattern, [buffer])`
- `flush(channel)`
- `cancelled()`
- `all([...effects])`
- `race(effects)`
- `setContext(props)`
- `getContext(prop)`
- `delay(time)`
