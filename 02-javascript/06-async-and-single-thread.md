# 2.6 비동기와 싱글 스레드

웹 애플리케이션은 서버에서 데이터를 가져오고 파일을 읽고 사용자의 입력을 기다린다.
이런 작업은 끝날 때까지 시간이 걸리기 때문에 비동기로 처리한다.

비동기 처리는 작업이 끝날 때까지 다른 코드의 실행을 막지 않는 방식이다.
데이터를 기다리는 동안 스켈레톤 UI나 로딩 스피너를 보여주면 사용자는 화면이 멈췄다고 느끼지 않는다.

## 싱글 스레드

JavaScript는 한 번에 하나의 작업을 처리하는 싱글 스레드 언어다.
코드는 호출 스택(call stack)에 들어온 순서대로 실행된다.

오래 걸리는 동기 작업이 호출 스택을 차지하면 그동안 화면 렌더링과 사용자 입력도 처리할 수 없다.
이런 상태를 블로킹(blocking)이라고 한다.

- UI가 멈춘 것처럼 보인다.
- 클릭과 키보드 입력에 늦게 반응한다.
- 브라우저가 페이지 응답 없음 경고를 보여줄 수 있다.

## 이벤트 루프

JavaScript는 브라우저나 Node.js가 제공하는 기능과 이벤트 루프를 이용해 비동기 작업을 처리한다.

1. 동기 코드는 호출 스택에서 실행된다.
2. `setTimeout`과 `fetch`와 이벤트 리스너 같은 작업은 호스트 환경에 맡긴다.
3. 작업이 끝나면 콜백은 태스크 큐나 마이크로태스크 큐에 들어간다.
4. 이벤트 루프는 호출 스택이 비었는지 확인한다.
5. 호출 스택이 비면 큐의 작업을 스택으로 옮겨 실행한다.

`setTimeout`의 시간이 `0`이어도 즉시 실행되는 것은 아니다.
현재 동기 코드와 큐에 먼저 들어온 작업이 끝난 뒤 실행된다.

```js
console.log("1");

setTimeout(() => {
  console.log("2 - setTimeout");
}, 0);

console.log("3");

// 출력
// 1
// 3
// 2 - setTimeout
```

Promise의 `then`과 `catch`에 등록한 콜백은 마이크로태스크 큐를 사용한다.
마이크로태스크는 일반 태스크보다 먼저 처리된다.

```js
console.log("1");

setTimeout(() => {
  console.log("4 - task");
}, 0);

Promise.resolve().then(() => {
  console.log("3 - microtask");
});

console.log("2");

// 출력
// 1
// 2
// 3 - microtask
// 4 - task
```

## 오래 걸리는 동기 작업

다음 코드는 반복문이 끝날 때까지 메인 스레드를 점유한다.

```js
function heavyTask() {
  for (let i = 0; i < 1_000_000_000; i += 1) {
    // 무거운 연산
  }
}

heavyTask();
console.log("작업 완료");
```

작업을 작은 단위로 나누고 다음 작업을 타이머로 예약하면 이벤트 루프가 다른 이벤트를 처리할 틈을 만들 수 있다.

```js
let count = 0;
const target = 1_000_000_000;

function lightTask() {
  const chunkSize = 100_000;
  const end = Math.min(count + chunkSize, target);

  while (count < end) {
    count += 1;
  }

  if (count < target) {
    setTimeout(lightTask, 0);
  } else {
    console.log("작업 완료");
  }
}

lightTask();
```

작업을 나누면 화면이 덜 멈추지만 전체 작업 시간이 반드시 줄어드는 것은 아니다.
무거운 계산 자체를 다른 스레드로 옮기려면 Web Worker를 사용할 수 있다.

## Web Worker

Web Worker는 JavaScript를 별도 스레드에서 실행하는 브라우저 기능이다.
Worker는 DOM에 직접 접근할 수 없고 `postMessage`로 메인 스레드와 통신한다.

### worker.js

```js
self.onmessage = (event) => {
  const number = event.data;
  let sum = 0;

  for (let i = 1; i <= number; i += 1) {
    sum += i;
  }

  self.postMessage(sum);
};
```

### main.js

```js
const worker = new Worker("worker.js");

worker.onmessage = (event) => {
  console.log("총합은:", event.data);
};

worker.postMessage(1_000_000_000);
```

Web Worker는 이미지 처리와 데이터 분석과 같은 무거운 작업에 적합하다.
대신 별도 파일과 메시지 통신이 필요하므로 간단한 작업에는 과할 수 있다.

## 콜백과 Promise와 async/await

비동기 코드는 콜백에서 Promise로 발전했고 Promise를 더 읽기 쉽게 작성하기 위해 `async/await`이 추가됐다.

### 콜백

콜백은 작업이 끝난 뒤 실행할 함수를 인자로 전달하는 방식이다.

```js
function fetchData(callback) {
  setTimeout(() => {
    callback("데이터 수신 완료");
  }, 1000);
}

fetchData((result) => {
  console.log(result); // "데이터 수신 완료"
});
```

작업이 여러 단계로 이어지면 콜백이 깊게 중첩될 수 있다.
이런 구조를 콜백 지옥(callback hell)이라고 한다.

```js
login(user, (id) => {
  getUserData(id, (data) => {
    showProfile(data, () => {
      console.log("프로필 표시 완료");
    });
  });
});
```

### Promise

Promise는 아직 끝나지 않은 비동기 작업의 결과를 나타내는 객체다.

- `pending`: 작업이 진행 중인 상태
- `fulfilled`: 작업이 성공한 상태
- `rejected`: 작업이 실패한 상태

```js
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("데이터 수신 완료");
    }, 1000);
  });
}

fetchData()
  .then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.error("에러:", error);
  });
```

`then`은 성공한 결과를 이어서 처리하고 `catch`는 실패를 처리한다.
Promise 체이닝을 사용하면 콜백의 중첩을 줄일 수 있다.

### async/await

`async` 함수는 항상 Promise를 반환한다.
`await`은 Promise가 끝날 때까지 해당 `async` 함수 안에서 다음 코드의 실행을 잠시 기다린다.

```js
async function getData() {
  try {
    const result = await fetchData();
    console.log(result);
  } catch (error) {
    console.error("에러:", error);
  }
}

getData();
```

`await`이 전체 JavaScript 스레드를 멈추는 것은 아니다.
기다리는 동안 다른 이벤트와 작업은 이벤트 루프를 통해 처리된다.

## 비동기 코드 리팩터링

### 콜백 중첩 줄이기

```js
// 리팩터링 전
fetchA((a) => {
  fetchB(a, (b) => {
    fetchC(b, (c) => {
      console.log(c);
    });
  });
});
```

### Promise 체이닝

```js
fetchA()
  .then((a) => fetchB(a))
  .then((b) => fetchC(b))
  .then((c) => console.log(c))
  .catch((error) => console.error("에러:", error));
```

### async/await

```js
async function fetchAll() {
  try {
    const a = await fetchA();
    const b = await fetchB(a);
    const c = await fetchC(b);

    console.log(c);
  } catch (error) {
    console.error("에러:", error);
  }
}
```

세 단계가 반드시 앞 단계의 결과를 필요로 한다면 순차 처리가 맞다.
서버와 협의해 여러 단계를 한 번에 처리하는 API를 만들 수 있다면 요청 횟수를 줄일 수도 있다.

## 순차 처리와 병렬 처리

서로 의존하지 않는 작업을 순서대로 `await`하면 불필요하게 오래 걸릴 수 있다.

```js
// 순차 처리
const a = await fetchA();
const b = await fetchB();
const c = await fetchC();
```

서로 독립적인 작업은 `Promise.all`로 동시에 시작할 수 있다.

```js
// 병렬 처리
const [a, b, c] = await Promise.all([
  fetchA(),
  fetchB(),
  fetchC(),
]);
```

`Promise.all`은 하나라도 실패하면 전체 Promise가 실패한다.
각 작업의 성공과 실패를 모두 확인하려면 `Promise.allSettled`를 사용한다.

```js
const results = await Promise.allSettled([
  fetchSafe(1),
  fetchSafe(2),
  fetchSafe(3),
]);

results.forEach((result) => {
  if (result.status === "fulfilled") {
    console.log("성공:", result.value);
  } else {
    console.error("실패:", result.reason);
  }
});
```

## 비동기 코드에서 자주 하는 실수

### `forEach`에서 `await` 사용하기

`forEach`는 내부 콜백이 끝날 때까지 기다려주지 않는다.

```js
const items = [1, 2, 3];

items.forEach(async (item) => {
  await doSomething(item);
});

console.log("forEach는 비동기 작업을 기다리지 않는다.");
```

순서가 중요하면 `for...of`를 사용한다.

```js
for (const item of items) {
  await doSomething(item);
}
```

순서가 상관없고 모든 작업을 함께 시작해도 된다면 `Promise.all`을 사용한다.

```js
await Promise.all(items.map((item) => doSomething(item)));
```

### 순차 실행이 필요한지 확인하지 않기

모든 작업을 병렬로 실행하면 빠를 수 있지만 서버 부하가 커질 수 있다.
API 제한이나 작업 순서를 고려해 실행 방식을 선택해야 한다.

### `then`과 `await`을 무분별하게 섞기

한 함수 안에서는 한 가지 스타일을 유지하는 편이 읽기 쉽다.

```js
async function loadUser() {
  try {
    const response = await fetchData();
    const data = await response.json();

    return data;
  } catch (error) {
    console.error("에러 발생:", error);
    throw error;
  }
}
```

에러를 로그만 남기고 삼키면 호출한 쪽에서 실패를 알 수 없다.
필요하면 `throw`로 에러를 다시 전달해야 한다.

## 핵심 정리

- JavaScript는 싱글 스레드지만 이벤트 루프로 비동기 작업을 처리한다.
- 호출 스택과 큐와 이벤트 루프의 관계를 이해해야 실행 순서를 예측할 수 있다.
- Promise의 콜백은 마이크로태스크 큐에서 먼저 실행된다.
- `async/await`은 Promise를 읽기 쉬운 형태로 작성하는 문법이다.
- 독립적인 작업은 `Promise.all`로 병렬 처리할 수 있다.
- `forEach`는 `await`을 기다리지 않으므로 `for...of`나 `Promise.all`을 사용한다.
- 로딩 UI와 에러 처리는 비동기 화면에서 함께 설계해야 한다.
