# 통합 테스트

**글로벌 `Promise`가 필요합니다.**

Redux Saga Test Plan은 통합 테스트를 위해 saga가 반환하는 effect를 검증하기 위한 API를
반환하는 `expectSaga`함수를 제공합니다. `expectSaga`를 사용하기 위해서는 제너레이터 함수를 
첫 번째 인수로 전달합니다. 그리고 그 제너레이터 함수에 전달될 인수들을 추가적으로 전달합니다.
<!-- For integration testing, Redux Saga Test Plan exports an `expectSaga` function
that returns an API for asserting that a saga yields certain effects. To use
`expectSaga`, pass in your generator function as the first argument. Pass in
additional arguments which will be the arguments passed on to the generator
function. -->

`expectSaga`는 Redux Saga의 `runSaga` 함수를 사용하여 saga를 실행하기 때문에 실제 앱에서
동작 하는 것처럼 실행될 것입니다. saga가 비동기적으로 실행되는 것 처럼 `expectSaga` 역시 비동기적으로
실행됩니다.
<!-- `expectSaga` runs your saga with Redux Saga's `runSaga` function, so it will run
just like it would in your application. This also means your saga will likely
run asynchronously, so `expectSaga` will also be asynchronous. -->

`expectSaga`를 호출하고 검증문을 작성한 후에 `run` 메서드로 saga를 시작할 수 있습니다. `run`
메서드는 `Promise`를 반환하고 당신이 좋아하는 테스팅 프레임워크와 함께 사용할 수 있습니다. 검증이
실패한다면 `expectSaga`는 반환된 `Promise`를 리젝트(reject)할 것입니다. 모든 검증이 통과되면,
`Promise`는 리졸브(resolve)될 것입니다.

<!-- After calling `expectSaga` on your saga and making some assertions, you can
start the saga with the `run` method. The `run` method will return a `Promise`,
that you can then use with your favorite testing framework. If any assertions
fail, then `expectSaga` will reject the returned `Promise`. If all assertions
pass, then the `Promise` will resolve. -->

다음 예제는 테스팅 프레임워크로 [Jest](https://facebook.github.io/jest/)를 사용했습니다. 
테스트가 완료될때 Jest가 알 수 있도록 `Promise`를 반환합니다. 그리고 `expectSaga`로 `call`을 
테스트하지도 않았습니다.

<!-- Look at the example below that uses [Jest](https://facebook.github.io/jest/) as
the testing framework. Notice that we return the `Promise` so Jest knows when
the test completes. Also notice that we don't even have to bother testing the
`call` effect with `expectSaga`. -->

```js
import { call, put } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

function* mainSaga(x, y) {
  yield call([console.log, console], 'hello');
  yield put({ type: 'ADD', payload: x + y });
}

it('puts an ADD action', () => {
  return expectSaga(mainSaga, 40, 2)
    // saga가 최종적으로 예상되는 액션을 `put` 하는지 검증합니다.
    .put({ type: 'ADD', payload: 42 })

    // run it
    .run();
});
```
