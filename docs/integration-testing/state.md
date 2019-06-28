# 상태 테스트하기

`withState`, `withReducer`, `hasFinalState` 메서드를 통해 스토어 상태와 리듀서를
함께 통합 테스트할 수 있습니다.
<!-- You can test your saga's integration with your reducer and store state via the
`withState`, `withReducer`, and `hasFinalState` methods. -->

## `withState`를 통해 정적 상태 테스트하기

정적 상태의 경우 `select` 이팩트가 동작하는 `withState` 메서드를 사용거나
[프로바이더](/integration-testing/mocking/README.md)를 사용할 수 있습니다.
<!-- For static state, you can just use the `withState` method to allow `select`
effects to work. You can also use
[providers](/integration-testing/mocking/README.md) for this. -->

```js
const storeState = {
  data: {
    foo: 'bar',
  },
};

function getData(state) {
  return state.data;
}

function* saga() {
  const data = yield select(getData);
  yield put({ type: 'DATA', payload: data });
}

it('can take store state', () => {
  return expectSaga(saga)
    .withState(storeState)
    .put({ type: 'DATA', payload: storeState.data })
    .run();
});
```

## `withReducer`를 통해 동적 상태 테스트하기

변경되는 상태에 대해서는 `withReducer` 메서드를 사용할 수 있습니다. `withReducer` 메서드는
2개의 인자로 리듀서와 상태(생략가능) 초기값을 가집니다. 초기값을 제공하지 않으면 `withReducer`는
리덕스처럼 리듀서의 초기 액션으로 부터 초기값을 얻어내려 할 것입니다.
<!-- For state that might change, you can use the `withReducer` method. It takes two
arguments: your reducer and optional initial state. If you don't supply the
initial state, then `withReducer` will extract it by passing an initial action
into your reducer like Redux. -->

`select` 이팩트는 사용된곳에서 변경된 상태를 테스트할 수 있지만, `hasFinalState`는 saga가 완료된
후에 스토어 상태 값을 테스트 할 수 있습니다.

<!-- Any `select` effects will reflect state changes where appropriate. More
importantly, you can test your store state after the saga completes via
`hasFinalState`. -->

```js
const initialDog = {
  name: 'Tucker',
  age: 11,
};

function dogReducer(state = initialDog, action) {
  if (action.type === 'HAVE_BIRTHDAY') {
    return {
      ...state,
      age: state.age + 1,
    };
  }

  return state;
}

function* saga() {
  yield put({ type: 'HAVE_BIRTHDAY' });
}

it('handles reducers when not supplying initial state', () => {
  return expectSaga(saga)
    .withReducer(dogReducer)
    .hasFinalState({
      name: 'Tucker',
      age: 12, // <-- 스토어에서 변경된 age
    })
    .run();
});

it('handles reducers when supplying initial state', () => {
  return expectSaga(saga)
    .withReducer(dogReducer, initialDog)
    .hasFinalState({
      name: 'Tucker',
      age: 12, // <-- 스토어에서 변경된 age
    })
    .run();
});
```

## 스토어 상태 제공

`run`가 반환하는 `Promise`는 세밀한 테스트를 할수있는 `storeState` 객체를 리졸브한다.
<!-- The `Promise` returned from `run` resolves with a `storeState` object that you
can inspect for more fine-grained testing. -->

```js
it('exposes the store state', () => {
  return expectSaga(saga)
    .withReducer(dogReducer, initialDog)
    .run()
    .then((result) => {
      expect(result.storeState).toEqual({
        name: 'Tucker',
        age: 12, // <-- 스토어에서 변경된 age
      });
    });
});

it('exposes the store state using async/await', async () => {
  const { storeState } = await expectSaga(saga)
    .withReducer(dogReducer, initialDog)
    .run();

  expect(storeState).toEqual({
    name: 'Tucker',
    age: 12, // <-- 스토어에서 변경된 age
  });
});
```
