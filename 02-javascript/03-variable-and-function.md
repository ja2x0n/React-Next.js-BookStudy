# 2.3 변수와 함수

변수와 함수는 대부분의 프로그래밍 언어에서 기본이 되는 요소다.
JavaScript에서는 동적 타입과 일급 객체라는 특징 때문에 더 중요한 의미를 가진다.

- 변수는 값을 저장하는 이름이다.
- 함수는 작업을 하나로 묶은 코드다.
- 함수는 값처럼 다룰 수 있다.

함수는 변수에 할당할 수 있다.
다른 함수의 인자로 전달할 수도 있다.
함수의 반환값으로 사용할 수도 있다.
이런 특징을 바탕으로 콜백 함수와 고차 함수와 클로저를 구현한다.

## 변수 선언과 스코프

JavaScript의 변수 선언 방식에는 `var`와 `let`과 `const`가 있다.
`let`과 `const`는 ES6에서 도입되었다.

### `var`

`var`는 함수 스코프를 가진다.
함수 안에서 선언하면 선언된 위치와 관계없이 해당 함수 전체에서 접근할 수 있다.

또한 `var`는 호이스팅의 영향을 받는다.
호이스팅은 변수 선언이 코드의 위쪽으로 끌어올려진 것처럼 동작하는 현상이다.
값을 할당하는 과정까지 올라가는 것은 아니다.

```js
console.log(x); // undefined
var x = 10;
```

위 코드는 다음과 비슷하게 동작한다.

```js
var x;
console.log(x);
x = 10;
```

`var`는 중복 선언과 예상하지 못한 값 변경을 만들기 쉽다.
그래서 현대 JavaScript에서는 사용을 피하는 편이다.

### `let`과 `const`

`let`과 `const`는 블록 스코프를 가진다.
블록은 중괄호 `{}`로 감싸진 코드 영역이다.

```js
if (true) {
    let message = "hello";
}

console.log(message); // ReferenceError
```

`message`는 블록 안에서만 사용할 수 있다.
이런 제한 덕분에 변수의 사용 범위를 예측하기 쉽다.

`let`은 값을 다시 할당할 수 있다.
`const`는 변수 자체에 다른 값을 다시 할당할 수 없다.

```js
let count = 1;
count = 2;

const name = "Alice";
name = "Bob"; // TypeError
```

### `const`와 완전한 불변성

`const`는 변수에 다른 값을 다시 할당하지 못하게 한다.
객체나 배열 내부의 값까지 바꾸지 못하게 만드는 것은 아니다.

```js
const numbers = [1, 2];
numbers.push(3, 4);
console.log(numbers); // [1, 2, 3, 4]

const user = { name: "Alice" };
user.name = "Bob";
```

따라서 `const`를 사용했다고 값 전체가 완전히 불변이 되는 것은 아니다.
변수와 값의 차이를 구분해야 한다.

### 변수 선언 기준

| 키워드 | 스코프 | 재할당 | 완전한 불변성 | 사용 기준 |
| --- | --- | --- | --- | --- |
| `var` | 함수 | 가능 | 없음 | 사용하지 않는 것을 권장 |
| `let` | 블록 | 가능 | 없음 | 값을 바꿔야 할 때 사용 |
| `const` | 블록 | 불가능 | 보장하지 않음 | 기본적으로 사용 |

실무에서는 먼저 `const`를 사용한다.
값을 다시 할당해야 할 때만 `let`으로 바꾼다.
`var`는 특별한 이유가 없다면 사용하지 않는다.

## 함수 선언 방식

### 함수 선언식

```js
function greet(name) {
    return `Hello, ${name}`;
}

console.log(greet("Alice"));
```

함수 선언식은 이름을 명확하게 정의하는 전통적인 방식이다.
함수 선언식은 호이스팅되므로 선언 전에 호출할 수 있다.

```js
sayHi(); // Hi!

function sayHi() {
    console.log("Hi!");
}
```

### 함수 표현식

함수 표현식은 함수를 변수에 할당하는 방식이다.

```js
const greet = function (name) {
    return `Hello, ${name}`;
};

console.log(greet("Alice"));
```

함수 표현식은 선언 전에 호출할 수 없다.

```js
sayHi(); // ReferenceError

const sayHi = function () {
    console.log("Hi!");
};
```

함수의 선언과 사용 순서를 명확하게 관리하고 싶을 때 유용하다.

### 화살표 함수

화살표 함수는 ES6에서 도입된 간결한 함수 표현식이다.

```js
const greet = (name) => `Hello, ${name}`;

console.log(greet("Alice"));
```

화살표 함수는 자신만의 `this`를 만들지 않는다.
대신 함수를 감싸고 있는 외부 스코프의 `this`를 사용한다.

```js
const user = {
    name: "Alice",
    sayHi: () => {
        console.log(this.name); // undefined
    },
};

user.sayHi();
```

이런 이유로 객체 메서드에는 일반 함수를 사용하는 편이 적절하다.
반면 콜백 함수와 이벤트 핸들러와 React의 `useEffect` 안에서는 화살표 함수를 자주 사용한다.

### 즉시 실행 함수 표현식

즉시 실행 함수 표현식은 함수를 정의하자마자 실행하는 패턴이다.

```js
(function () {
    console.log("이 함수는 바로 실행됩니다.");
})();
```

과거에는 전역 스코프 오염을 막고 변수를 숨기기 위해 자주 사용했다.
현재는 ES 모듈이 널리 사용되면서 사용 빈도가 줄었다.
레거시 코드나 오래된 라이브러리에서 만날 수 있다.

## 함수 선언 방식 선택 기준

| 방식 | 특징 | 자주 사용하는 상황 |
| --- | --- | --- |
| 함수 선언식 | 호이스팅됨 | 공용 유틸 함수 |
| 함수 표현식 | 선언 이후에만 호출 가능 | 호출 순서를 명확히 관리할 때 |
| 화살표 함수 | 간결하고 외부 `this`를 사용함 | 콜백과 이벤트 핸들러 |
| IIFE | 정의와 동시에 실행함 | 레거시 코드와 초기화 코드 |

## 함수는 일급 객체

JavaScript에서 함수는 값처럼 다룰 수 있다.
이런 대상을 **일급 객체**라고 부른다.

함수는 다음과 같이 사용할 수 있다.

- 변수에 할당할 수 있다.
- 다른 함수의 인자로 전달할 수 있다.
- 다른 함수의 반환값으로 사용할 수 있다.

### 함수를 변수에 할당하기

```js
const sayHello = function () {
    console.log("Hello");
};

sayHello();
```

`sayHello`는 함수의 이름처럼 보이지만 실제로는 함수 값을 저장한 변수다.

### 함수를 인자로 전달하기

함수를 다른 함수의 인자로 전달하면 콜백 함수가 된다.

```js
function greet(callback) {
    console.log("Before greeting");
    callback();
    console.log("After greeting");
}

greet(function () {
    console.log("Hi there!");
});
```

콜백 함수는 비동기 처리와 이벤트 처리에서 자주 사용한다.

### 함수를 반환값으로 사용하기

함수를 반환하는 함수를 고차 함수라고 한다.

```js
function multiply(factor) {
    return function (value) {
        return value * factor;
    };
}

const double = multiply(2);
console.log(double(5)); // 10
```

`multiply`가 반환한 함수는 외부의 `factor` 값을 기억한다.
이런 동작은 클로저와 관련이 있다.

## 일반 함수와 화살표 함수의 `this`

일반 함수에서 `this`는 호출 방식에 따라 결정된다.
객체의 메서드로 호출하면 해당 객체를 가리킨다.

```js
const user = {
    name: "Alice",
    sayHi: function () {
        console.log(this.name); // Alice
    },
};

user.sayHi();
```

화살표 함수는 호출 방식으로 `this`를 결정하지 않는다.
함수가 정의된 외부 스코프의 `this`를 사용한다.

```js
const user = {
    name: "Alice",
    sayHi: () => {
        console.log(this.name); // undefined
    },
};

user.sayHi();
```

따라서 객체 메서드에는 일반 함수를 사용하는 편이 안전하다.

DOM 이벤트에서도 두 방식의 `this`가 다르게 동작한다.

```js
const button = document.querySelector("button");

button.addEventListener("click", function () {
    console.log(this); // 클릭된 button 요소
});

button.addEventListener("click", () => {
    console.log(this); // 외부 스코프의 this
});
```

## 핵심 정리

JavaScript에서는 `const`를 기본으로 사용하고 값의 재할당이 필요할 때만 `let`을 사용한다.
함수는 값처럼 전달하고 반환할 수 있으므로 콜백과 고차 함수와 클로저를 만들 수 있다.
함수 선언 방식과 `this`의 차이를 이해하면 React 코드를 읽고 작성하기 쉬워진다.
