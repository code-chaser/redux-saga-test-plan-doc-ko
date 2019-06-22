# Redux Saga Test Plan

[![npm](https://img.shields.io/npm/v/redux-saga-test-plan.svg?style=flat-square)](https://www.npmjs.com/package/redux-saga-test-plan)
[![Travis branch](https://img.shields.io/travis/jfairbank/redux-saga-test-plan/master.svg?style=flat-square)](https://travis-ci.org/jfairbank/redux-saga-test-plan)
[![Codecov](https://img.shields.io/codecov/c/github/jfairbank/redux-saga-test-plan.svg?style=flat-square)](https://codecov.io/gh/jfairbank/redux-saga-test-plan)

#### 쉽게 Redux Saga를 테스트하세요.

Redux Saga Test Plan은 saga 테스트를 쉽게 만듭니다. effect들의 검증과 순서를 테스트 해야 하거나
특정 시점에 saga `put`의 특정 액션을 테스트 해야 할때 Redux Saga Test Plan를 적용하세요.
<!-- Redux Saga Test Plan makes testing sagas a breeze. Whether you need to test
exact effects and their ordering or just test your saga `put`'s a specific
action at some point, Redux Saga Test Plan has you covered. -->

Redux Saga Test Plan는 saga를 쉽게 테스트 할 수 있게 단위 테스트와 통합 테스트 방법에 대해 모두 
포용하는 것을 목표로 합니다.
<!-- Redux Saga Test Plan aims to embrace both unit testing and integration testing
approaches to make testing your sagas easy. -->

## 통합 테스트 하기

**글로벌 `Promise`가 필요합니다.**

saga 단위 테스트의 단점은 테스트가 구현코드와 결합된다는 것입니다. 단순히 saga에서 반환되는 effect들의
순서를 변경하는것 만으로도 테스트가 깨집니다. 테스트가 기능적으로 동일하더라도 그렇습니다. 
saga에서 반환되는 effect의 검증이나 순서에 관심이 없다면, Redux Saga가 실행될 때 saga의 동작을 
통합적으로 접근해서 테스트할 수 있습니다. 그럴려면 `expectSaga` 테스트 함수를 사용해서 saga가 실행되는 
동안 반환된 일부 effect를 간단하게 테스트 하면 됩니다. 
<!-- One downside to unit testing sagas is that it couples your test to your
implementation. Simple reordering of yielded effects in your saga could break
your tests even if the functionality stays the same. If you're not concerned
with the order or exact effects your saga yields, then you can take a
integrative approach, testing the behavior of your saga when run by Redux Saga.
Then, you can simply test that a particular effect was yielded during the saga
run. For this, use the `expectSaga` test function. -->

### 간단한 예제

`expectSaga` 함수를 import 하고 saga 함수를 첫번째 인자로 전달하세요. 그리고 saga 함수의 인자들을 
`expectSaga` 함수의 추가 인자로 전달하세요. `expectSaga` 함수의 반환값은 Redux Saga에서 사용가능한
다양한 effect 생성자들을 검증할 수 있는 체인화된 API 입니다.
<!-- Import the `expectSaga` function and pass in your saga function as an argument.
Any additional arguments to `expectSaga` will become arguments to the saga
function. The return value is a chainable API with assertions for the different
effect creators available in Redux Saga. -->

다음 예제는, 성공시에 `fakeUser` 데이터를 가지는 `RECEIVE_USER` 액션을 `put`하는
`userSaga`를 테스트합니다. `expectSaga`를 호출하고 `userSaga`와 `api` 객체를 인자로 제공합니다.
`put` 검증 메서드를 통해 `put` effect를 검증합니다. 그리고, 사용자 id 데이터를
포함하는 `REQUSET_USER` 액션을 `dispatch` 메서드와 함께 호출합니다. `dispatch` 메서드는 `take`
effect에게 액션을 제공합니다. 마지막으로 `Promise`를 반환하는 `run` 메서드를 호출하여 test를 시작합니다.
`expectSaga`로 하는 테스트는 항상 비동기적으로 실행되므로 saga가 종료되거나 `expextSaga`가 타임아웃 될때
반환된 `Promise`가 실행됩니다. Jest 같은 테스트 러너를 사용한다면, Jest 내부로 `Promise`를 반환하여
Jest가 테스트의 종료를 알게 할 수 있습니다.
<!-- In the example below, we test that the `userSaga` successfully `put`s a
`RECEIVE_USER` action with the `fakeUser` as the payload. We call `expectSaga`
with the `userSaga` and supply an `api` object as an argument to `userSaga`. We
assert the expected `put` effect via the `put` assertion method. Then, we call
the `dispatch` method with a `REQUEST_USER` action that contains the user id
payload. The `dispatch` method will supply actions to `take` effects. Finally,
we start the test by calling the `run` method which returns a `Promise`. Tests
with `expectSaga` will always run asynchronously, so the returned `Promise`
resolves when the saga finishes or when `expectSaga` forces a timeout. If you're
using a test runner like Jest, you can return the `Promise` inside your Jest
test so Jest knows when the test is complete. -->

```js
import { call, put, take } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

function* userSaga(api) {
  const action = yield take('REQUEST_USER');
  const user = yield call(api.fetchUser, action.payload);

  yield put({ type: 'RECEIVE_USER', payload: user });
}

it('just works!', () => {
  const api = {
    fetchUser: id => ({ id, name: 'Tucker' }),
  };

  return expectSaga(userSaga, api)
    // 최종적으로 `put`이 실행되는지를 검증합니다.
    .put({
      type: 'RECEIVE_USER',
      payload: { id: 42, name: 'Tucker' },
    })

    // saga가 `take`할 액션을 발행하세요.
    .dispatch({ type: 'REQUEST_USER', payload: 42 })

    // 테스트를 시작되고 Promise가 반환됩니다.
    .run();
});
```

### Providers 로 모의(Mock)하기

`expectSaga`는 Redux Saga로 saga를 실행하기 때문에 어플리케이션에서의 Redux Saga처럼 effect들을 
실행하려 합니다. 통합 테스트에서는 이것이 장점이지만, 가끔 빠르게 전체 앱을 테스트 하기가 어려워지기도 합니다.
이런 경우 `expectSaga`의 값들을 완벽하게 모의하는 _providers_ 를 사용 할 수 있습니다. providers는 
미들웨어와 비슷하게 effect들이 Redux Saga에 접근하기 전에 가로챕니다. Redux Saga에서 effect를
처리하는 대신 모의 값을 반환하도록 할 수도 있고, 다른 전에 providers나 Redux Saga에 effect를 전달
할 수도 있습니다.
<!-- `expectSaga` runs your saga with Redux Saga, so it will try to resolve effects
just like Redux Saga would in your application. This is great for integration
testing, but sometimes it can be laborious to bootstrap your entire application
for tests or mock things like server APIs. In those cases, you can use
_providers_ which are perfect for mocking values directly with `expectSaga`.
Providers are similar to middleware that allow you to intercept effects before
they reach Redux Saga. You can choose to return a mock value instead of allowing
Redux Saga to handle the effect, or you can pass on the effect to other
providers or eventually Redux Saga. -->

`expectSaga`의 providers는 _static providers_와 _dynamic providers_, 2가지로 나뉩니다.
정적 providers는 쉽게 조합과 재사용이 되고, 동적 providers는 비결정적인 effect들로 유연함을 줍니다.
다음 예제는 정적 providers를 사용하였습니다. providers의 더 많은 예제는 
[여기](http://redux-saga-test-plan.jeremyfairbank.com/integration-testing/mocking/)
에 있습니다.
<!-- `expectSaga` has two flavors of providers, _static providers_ and _dynamic
providers_. Static providers are easier to compose and reuse, but dynamic
providers give you more flexibility with non-deterministic effects. Here is one
example below using static providers. There are more examples of providers [in
the
docs](http://redux-saga-test-plan.jeremyfairbank.com/integration-testing/mocking/). -->

```js
import { call, put, take } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';
import * as matchers from 'redux-saga-test-plan/matchers';
import { throwError } from 'redux-saga-test-plan/providers';
import api from 'my-api';

function* userSaga() {
  try {
    const action = yield take('REQUEST_USER');
    const user = yield call(api.fetchUser, action.payload);
    const pet = yield call(api.fetchPet, user.petId);

    yield put({
      type: 'RECEIVE_USER',
      payload: { user, pet },
    });
  } catch (e) {
    yield put({ type: 'FAIL_USER', error: e });
  }
}

it('fetches the user', () => {
  const fakeUser = { name: 'Jeremy', petId: 20 };
  const fakeDog = { name: 'Tucker' };

  return expectSaga(userSaga, api)
    .provide([
      [call(api.fetchUser, 42), fakeUser],
      [matchers.call.fn(api.fetchPet), fakeDog],
    ])
    .put({
      type: 'RECEIVE_USER',
      payload: { user: fakeUser, pet: fakeDog },
    })
    .dispatch({ type: 'REQUEST_USER', payload: 42 })
    .run();
});

it('handles errors', () => {
  const error = new Error('error');

  return expectSaga(userSaga, api)
    .provide([
      [matchers.call.fn(api.fetchUser), throwError(error)],
    ])
    .put({ type: 'FAIL_USER', error })
    .dispatch({ type: 'REQUEST_USER', payload: 42 })
    .run();
});
```
matcher와 가짜 값을 포함하는 튜플의 쌍(혹은 배열의 쌍)의 배열을 전달하는 것을 확인하세요. Redux Saga의
effect 생성자 혹은 effect에 대응하는 `redux-saga-test-plan/matchers` 모듈의 matchers를 사용할
수 있습니다. Redux Saga Test Plan의 matchers를 사용하면 `call.fn` 같은 martchers의 경우 이 함수와
대응되는 실제 `call` effect가 포함하는 특정 `args` 에 대헤 신경쓰지 않고 사용할 수 있습니다. 두번째
테스트는 `redux-saga-test-plan/providers` 모듈의 `throwError` 함수로 에러들을 시뮬레이트합니다. 
이것은 서버 문제를 시뮬레이션하기 좋습니다.
<!-- Notice we pass in an array of tuple pairs (or array pairs) that contain a
matcher and a fake value. You can use the effect creators from Redux Saga or
matchers from the `redux-saga-test-plan/matchers` module to match effects. The
bonus of using Redux Saga Test Plan's matchers is that they offer special
partial matchers like `call.fn` which matches by the function without worrying
about the specific `args` contained in the actual `call` effect. Notice in the
second test that we can also simulate errors with the `throwError` function from
the `redux-saga-test-plan/providers` module. This is perfect for simulating
server problems. -->

### Reducer 테스트하기

리듀서 테스트 역시 통합 테스트의 좋은 예입니다. 리듀서 함수를 `withReducer` 메서드와 함께 호출하는 것만으로
리듀서를 테스트해볼 수 있습니다.
<!-- One good use case for integration testing is testing your reducer too. You can
hook up your reducer to your test by calling the `withReducer` method with your
reducer function. -->

```js
import { call, select, take } from 'redux-saga/effects';
import { expectSaga } from 'redux-saga-test-plan';

const HAVE_BIRTHDAY = 'HAVE_BIRTHDAY';
const UPDATE_DOG = 'UPDATE_DOG';

const initialState = {
  dog: {
    name: 'Tucker',
    age: 11,
  },
};

function reducer(state = initialState, action) {
  if (action.type === HAVE_BIRTHDAY) {
    return {
      ...state,
      dog: {
        ...state.dog,
        age: state.dog.age + 1,
      },
    };
  }

  return state;
}

const getDog = state => state.dog;

function* saga(api) {
  yield take(UPDATE_DOG);
  const dog = yield select(getDog);
  yield call(api.updateDog, dog);
}

it('handles reducers', () => {
  const api = { updateDog() {} };

  return expectSaga(saga, api)
    .withReducer(reducer)

    .call(api.updateDog, {
      name: 'Tucker',
      age: 12,
    })

    .dispatch({ type: HAVE_BIRTHDAY })
    .dispatch({ type: UPDATE_DOG })

    .run();
});
```

## 단위 테스트하기

만약 saga가 반환하는 effect들이 특정한 순서이기를 검증하고 싶다면 `testSaga` 함수를 사용하세요. 
간단한 예제입니다:
<!-- If you want to ensure that your saga yields specific types of effects in a
particular order, then you'll want to use the `testSaga` function. Here's a
simple example: -->

```js
import { testSaga } from 'redux-saga-test-plan';

function identity(value) {
  return value;
}

function* mainSaga(x, y) {
  const action = yield take('HELLO');

  yield put({ type: 'ADD', payload: x + y });
  yield call(identity, action);
}

const action = { type: 'TEST' };

it('works with unit tests', () => {
  testSaga(mainSaga, 40, 2);
    // `next()`로 saga 진행
    .next()

    // saga가 `'HELLO'` 유형을 `take` 하는지 검증한다
    .take('HELLO')

    // 반환된 값을 전달한다
    .next(action)

    // saga가 예상되는 액션을 `put`하는지 검증한다
    .put({ type: 'ADD', payload: 42 })

    .next()

    // saga가 `identity`를 `action` 인자로 `call`하는지를 검증한다
    .call(identity, action)

    .next()

    // saga가 종료된 것을 검증한다
    .isDone();
});
```

## Table of Contents

- [소개](README.md)
- [시작하기](docs/getting-started.md)
- [통합테스트하기](docs/integration-testing/README.md)
  - [Effect Creator Assertions](docs/integration-testing/effect-creators.md)
  - [Dispatching](docs/integration-testing/dispatching.md)
  - [Timeout](docs/integration-testing/timeout.md)
  - [State](docs/integration-testing/state.md)
  - [Mocking](docs/integration-testing/mocking/README.md)
    - [Static Providers](docs/integration-testing/mocking/static-providers.md)
    - [Dynamic Providers](docs/integration-testing/mocking/dynamic-providers.md)
  - [Partial Assertions](docs/integration-testing/partial-matching.md)
  - [Negated Assertions](docs/integration-testing/negated-assertions.md)
  - [Exposed Effects](docs/integration-testing/exposed-effects.md)
  - [Snapshot Testing](docs/integration-testing/snapshot-testing.md)
  - [Return Value](docs/integration-testing/return-value.md)
  - [Forked Sagas](docs/integration-testing/forked-sagas.md)
- [Unit Testing](docs/unit-testing/README.md)
  - [Error Messages](docs/unit-testing/error-messages.md)
  - [Effect Creators](docs/unit-testing/effect-creators.md)
  - [Saga Helpers](docs/unit-testing/saga-helpers.md)
  - [General Assertions](docs/unit-testing/general-assertions.md)
  - [Time Travel](docs/unit-testing/time-travel.md)
