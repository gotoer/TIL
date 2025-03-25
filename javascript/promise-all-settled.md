# Promise.all과 promise.allSettled

## Promise

JavaScript에서 Promise는 비동기적으로 실행하는 작업의 결과를 나타내는 객체입니다.  
비동기의 성공 혹은 실패 결과를 객체화 시킵니다.

<br>

다른 생성자들처럼 new Promise()로 객체를 만들면 되고, 이 때 인자는 resolve, reject 두 함수를 매개변수로 받는 실행함수 executor입니다.  
Executor는 비동기 작업이 성공적으로 끝나면 resolve 함수를, 작업 도중 오류가 발생하면 reject 함수를 호출합니다.  

<br>

Promise 객체는 대기(pending) - 작업 수행 전, 이행(fulfilled) - 작업 성공, 거부(rejected) - 작업 실패 세 가지 상태를 가집니다.  
이 때 작업이 이행되거나 거부되었을 때 이어지는 작업을 then 메서드를 통해 진행할 수 있습니다.  

<br>

Then 메서드는 promise객체를 리턴하고 성공 콜백 함수, 실패 콜백 함수 두 개의 함수를 인수로 받습니다.  
아래 예시처럼 chaining 할 수도 있습니다.

```typescript
var promise = new Promise(function (resolve, reject) {
  setTimeout(function () {
    resolve(1);
  }, 1000);
});

promise.then(function (num) {
  console.log(num + 'complete'); /// 1complete
  return num + 1; /// return = 2
}).then(function (value) {
  console.log(value) // 2
});
```

<br>
<br>
<br>

## Promise.all

비동기적으로 작업을 처리하는 promise가 여러 개인 경우에도 여러 promise를 비동기로 처리할 수 있습니다.  
Promise.all() 메서드는 인자로 promise 배열을 받고 비동기로 처리합니다.  

<br>
<br>

```typescript
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);

Promise.all([p1, p2])
  .then(console.log) // [{ "status": "fulfilled", "value": 1 }, { "status": "fulfilled", "value": 2 }]
```

p1, p2가 각각 2s, 4s가 소요되는 promise 배열을 넘겼을 때 promise.all 메서드는 비동기로 처리하기 때문에
총 소요시간은 작업 소요시간 중 가장 긴 시간인 4s 정도가 됩니다.

<br>
<br>

단 promise.all 메서드는 모든 promise가 성공을 해야 작업 결과를 반환합니다.  
하나라도 실패하는 순간 즉시 거부(rejected) 상태가 되고 성공한 작업 결과들은 무시합니다.

```typescript
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.reject(new Error("실패!"));

Promise.all([p1, p2, p3])
  .then(console.log) // 실행되지 않음
  .catch(console.error); // Error: 실패!
```

<br>
<br>
<br>

## Promise.allSettled

promise 작업들이 중간에 실패하더라도 실패한 promise를 포함해 처리를 원할 때 promise.allSettled() 메서드를 사용합니다.  
promise.allSettled 실행 시 하나가 실패해도 나머지를 기다리며, 개별 상태를 확인할 수 있습니다.

```typescript
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.reject(new Error("실패!"));

Promise.allSettled([p1, p2, p3]).then(console.log);

/**
 * [출력 결과]
 [
  { "status": "fulfilled", "value": 1 },
  { "status": "fulfilled", "value": 2 },
  { "status": "rejected", "reason": "Error: 실패!" }
 ] 
 */
```

<br>
<br>
<br>

## 결론

여러 Promise 작업들을 처리해야할 때 상황에 맞게 사용하면 될 것 같습니다.  
모든 promise들이 성공해야 의미가 있고 여러 api 요청이 병렬적으로 이뤄져야할 때는 promise.all()를 사용합니다.  
일부 실패해도 나머지 promise 작업들을 처리해야 하고 나머지 실패한 작업들을 따로 처리해줄 필요가 있을 때 promise.allSettled()를 사용합니다.
