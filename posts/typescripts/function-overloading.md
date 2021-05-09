---
description: 인자의 타입에 따라 리턴 타입을 달리 설정하기
slug: typescripts/function-overloading
---

# Function Overloading

JavaScript에서 함수는, 인자 타입이나 개수를 다양한 형태로 받아서 함수 호출이 되는 경우가 있습니다. 이런 경우 TypeScript에서는 Function Overloading 개념을 이용하여 인자의 종류와 개수에 따라서 리턴 타입을 달리 명시해주는 방식을 사용할 수 있습니다.

```typescript
function reverse(input: string | string[]) {
  if (typeof input === "string") {
    return input
      .split("")
      .reverse()
      .join("");
  } else {
    return input.slice().reverse();
  }
}

const reversedHello = reverse("hello"); // 'olleh'
const reversedHelloArray = reverse(["h", "e", "l", "l", "o"]); // ['o', 'l', 'l', 'e', 'h']
```

위 함수는 input으로 `string | string[]`을 받을 수 있는데요, 어떤 인자 타입인지에 따라서 결과물이 달라지게 됩니다. 원하는 동작은 하지만, 이 함수의 리턴 타입은 `string | string[]`으로 추론이 됩니다. 사실 정확하게는 input의 type이 `string`일 때는 리턴 타입이 `string`이고, `string[]` 일 때는 `string[]` 인 것이 정확합니다. 이를 해결해주기 위해 function overloading을 이용합니다.



```typescript
function reverse(input: string): string;
function reverse(input: string[]): string[];

function reverse(input: string | string[]) {
  if (typeof input === "string") {
    return input
      .split("")
      .reverse()
      .join("");
  } else {
    return input.slice().reverse();
  }
}
```

함수 선언문 위에 두 줄을 추가해줬습니다. 이제 reverse 함수의 리턴 타입은 인자의 종류에 따라서 정확하게 추론이 가능합니다. 👻  
타입스크립트 공식 홈페이지에 있는 하나의 사례를 추가로 들어보겠습니다.



```typescript
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
// No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.(2575)
```

새로운 날짜 instance를 생성하는 함수인데요, 사용의 편의성을 위해 `timestamp`를 받을 수도 있고, 아니면 month, day, year를 각각 받을 수도 있게 해주는 함수입니다. 위처럼 선언해주면, 인자가 한 개일 경우에는 else 문의 로직을 타고 Date 타입을 리턴하고, 인자가 3개라면 if 문을 타고 Date 타입을 리턴합니다.  
그러나 맨 마지막 `d3`처럼 인자를 2개만 넣어주게 되면 어떤 조건에도 해당되지 않기 때문에 타입 에러를 일으키게 됩니다.