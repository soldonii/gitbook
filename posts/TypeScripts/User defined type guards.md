---
description: 유저가 정의한 타입으로 narrowing하는 방법
slug: typescripts/user-defined-type-guards
---

TypeScript를 사용하다보면, 한 타입이 여러 개의 타입을 가지는 경우가 존재합니다.
예를 들어, `type Foo = string | number | string[] | number[]` 같은 타입이 존재할 수 있는데, 여러 union 타입 중 어떤 타입에 해당되는지에 따라 로직을 나누어 처리할 경우가 생깁니다.

이렇듯 TypeScript에서는 내가 처리해야 하는 type으로 범위를 좁혀나가는 방식을 종종 활용하게 되는데요, 이렇게 타입을 좁혀나가는 것을 type narrowing이라고 합니다. 이 포스트에서는 내가 원하는 type으로 narrowing하는 방법을 알아보고자 합니다.

# Type Narrowing

상황에 따라 여러 방법으로 타입을 좁힐 수 있습니다.

### 1. 공통되지 않은 property로 좁히기

```typescript
type Square = {
  size: number;
};

type Rectangle = {
  width: number;
  height: number;
};

type Shape = Square | Rectangle;

function area(shape: Shape) {
  if ("size" in shape) {
    // shape가 Square임을 파악하기 위한 type narrowing
    return shape.size * shape.size;
  }
  if ("width" in shape) {
    // shape가 Rectangle임을 파악하기 위한 type narrowing
    return shape.width * shape.height;
  }
  const _ensure: never = shape;
  return _ensure;
}
```

간단한 코드라 이해가 어렵지는 않을 것 같습니다. `Square` 타입은 `size` property를 가지고 있고, `Rectangle`은 `width` property를 가지고 있으니 이 특징을 활용하여 타입을 좁혀줬습니다.

그러나 확장성의 관점에서는 상당히 취약하다는 느낌이 듭니다. 만약 `Triangle` 이라는 타입이 추가되어야 하는 상황이고, `Triangle`이 `width`와 `height`를 property로 가지게 된다면 `if ("width" in shape)` 코드 만으로는 특정 타입으로 좁힐 수가 없습니다. `Rectangle`과 `Triangle` 모두 `"width"` property를 가지기 때문입니다.

---

### 2. 공통 property로 좁히기

이런 문제를 해결하기 위해 모든 shape에 공통 property를 줄 수도 있습니다.

```typescript
type Square = {
  name: "square";
  size: number;
};

type Rectangle = {
  name: "rectangle";
  width: number;
  height: number;
};

type Shape = Square | Rectangle;

function area(shape: Shape) {
  if (shape.name === "square") {
    // shape가 Square임을 파악하기 위한 type narrowing
    return shape.size * shape.size;
  }
  if (shape.name === "rectangle") {
    // shape가 Rectangle임을 파악하기 위한 type narrowing
    return shape.width * shape.height;
  }
  const _ensure: never = shape;
  return _ensure;
}
```

`name` property는 모든 shape가 공동으로 가지면서도 그 값은 모두 다르므로 타입을 좁히기에 용이합니다.

여기서 유저가 직접 정의하는 type guard의 방식을 활용해 볼 수 있습니다.

### 3. user defined type guards로 좁히기

```typescript
type Square = {
  name: "square";
  size: number;
};

type Rectangle = {
  name: "rectangle";
  width: number;
  height: number;
};

type Shape = Square | Rectangle;

function isSquare(shape: Shape) {
  return shape.name === "square";
}

function area(shape: Shape) {
  if (isSquare(shape)) {
    return shape.size * shape.size;
    // Error: Property 'size' does not exist on type 'Rectangle'
  }
  // ...
}
```

`isSquare(shape)` 함수를 통과하면 우리는 분명 shape의 타입이 `Square`라는 것을 알고 있습니다. 하지만 타입스크립트 컴파일러는 `shape`의 타입이 `Square`라는 것을 인지하지 못합니다.

이를 해결하려면 `isSquare(shape)` 함수를 따로 빼지 않고, 인라인으로 처리할 수는 있습니다.

```typescript
function area(shape: Shape) {
  if (shape.name === "square") {
    // `isSquare` 함수를 인라인으로 처리
    return shape.size * shape.size;
  }
  // ...
}
```

그러나 인라인으로 빼지 않고도 타입스크립트 컴파일러에게 타입을 알려줄 수도 있습니다. 유저가 직접 type guard를 해주면 가능합니다.

함수의 return type을 명시적으로 정의해주면 되는데요, 기본 문법은 `parameter is Type`처럼 작성하면 됩니다.

```typescript
type Square = {
  name: "square";
  size: number;
};

type Rectangle = {
  name: "rectangle";
  width: number;
  height: number;
};

type Shape = Square | Rectangle;

function isSquare(shape: Shape): shape is Square {
  return shape.name === "square";
}

function isRectangle(shape: Shape): shape is Rectangle {
  return shape.name === "rectangle";
}

function area(shape: Shape) {
  if (isSquare(shape)) {
    return shape.size * shape.size;
  }
  // ...
}
```

`isSquare` 함수의 return type을 명시적으로 작성해주었는데요, 만약 `isSquare` 함수가 `true`를 리턴한다면, 그 이후에 등장하는 `shape`는 `Square` 타입으로 좁혀질 수 있습니다. 👻

{% hint style="info" %}
📕 **References**
https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates
{% endhint %}

{% hint style="info" %}
**학습하며 배운 내용을 정리한 것이므로 틀린 내용이 있을 수 있습니다.**
**정확한 것은 공식 문서를 참고 부탁드리고, 틀린 내용은 지적해주시면 수정하겠습니다.** 🙏🏻
{% endhint %}
