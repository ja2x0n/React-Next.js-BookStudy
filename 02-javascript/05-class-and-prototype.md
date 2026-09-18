# 2.5 클래스와 프로토타입

JavaScript는 프로토타입 기반으로 객체를 상속하는 언어다.
ES6에서 `class` 문법이 추가됐지만 내부 동작은 여전히 프로토타입을 바탕으로 한다.

- 프로토타입은 객체가 다른 객체의 속성과 메서드를 사용할 수 있게 하는 연결고리다.
- 클래스는 객체를 만들기 위한 설계도처럼 사용할 수 있는 문법이다.
- React에서는 클래스보다 함수 컴포넌트와 조합을 더 자주 사용한다.

## 클래스 문법

### 생성자와 인스턴스 메서드

```js
class Car {
  constructor(brand, model, year) {
    this.brand = brand;
    this.model = model;
    this.year = year;
  }

  getCarInfo() {
    return `${this.year} ${this.brand} ${this.model}`;
  }

  static compareCars(car1, car2) {
    if (car1.year === car2.year) {
      return "두 자동차의 연식이 같습니다.";
    }

    return car1.year > car2.year
      ? `${car1.model}이 더 최신입니다.`
      : `${car2.model}이 더 최신입니다.`;
  }
}

const car1 = new Car("Tesla", "Model S", 2020);
const car2 = new Car("Hyundai", "Ioniq 5", 2021);

console.log(car1.getCarInfo()); // "2020 Tesla Model S"
console.log(Car.compareCars(car1, car2)); // "Ioniq 5이 더 최신입니다."
```

- `constructor`는 `new`로 인스턴스를 만들 때 실행된다.
- `this`는 새로 만들어진 인스턴스를 가리킨다.
- `getCarInfo`는 인스턴스가 사용하는 메서드다.
- `static` 메서드는 인스턴스가 아니라 클래스에서 직접 호출한다.

위 코드에서 `car1`과 `car2`는 `Car` 클래스로 만든 인스턴스다.
클래스 문법은 객체 생성과 상속 코드를 읽기 쉽게 작성하도록 도와준다.

### 문법적 설탕

클래스는 프로토타입을 대체하는 새로운 객체 모델이 아니다.
기존 프로토타입 기반 동작을 더 간결하게 표현하는 문법적 설탕(syntactic sugar)에 가깝다.

문법적 설탕은 기능은 그대로 두고 코드를 더 읽기 쉽게 만드는 문법을 뜻한다.

## 프로토타입 방식

ES6 이전에는 생성자 함수와 `prototype`을 직접 사용해 객체를 만들었다.

```js
function Car(brand, model, year) {
  this.brand = brand;
  this.model = model;
  this.year = year;
}

Car.prototype.getCarInfo = function () {
  return `${this.year} ${this.brand} ${this.model}`;
};

Car.compareCars = function (car1, car2) {
  return car1.year > car2.year
    ? `${car1.model}이 더 최신입니다.`
    : `${car2.model}이 더 최신입니다.`;
};

const oldCar = new Car("Kia", "EV6", 2022);
console.log(oldCar.getCarInfo()); // "2022 Kia EV6"
```

`Car.prototype.getCarInfo`에 메서드를 저장하면 모든 인스턴스가 같은 메서드를 공유한다.
클래스의 인스턴스 메서드도 내부적으로는 이와 비슷하게 프로토타입에 저장된다.

## 프로토타입 체인

객체가 특정 속성이나 메서드를 직접 가지고 있지 않으면 JavaScript는 프로토타입을 따라가며 찾는다.
이 연결 구조를 프로토타입 체인이라고 한다.

```js
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function () {
  return `Hello, my name is ${this.name}`;
};

const john = new Person("John");

console.log(john.greet()); // "Hello, my name is John"
console.log(john.hasOwnProperty("name")); // true
console.log(john.hasOwnProperty("greet")); // false
```

`john`은 `greet`를 직접 가지고 있지 않다.
그래도 `Person.prototype`에서 메서드를 찾아 호출할 수 있다.

- `hasOwnProperty("name")`은 객체 자신이 가진 속성이므로 `true`다.
- `hasOwnProperty("greet")`은 프로토타입에 있는 메서드이므로 `false`다.

프로토타입은 `Object.getPrototypeOf(object)`로 확인할 수 있다.

```js
console.log(Object.getPrototypeOf(john) === Person.prototype); // true
```

## 캡슐화

캡슐화는 객체 내부의 데이터를 외부에서 함부로 바꾸지 못하게 감추는 방식이다.
외부에는 필요한 메서드만 공개하고 내부 데이터는 정해진 규칙을 통해 수정한다.

### getter와 setter

getter와 setter는 속성에 접근하는 것처럼 메서드를 실행할 수 있게 해준다.

```js
class User {
  constructor(name) {
    this._name = name;
  }

  get name() {
    return this._name;
  }

  set name(value) {
    if (value.trim().length === 0) {
      throw new Error("이름은 비어 있을 수 없습니다.");
    }

    this._name = value;
  }
}

const user = new User("Alice");
console.log(user.name); // getter 호출

user.name = "Bob"; // setter 호출
console.log(user.name); // "Bob"
```

`_`으로 시작하는 이름은 내부용이라는 개발자 규칙일 뿐 실제 접근을 막지는 못한다.

### private field

private field는 식별자 앞에 `#`을 붙여 클래스 밖에서 접근할 수 없게 만든다.
private field는 ES2022에 정식으로 도입됐다.

```js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
    }
  }

  get balance() {
    return this.#balance;
  }
}

const account = new BankAccount();
account.deposit(10000);

console.log(account.balance); // 10000
// console.log(account.#balance); // SyntaxError
```

`#balance`은 클래스 밖에서 직접 읽거나 수정할 수 없다.
대신 `deposit`과 `balance`처럼 공개된 메서드와 getter를 통해서만 다룬다.

## 상속

상속은 기존 클래스의 기능을 물려받아 새로운 클래스를 만드는 방식이다.

```js
class Animal {
  speak() {
    console.log("Some sound");
  }
}

class Dog extends Animal {
  bark() {
    console.log("Woof!");
  }
}

const dog = new Dog();
dog.speak(); // Some sound
dog.bark(); // Woof!
```

`extends`를 사용하면 부모 클래스의 메서드를 재사용할 수 있다.
하지만 상속 단계가 깊어지면 부모 클래스에 대한 의존성이 커져 수정이 어려워질 수 있다.

## 조합

조합(composition)은 여러 작은 기능을 필요한 객체에 조립하는 방식이다.
상속보다 기능을 유연하게 선택할 수 있어서 React의 컴포넌트 설계에서도 자주 사용한다.

```js
const canBark = (state) => ({
  bark: () => console.log(`${state.name} barks!`),
});

const canRun = (state) => ({
  run: () => console.log(`${state.name} runs!`),
});

const createDog = (name) => {
  const state = { name };

  return {
    ...canBark(state),
    ...canRun(state),
  };
};

const composedDog = createDog("Buddy");
composedDog.bark(); // Buddy barks!
composedDog.run(); // Buddy runs!
```

조합은 필요한 기능만 선택해서 사용할 수 있다는 장점이 있다.
React에서 여러 작은 컴포넌트를 조합해 화면을 만드는 방식도 조합의 예다.

## 핵심 정리

- JavaScript의 객체 상속은 프로토타입 체인을 통해 동작한다.
- `class`는 프로토타입 기반 동작을 읽기 쉽게 표현하는 문법이다.
- `constructor`는 인스턴스 초기화에 사용한다.
- `static` 메서드는 클래스에서 직접 호출한다.
- 캡슐화는 내부 데이터를 보호하고 사용 방법을 제한한다.
- 상속은 기능 재사용에 유용하지만 복잡한 계층은 유지보수를 어렵게 만든다.
- 조합은 필요한 기능을 작은 단위로 나누어 유연하게 재사용하는 방식이다.
