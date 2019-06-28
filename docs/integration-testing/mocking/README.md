# 프로바이더로 모의하기

가끔 서버 API에 대한 `call`을 모의하거나 가짜 상태와 `select`를 사용하는 셀렉터를 만들어야 할때 
saga의 통합 테스트가 힘들수 있다.
<!-- Sometimes integration testing sagas can be laborious, especially when you have
to mock server APIs for `call` or create fake state and selectors to use with
`select`. -->

테스트를 간단하게 만들기 위해서, Redux Saga Test Plan은 Redux Saga가 처리하도록 하지 않고 
이팩트 생성가자 가로채거나 처리하도록 합니다. 이것은 Redux Saga Test Plan이 프로바이더를 호출하는 
미들웨어 계층과 유사합니다.
<!-- To make tests simpler, Redux Saga Test Plan allows you to intercept and handle
effect creators instead of letting Redux Saga handle them. This is similar to a
middleware layer that Redux Saga Test Plan calls _providers_. -->

프로바이더를 사용하기 위해서 `provide` 메서드를 사용합니다. `provide` 메서드는 객체 리터럴 또는 배열인 
하나의 인자를 가닙니다.
<!-- To use providers, you can call the `provide` method. The `provide` method takes
one argument which can either be an array or an object literal. -->
