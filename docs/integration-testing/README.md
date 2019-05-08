# 통합 테스트

**`Promise`가 사용 가능 해야합니다.**

Redux Saga Test Plan은 통합 테스트를 하기 위해 saga effect들이 반환하는 값을 검증을 위한 API를
반환하는 `expectSaga`함수를 제공합니다. `expectSaga`를 사용하기 위해서는 첫 번째 인자에 generator 함수를
넣습니다. 그 generator 함수에 사용될 인자의 값들을 추가적인 인자로 넣습니다.

`expectSaga` runs your saga with Redux Saga's `runSaga` function, so it will run
just like it would in your application. This also means your saga will likely
run asynchronously, so `expectSaga` will also be asynchronous.

After calling `expectSaga` on your saga and making some assertions, you can
start the saga with the `run` method. The `run` method will return a `Promise`,
that you can then use with your favorite testing framework. If any assertions
fail, then `expectSaga` will reject the returned `Promise`. If all assertions
pass, then the `Promise` will resolve.

Look at the example below that uses [Jest](https://facebook.github.io/jest/) as
the testing framework. Notice that we return the `Promise` so Jest knows when
the test completes. Also notice that we don't even have to bother testing the
`call` effect with `expectSaga`.

```js
import { call, put } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

function* mainSaga(x, y) {
  yield call([console.log, console], 'hello');
  yield put({ type: 'ADD', payload: x + y });
}

it('puts an ADD action', () => {
  return expectSaga(mainSaga, 40, 2)
    // assert that the saga will eventually yield `put`
    // with the expected action
    .put({ type: 'ADD', payload: 42 })

    // run it
    .run();
});
```
<!--
# Integration Testing

**Requires global `Promise` to be available**

For integration testing, Redux Saga Test Plan exports an `expectSaga` function
that returns an API for asserting that a saga yields certain effects. To use
`expectSaga`, pass in your generator function as the first argument. Pass in
additional arguments which will be the arguments passed on to the generator
function.

`expectSaga` runs your saga with Redux Saga's `runSaga` function, so it will run
just like it would in your application. This also means your saga will likely
run asynchronously, so `expectSaga` will also be asynchronous.

After calling `expectSaga` on your saga and making some assertions, you can
start the saga with the `run` method. The `run` method will return a `Promise`,
that you can then use with your favorite testing framework. If any assertions
fail, then `expectSaga` will reject the returned `Promise`. If all assertions
pass, then the `Promise` will resolve.

Look at the example below that uses [Jest](https://facebook.github.io/jest/) as
the testing framework. Notice that we return the `Promise` so Jest knows when
the test completes. Also notice that we don't even have to bother testing the
`call` effect with `expectSaga`.

```js
import { call, put } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

function* mainSaga(x, y) {
  yield call([console.log, console], 'hello');
  yield put({ type: 'ADD', payload: x + y });
}

it('puts an ADD action', () => {
  return expectSaga(mainSaga, 40, 2)
    // assert that the saga will eventually yield `put`
    // with the expected action
    .put({ type: 'ADD', payload: 42 })

    // run it
    .run();
});
```
-->
