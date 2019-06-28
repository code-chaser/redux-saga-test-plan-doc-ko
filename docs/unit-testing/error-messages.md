# 에러 메시지

반환된 이팩트와 검증 이팩트가 일치하지 않으면 saga는 둘의 
차이를 보여주는 에러를 발생시킬 것입니다.
<!-- If a yielded effect and assertion effect call don't match, then the mock saga
will throw an error showing the difference between the two. -->

```js
function identity(value) {
  return value;
}

function* otherSaga() {}

function* mainSaga(x, y, z) {
  try {
    const action = yield take('HELLO');
    yield put({ type: 'ADD', payload: x + y });
    yield call(identity, action);
    yield fork(otherSaga, z);
  } catch (e) {
    yield put({ type: 'ERROR', payload: e });
  }
}

const action = { type: 'TEST' };

let saga = testSaga(mainSaga, 40, 2, 20);

saga.next().take('HI');

// Throws with below:
//
// SagaTestError:
// Assertion 1 failed: take effects do not match
// 
// Expected
// --------
// { channel: null, pattern: 'HI' }
// 
// Actual
// ------
// { channel: null, pattern: 'HELLO' }

saga = testSaga(mainSaga, 40, 2, 20);
saga
  .next()
  .take('HELLO')

  .next(action)
  .put({ type: 'ADD', payload: 43 });

// Throws with below:
//
// SagaTestError:
// Assertion 2 failed: put effects do not match
// 
// Expected
// --------
// { channel: null, action: { type: 'ADD', payload: 43 } }
// 
// Actual
// ------
// { channel: null, action: { type: 'ADD', payload: 42 } }
```

반환된 이팩트와 검증에 사용된 이펙트의 타입이 다르면 그 차이를 보여주는
에러를 발생시킬 것입니다.
<!-- If the yielded effect and asserted effect are different types of effects, then
the saga will throw an error with a message showing the difference. -->

```js
function identity(value) {
  return value;
}

function* otherSaga() {}

function* mainSaga(x, y, z) {
  try {
    const action = yield take('HELLO');
    yield put({ type: 'ADD', payload: x + y });
    yield call(identity, action);
    yield fork(otherSaga, z);
  } catch (e) {
    yield put({ type: 'ERROR', payload: e });
  }
}

const action = { type: 'TEST' };

const saga = testSaga(mainSaga, 40, 2, 20);

saga
  .next()
  .take('HELLO')

  .next(action)
  .take('WORLD');

// SagaTestError:
// Assertion 2 Failed: expected take effect, but the saga yielded a different effect
// 
// Expected
// --------
// { '@@redux-saga/IO': true,
//   TAKE: { channel: null, pattern: 'WORLD' } }
// 
// Actual
// ------
// { '@@redux-saga/IO': true,
//   PUT: { channel: null, action: { type: 'ADD', payload: 42 } } }
```
