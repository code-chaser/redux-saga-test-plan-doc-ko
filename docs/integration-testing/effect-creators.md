# 이팩트 생성자 검증

`expectSaga` API는 Redux Saga에 있는 대부분의 이팩트 생성자들을 검증할 수 있습니다. effect
생성자들에 대해서는 Redux Saga의 [문서](http://redux-saga.github.io/redux-saga/docs/api/index.html#effect-creators)를 참조하세요.
<!-- The `expectSaga` API has assertions for most of the effect creators available in
Redux Saga. You can reference effect creators in Redux Saga's docs
[here](http://redux-saga.github.io/redux-saga/docs/api/index.html#effect-creators). -->

- `take(pattern)`
- `take.maybe(pattern)`
- `put(action)`
- `put.resolve(action)`
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
- `select(selector, ...args)`
- `actionChannel(pattern, [buffer])`
- `race(effects)`
- `setContext(props)`
- `getContext(prop)`

`returns` 메서드를 통해 saga의 반환 값을 검증합니다. 이것은 테스트의 가장 상위인 saga에서만 동작합니다. 즉, `call`,
`fork`, 또는 `spwn`을 통해 호출 되는 다른 saga들에 대해서는 동작하지 않습니다.
<!-- You can assert the return value of a saga via the `returns` method. This only
works for the top-level saga under test, meaning other sagas that are invoked
via `call`, `fork`, or `spawn` won't report their return value. -->

```js
function* saga() {
  return { hello: 'world' };
}

it('returns a greeting', () => {
  return expectSaga(saga)
    .returns({ hello: 'world' })
    .run();
});
```
