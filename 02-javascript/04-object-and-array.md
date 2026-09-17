# 2.4 객체와 배열

객체와 배열은 JavaScript에서 데이터를 구조화할 때 가장 많이 사용하는 자료구조다.

- 객체는 키와 값의 조합으로 데이터를 표현한다.
- 배열은 여러 값을 순서대로 저장한다.
- React에서는 객체와 배열로 컴포넌트의 상태와 목록 데이터를 관리한다.

## 객체(Object)

객체는 중괄호 안에 `key: value` 형태로 속성을 작성한다.
값에는 문자열과 숫자뿐 아니라 배열과 함수와 다른 객체도 넣을 수 있다.

```js
const person = {
  name: "John",
  age: 30,
  job: "Developer",
};
```

### 속성 접근

점 표기법과 대괄호 표기법으로 객체의 속성에 접근할 수 있다.

```js
console.log(person.name); // "John"
console.log(person["job"]); // "Developer"
```

대괄호 표기법은 속성 이름을 변수로 사용할 때 유용하다.

```js
const property = "name";
console.log(person[property]); // "John"
```

### 속성 추가와 수정과 삭제

`const`로 선언한 객체도 속성의 값은 수정할 수 있다.
`const`는 객체 자체를 다른 객체로 바꾸지 못하게 할 뿐이다.

```js
person.age = 31; // 수정
person["job"] = "Senior Developer"; // 수정
person.city = "Seoul"; // 추가

delete person.age; // 삭제

console.log(person);
// { name: "John", job: "Senior Developer", city: "Seoul" }
```

### 객체를 탐색하는 메서드

`Object.keys`와 `Object.values`와 `Object.entries`로 객체의 데이터를 배열로 변환할 수 있다.

```js
const user = {
  name: "Alice",
  age: 28,
};

console.log(Object.keys(user)); // ["name", "age"]
console.log(Object.values(user)); // ["Alice", 28]
console.log(Object.entries(user));
// [["name", "Alice"], ["age", 28]]
```

## ES6+ 객체 문법

### 구조 분해 할당

구조 분해 할당은 객체의 속성을 꺼내서 변수에 저장하는 문법이다.
React 컴포넌트의 props를 받을 때 자주 사용한다.

```js
const product = {
  name: "Keyboard",
  price: 50000,
};

const { name, price } = product;

console.log(name); // "Keyboard"
console.log(price); // 50000
```

### 객체 리터럴 개선

속성 이름과 변수 이름이 같으면 이름을 한 번만 작성할 수 있다.

```js
const name = "Alice";
const age = 25;

const user = { name, age };
console.log(user); // { name: "Alice", age: 25 }
```

### 스프레드 연산자로 객체 병합

스프레드 연산자(`...`)로 객체를 펼쳐 새로운 객체를 만들 수 있다.
같은 키가 있으면 뒤에 있는 객체의 값이 사용된다.

```js
const first = { name: "Alice", age: 25 };
const second = { age: 26, city: "Seoul" };

const merged = { ...first, ...second };

console.log(merged);
// { name: "Alice", age: 26, city: "Seoul" }
```

React에서 상태를 수정할 때 기존 객체를 직접 바꾸지 않고 새 객체를 만드는 방식으로 자주 사용한다.

## 배열(Array)

배열은 대괄호 안에 값을 순서대로 저장한다.
인덱스는 `0`부터 시작한다.

```js
const fruits = ["Apple", "Banana", "Cherry"];

console.log(fruits[0]); // "Apple"
console.log(fruits.length); // 3
```

### 배열 요소 수정과 추가와 삭제

```js
fruits[1] = "Blueberry"; // 수정
fruits.push("Mango"); // 마지막에 추가
fruits.pop(); // 마지막 요소 삭제
fruits.splice(1, 1); // 인덱스 1부터 1개 삭제
```

`push`와 `pop`과 `splice`는 기존 배열을 직접 수정한다.
React 상태에서는 기존 배열을 직접 수정하기보다 새 배열을 만드는 방식을 사용한다.

### 배열 구조 분해 할당

배열의 요소를 순서대로 변수에 할당할 수 있다.

```js
const colors = ["red", "green", "blue"];
const [first, second] = colors;

console.log(first); // "red"
console.log(second); // "green"
```

### 배열 스프레드

스프레드 연산자로 배열을 펼쳐 새로운 배열을 만들 수 있다.

```js
const fruits = ["Apple", "Banana"];
const combined = [...fruits, "Mango"];

console.log(combined); // ["Apple", "Banana", "Mango"]
```

### 자주 사용하는 배열 메서드

```js
const users = [
  { name: "Alice", age: 28 },
  { name: "Bob", age: 17 },
  { name: "Charlie", age: 33 },
];

// forEach: 순회하면서 작업을 실행한다. 새 배열을 반환하지 않는다.
users.forEach((user) => {
  console.log(`${user.name} is ${user.age} years old`);
});

// map: 각 요소를 변환한 새 배열을 반환한다.
const names = users.map((user) => user.name);
console.log(names); // ["Alice", "Bob", "Charlie"]

// filter: 조건을 만족하는 요소만 모은 새 배열을 반환한다.
const adults = users.filter((user) => user.age >= 20);
console.log(adults);
// [
//   { name: "Alice", age: 28 },
//   { name: "Charlie", age: 33 }
// ]
```

## 이터러블과 이터레이터

배열은 이터러블(iterable) 객체다.
이터러블은 값을 하나씩 꺼낼 수 있는 객체를 뜻한다.

이터레이터(iterator)는 값을 꺼내는 방법을 제공하는 객체다.
`next()`를 호출할 때마다 다음 값을 반환한다.

```js
const array = ["a", "b", "c"];
const iterator = array.values();

console.log(iterator.next()); // { value: "a", done: false }
console.log(iterator.next()); // { value: "b", done: false }
console.log(iterator.next()); // { value: "c", done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

실무에서는 보통 `for...of`나 `map`과 같은 고수준 메서드를 사용한다.
이터러블과 이터레이터의 구조를 이해하면 `Map`과 `Set`과 제너레이터를 배울 때 도움이 된다.

```js
for (const value of array) {
  console.log(value);
}
// a
// b
// c
```

## 객체와 배열의 복사

객체와 배열은 참조 타입이다.
변수에 객체를 할당하면 값 자체가 아니라 메모리의 참조가 복사된다.

```js
const original = { name: "Alice" };
const copy = original;

copy.name = "Bob";

console.log(original.name); // "Bob"
```

`original`과 `copy`가 같은 객체를 바라보기 때문에 하나를 수정하면 다른 변수에서도 변경된 값이 보인다.

### 얕은 복사

얕은 복사는 가장 바깥 객체만 새로 만든다.
중첩된 객체는 기존 참조를 공유한다.

```js
const user = {
  name: "Alice",
  address: {
    city: "Seoul",
  },
};

const shallowCopy = { ...user };
shallowCopy.address.city = "Busan";

console.log(user.address.city); // "Busan"
```

### 깊은 복사

깊은 복사는 중첩된 객체와 배열까지 새로운 참조로 복사한다.

```js
const user = {
  name: "Alice",
  address: {
    city: "Seoul",
  },
};

const deepCopy = structuredClone(user);
deepCopy.address.city = "Busan";

console.log(user.address.city); // "Seoul"
```

`structuredClone`은 최신 브라우저와 Node.js에서 사용할 수 있다.
JSON 방식도 있지만 함수와 `undefined`와 `Date`와 `Map`와 `Set`을 원래 형태로 복사하지 못한다.

```js
const jsonCopy = JSON.parse(JSON.stringify(user));
```

## 핵심 정리

- 객체는 키와 값으로 데이터를 표현한다.
- 배열은 순서가 있는 데이터를 관리한다.
- 구조 분해 할당과 스프레드 연산자는 React에서 자주 사용한다.
- `map`과 `filter`는 기존 배열을 바꾸지 않고 새 배열을 반환한다.
- 객체와 배열은 참조 타입이므로 복사 방식에 주의해야 한다.
- React 상태를 업데이트할 때는 새로운 객체나 배열을 만들어야 한다.
