# Redux Saga Test Plan

[![npm](https://img.shields.io/npm/v/redux-saga-test-plan.svg?style=flat-square)](https://www.npmjs.com/package/redux-saga-test-plan)
[![Travis branch](https://img.shields.io/travis/jfairbank/redux-saga-test-plan/master.svg?style=flat-square)](https://travis-ci.org/jfairbank/redux-saga-test-plan)
[![Codecov](https://img.shields.io/codecov/c/github/jfairbank/redux-saga-test-plan.svg?style=flat-square)](https://codecov.io/gh/jfairbank/redux-saga-test-plan)

#### 쉽게 Redux Saga 를 테스트하세요.

Redux Saga Test Plan은 saga 테스트를 쉽게 만듭니다. effect들의 검증과 순서를 테스트 해야 하거나
특정 시점에 saga `put`의 특정 액션을 테스트 해야 할때 Redux Saga Test Plan를 적용하세요.
<!-- Redux Saga Test Plan makes testing sagas a breeze. Whether you need to test
exact effects and their ordering or just test your saga `put`'s a specific
action at some point, Redux Saga Test Plan has you covered. -->

Redux Saga Test Plan는 saga를 쉽게 테스트 할 수 있게 단위 테스트와 통합 테스트 방법에 대해 모두 
포용하는 것을 목표로 합니다.
<!-- Redux Saga Test Plan aims to embrace both unit testing and integration testing
approaches to make testing your sagas easy. -->

## 통합 테스트

**글로벌 `Promise`가 필요합니다.**

saga 단위 테스트의 단점은 테스트가 구현코드와 결합된다는 것입니다. 단순히 saga에서 반환되는 effect들의
순서를 변경하는것 만으로도 테스트가 깨집니다. 심지어 saga와 테스트가 기능적으로 동일하더라도 그렇습니다. 
saga에서 반환되는 effect의 검증이나 순서에 관심이 없다면, Redux Saga가 실행될 때 saga의 동작을 
통합적으로 접근해서 테스트할 수 있습니다. 그리고 saga가 실행되는 동안 반환된 일부 effect를 간단하게
테스트를할 수도 있습니다.
이럴때 `expectSaga` 테스트 함수를 사용하세요.
<!-- One downside to unit testing sagas is that it couples your test to your
implementation. Simple reordering of yielded effects in your saga could break
your tests even if the functionality stays the same. If you're not concerned
with the order or exact effects your saga yields, then you can take a
integrative approach, testing the behavior of your saga when run by Redux Saga.
Then, you can simply test that a particular effect was yielded during the saga
run. For this, use the `expectSaga` test function. -->

### 간단한 예

`expectSaga` 함수를 import 하고 saga 함수를 인자로 전달하세요. saga 함수의 인자들이 `expectSaga` 
함수의 추가 인자가 될 것입니다. `expectSaga` 함수의 반환값은 Redux Saga에서 사용가능한 다양한 
effect 생성자들에 대한 assertion(단언문)들의 체인화된 API 입니다.
<!-- Import the `expectSaga` function and pass in your saga function as an argument.
Any additional arguments to `expectSaga` will become arguments to the saga
function. The return value is a chainable API with assertions for the different
effect creators available in Redux Saga. -->

다음 예제에서 처럼, 성공시에 `fakeUser` 데이터를 가지는 `RECEIVE_USER` 액션을 `put`하는
`userSaga`를 테스트합니다. `expectSaga`를 호출하고 `userSaga`와 `api` 객체를 인자로 제공합니다.
`put` assertion 메서드를 통해 `put` effect가 예상되는것을 확인합니다. 그리고, 사용자 id 데이터를
포함하는 `REQUSET_USER` 액션과 함께 `dispatch` 메서드를 호출합니다. `dispatch` 메서드는 `take`
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
    // 결국 `put`이 실행될 것을 확인합니다.
    .put({
      type: 'RECEIVE_USER',
      payload: { id: 42, name: 'Tucker' },
    })

    // `userSaga`가 `take`할 액션을 발행하세요.
    .dispatch({ type: 'REQUEST_USER', payload: 42 })

    // 테스트를 시작되고 Promise가 반환됩니다.
    .run();
});
```

### Mocking with Providers

`expectSaga`는 Redux Saga로 saga를 실행하기 때문에 어플리케이션에서의 Redux Saga처럼 effect들을 
실행하려 합니다. This is great for integration
testing, but sometimes it can be laborious to bootstrap your entire application
for tests or mock things like server APIs. In those cases, you can use
_providers_ which are perfect for mocking values directly with `expectSaga`.
Providers are similar to middleware that allow you to intercept effects before
they reach Redux Saga. You can choose to return a mock value instead of allowing
Redux Saga to handle the effect, or you can pass on the effect to other
providers or eventually Redux Saga.
<!-- `expectSaga` runs your saga with Redux Saga, so it will try to resolve effects
just like Redux Saga would in your application. This is great for integration
testing, but sometimes it can be laborious to bootstrap your entire application
for tests or mock things like server APIs. In those cases, you can use
_providers_ which are perfect for mocking values directly with `expectSaga`.
Providers are similar to middleware that allow you to intercept effects before
they reach Redux Saga. You can choose to return a mock value instead of allowing
Redux Saga to handle the effect, or you can pass on the effect to other
providers or eventually Redux Saga. -->

`expectSaga` has two flavors of providers, _static providers_ and _dynamic
providers_. Static providers are easier to compose and reuse, but dynamic
providers give you more flexibility with non-deterministic effects. Here is one
example below using static providers. There are more examples of providers [in
the
docs](http://redux-saga-test-plan.jeremyfairbank.com/integration-testing/mocking/).

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

Notice we pass in an array of tuple pairs (or array pairs) that contain a
matcher and a fake value. You can use the effect creators from Redux Saga or
matchers from the `redux-saga-test-plan/matchers` module to match effects. The
bonus of using Redux Saga Test Plan's matchers is that they offer special
partial matchers like `call.fn` which matches by the function without worrying
about the specific `args` contained in the actual `call` effect. Notice in the
second test that we can also simulate errors with the `throwError` function from
the `redux-saga-test-plan/providers` module. This is perfect for simulating
server problems.

### Example with Reducer

One good use case for integration testing is testing your reducer too. You can
hook up your reducer to your test by calling the `withReducer` method with your
reducer function.

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

## Unit Testing

If you want to ensure that your saga yields specific types of effects in a
particular order, then you'll want to use the `testSaga` function. Here's a
simple example:

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
    // advance saga with `next()`
    .next()

    // assert that the saga yields `take` with `'HELLO'` as type
    .take('HELLO')

    // pass back in a value to a saga after it yields
    .next(action)

    // assert that the saga yields `put` with the expected action
    .put({ type: 'ADD', payload: 42 })

    .next()

    // assert that the saga yields a `call` to `identity` with
    // the `action` argument
    .call(identity, action)

    .next()

    // assert that the saga is finished
    .isDone();
});
```

## Table of Contents

- [Introduction](README.md)
- [Getting Started](docs/getting-started.md)
- [Integration Testing](docs/integration-testing/README.md)
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

