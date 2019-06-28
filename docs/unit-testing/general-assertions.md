# 기타 검증 함수

`is`, `isDone`, `returns`, `inspect` 들은 헬퍼 함수와 검증 함수들입니다.
<!-- Other general assertions and helpers available are `is`, `isDone`, `returns`,
and `inspect`. -->

| Assertion  | Description                                                          |
| ---------- | -------------------------------------------------------------------- |
| `is`       | 일반적인 용도의 깊은 동등 검증                                       |
| `isDone`   | 사가의 종료를 검증                                                   |
| `returns`  | 사가가 반환하는 값과 종료 되었음을 검증                              |
| `inspect`  | 더 세밀한 테스트를 위해 next로 반환되는 값을 검사                    |

#### 일반적인 예제

```js
import { take } from 'redux-saga/effects';

function* mainSaga() {
  yield 42;
  yield { foo: { bar: 'baz' } };
}

let saga = testSaga(mainSaga);

saga
  .next()
  .is(42)
  
  .next()
  .is({ foo: { bar: 'baz' } })
  
  .next()
  .isDone();
```

#### `returns` 예제

```js
function* otherSaga(x) {
  return x * 2;
}

const saga = testSaga(otherSaga, 21);

saga
  .next()
  .returns(42);
```

#### `inspect` 예제

saga가 비결정적인 유형의 값을 반환한다면 이팩트 검증 함수나 일반적인 검증함수로
처리하기 어렵습니다. 이럴경우, `inspect` 함수를 사용하여 실제 반환되는 값을 
당신이 사용하는 검증 함수 라이브러리의 검증 함수로 탐색할 수 있습니다.
<!-- If your saga yields a nondeterministic type of value or something not easily
covered by the effect assertions or other general assertions, then you can use
`inspect` to retrieve the actual yielded value and perform your own assertions
with your favorite assertion library. -->

```js
function* saga() {
  yield () => 42;
}

testSaga(saga)
  .next()
  .inspect((fn) => {
    expect(fn()).toBe(42);
  });
```
