# 시간제한

`expectSaga`는 비동기적으로 saga를 실행하기 때문에, saga가 너무 오래 실행될 경우을 대비해 기본적인
시간제한을 가진다. 무한 루프의 다중 비둥기 액션을 가지는 saga나 `takeEvery` 같은 헬퍼함수를 사용하는
saga에 필요하다. `expectSaga`는 시간이 초과되면 경고메시지를 출력하고 saga를 취소시킬 것이다.
<!-- Because `expectSaga` runs sagas asynchronously, it has a default timeout in case
your saga runs too long. This is needed for sagas that have multiple
asynchronous actions, that have infinite loops, or that use saga helpers like
`takeEvery`. `expectSaga` will cancel your saga if it times out and print a
warning message. -->

```js
function* mainSaga() {
  while (true) {
    const action = yield take('READY');
    yield put({ type: 'DATA', payload: action.payload });
  }
}

it('times out', () => {
  return expectSaga(mainSaga)
    .put({ type: 'DATA', payload: 42 })
    .dispatch({ type: 'READY', payload: 42 })
    
    // saga는 스스로 종료되지 않음
    // 250ms가 지나면 saga가 취소되고 콘솔에 경고 메시지를 출력함
    .run();
});
```

### 경고 없애기

경고 메시지는 무한 루프 없이도 너무 오래 걸리는 saga를 테스트하기에도 유용합니다. 무한 루프를 가지는
saga라도 시간 초과해서 계속하기를 원할 수 있다. 그렇다면 경고 메시지를 없애기 위해 `silentRun`
메서드를 사용하면 된다. 이 함수는 `run` 메서드와 동일하지만 Redux Saga Test Plan의 시간 초과
경고를 무시합니다. **알림:** 테스트 프레임워크의 모든 시간 초과 경고를 무시하지는 않습니다. 
(예를 들면 Jest 같은)
<!-- The warning message is typically useful if a saga without an infinite loop is
taking too long. If you have a saga with an infinite loop, though, you will want
it to time out. Therefore, to silence the warning message, you can call the
`silentRun` method instead. It functions the same as the `run` method but
suppresses timeout warnings from Redux Saga Test Plan. **NOTE:** this will not
suppress any timeout warnings from your test runner (e.g. Jest). -->

```js
it('can be silenced', () => {
  return expectSaga(mainSaga)
    .put({ type: 'DATA', payload: 42 })
    .dispatch({ type: 'READY', payload: 42 })

    // 경고 메시지가 출력되지 않습니다.
    // 시간 초과하는것이 예측될 경우 유용함
    .silentRun();
});
```
`run` 메서드에 옵션으로 `silenceTimeout`를 전달함으로써 시간 초과를 무시하는 예전 방법도 여전히 가능합니다.
하지만 `silentRun` 메서드를 사용하는 것이 더 좋습니다.
<!-- The old method of silencing timeouts by passing a `silenceTimeout` option into
`run` is still available, but you're encouraged to use the cleaner `silentRun`
method. -->

```js
it('can be silenced', () => {
  return expectSaga(mainSaga)
    .put({ type: 'DATA', payload: 42 })
    .dispatch({ type: 'READY', payload: 42 })

    // 여전히 작동하지만 타이핑할게 많다.
    .run({ silenceTimeout: true });
});
```

### 시간제한 적용하기

시간제한 길이를 조정할 수 있습니다. 기본값은 250ms 입니다. `expectSaga`의 `DEFAULT_TIMEOUT`
프로퍼티 값을 ms단위로 설정하여 시간제한을 변경할 수 있습니다.
<!-- Instead of silencing warnings, you can adjust the timeout length. The default
timeout length is 250 milliseconds. You can change the default timeout by
setting the `DEFAULT_TIMEOUT` property of `expectSaga` in milliseconds. -->

```js
expectSaga.DEFAULT_TIMEOUT = 500; // set it to 500ms
```

특별한 경우 프로퍼티의 기본값을 무시하고 싶다면, `run` (혹은 `silentRun`) 메서드를 실행할 때
시간 제한 길이를 인자로 전달하면 됩니다.
<!-- If you want to override the timeout for a particular test case, then you can
pass in a timeout length to the `run` (or `silentRun`) method. -->

```js
const delay = time => new Promise(resolve => setTimeout(resolve, time));

function* mainSaga() {
  yield call(delay, 300);
  yield put({ type: 'HELLO' });
}

it('can have a different timeout length', () => {
  return expectSaga(mainSaga)
    .put({ type: 'HELLO' })

    // saga는 최소 300ms 걸림
    // 시간제한이 500ms이므로 정상적임
    .run(500);
});
```
또는 `run` 메서드에 `false`를 전달하여 saga가 스스로 종료될 때까지 `expectSaga`가 기다리게
할 수 있습니다. **경고:** 무한 루프로 동작하는 saga 일경우 스스로 종료되지 않기 때문에 동작하지
않을 것입니다.

<!-- Alternatively, you can opt out of the timeout behavior and force `expectSaga` to
wait until your saga is done on its own by passing in `false` to the `run`
method. **WARNING:** this won't work with sagas with infinite loops because the
saga will never finish on its own. -->

```js
it('never times out', () => {
  return expectSaga(mainSaga)
    .put({ type: 'HELLO' })

    // saga가 스스로 종료될 때까지 기다림
    .run(false);
});
```
