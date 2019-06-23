# 액션 디스패치하기

대부분의 saga들은 Redux 액션을 대기하는 `take` 이팩트를 사용합니다. `expectSaga`는
saga를 단순히 실행하기만 하기 때문에 `take` 이팩트가 반환되면 saga 실행이 멈추게됩니다.
saga를 계속 실행하기 위해서 (일반적으로 saga가 그러하듯) `expectSaga` 또한 `dispatch`가
필요합니다. saga를 실행하기 전에 saga가 기대하는 모든 작업을 원하는 순서대로 디스패치하세요.
`expectSaga`는 내부적으로 액션들을 큐에 넣고 필요할때마다 당신을 대신해서 액션들을 디스패치
할것입니다.
<!-- Most sagas use the `take` effect to wait on Redux actions. Because `expectSaga`
runs your saga as normal, it will block on yielded `take` effects. To ensure
your saga can keep running, `expectSaga` also has a `dispatch` method. Before
running your saga, dispatch any actions you expect your saga to take in the
order it takes them. Internally, `expectSaga` will queue the actions and
dispatch them on your behalf as needed. -->

```js
import { put } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

function* mainSaga(x, y) {
  yield take('HELLO');
  yield put({ type: 'ADD', payload: x + y });
  yield take('WORLD');
  yield put({ type: 'DONE' });
}

it('handles dispatching actions', () => {
  return expectSaga(mainSaga, 40, 2)
    // 검증은 순서와 상관 없습니다.
    .put({ type: 'DONE' })
    .put({ type: 'ADD', payload: 42 })

    // saga가 `take`한 순서대로 액션을 디스패치하세요.
    .dispatch({ type: 'HELLO' })
    .dispatch({ type: 'WORLD' })

    .run();
});
```

## 시간이 걸리는 액션 디스패치
saga의 실행에 시간이 걸리는 액션을 디스패치할 수도 있습니다. 이 경우, Redux Saga Test Plan이 너무 빠르게
디스패치 못하도록 액션을 지연하는것이 좋습니다.
<!-- You can also dispatch actions while a saga is running. This is useful for
delaying actions so Redux Saga Test Plan doesn't dispatch them too quickly. -->

```js
function* mainSaga() {
  // 거의 바로 실행됨
  yield take('FOO');

  // 250ms 걸림
  yield take('BAR');
  yield put({ type: 'DONE' });
}

const delay = time => new Promise((resolve) => {
  setTimeout(resolve, time);
});

it('can dispatch actions while running', () => {
  const saga = expectSaga(mainSaga);

  saga.put({ type: 'DONE' });

  saga.dispatch({ type: 'FOO' });

  const promise = saga.run({ timeout: false });

  return delay(250).then(() => {
    saga.dispatch({ type: 'BAR' });
    return promise;
  });
});
```

## `.delay()`로 액션을 지연시켜서 디스패치하기

실행 시간이 걸리는 액션을 디스패치하기 위해 지연시키는 것 말고도 다른 방법이 있습니다. 액션이 
디스패치되기만을 지연시키고 싶다면 `delay` 사용하세요. `delay`의 인자는 지연시킬 시간입니다. 
<!-- While being able to dispatch actions while the saga is running has use cases
besides only delaying, if you just want to delay dispatched actions, you can use
the `delay` method. It takes a delay time as its only argument. -->

```js
function* mainSaga() {
  // 거의 바로 실행됨
  yield take('FOO');

  // 250ms 걸림
  yield take('BAR');
  yield put({ type: 'DONE' });
}

it('can delay actions', () => {
  return expectSaga(mainSaga)
    .put({ type: 'DONE' })
    .dispatch({ type: 'FOO' })
    .delay(250)
    .dispatch({ type: 'BAR' })
    .run({ timeout: false });
});
```
