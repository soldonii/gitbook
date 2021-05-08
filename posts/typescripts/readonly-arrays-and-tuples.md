---
description: 언제, 왜 readonly 속성을 사용해야 할까?
slug: typescripts/readonly-arrays-and-tuples
---

# Readonly Arrays and Tuples

TypeScript를 사용하다보면, 함수 인자의 type에 `readonly` 키워드가 붙어있는 경우를 종종 볼 수 있습니다. 아래 예시를 통해 살펴보겠습니다.

```typescript
function reverseSorted(input: number[]): number[] {
  return input.sort().reverse();
}

const start = [1, 2, 3, 4, 5];
const result = reverseSorted(start);

console.log(start); // [5, 4, 3, 2, 1]
console.log(result); // [5, 4, 3, 2, 1]
```

`start` 배열을 인자로 받아서, 반대로 정렬된 `[5, 4, 3, 2, 1]`을 얻고 싶은 함수입니다. 그런데 결과를 보니 변수 `start`에 담긴 값도 같이 변경되었습니다. 왜냐하면 `Array.prototype.sort()` 메소드가 원본 배열을 변형시키는 mutable한 메소드이기 때문입니다.

따라서 의도치 않은 변형을 방지하려면 반드시 원본 배열을 복사한 후 사용해야 합니다. 타입스크립트를 이용하여 원본 배열을 건드리지 못하도록 강제할 수 있는데요, 이 때 `readonly` 키워드를 사용하면 좋습니다.



```typescript
function reverseSorted(input: readonly number[]): number[] {
  return input.sort().reverse();
  // Property 'sort' does not exist on type 'readonly number[]'
}
```

`input`에 `readonly`를 붙여주는 순간 에러가 발생합니다. 배열에 `readonly`를 붙여주는 순간, 원본 배열에 변형을 가할 수 있는 메소드들(e.g. `sort`, `push`, `pop` 등)은 사용이 불가하도록 타입 에러를 내줍니다.

```typescript
function reverseSorted(input: readonly number[]): number[] {
  const copied = input.slice(); // number[]
  return copied.sort().reverse();
  // Property 'sort' does not exist on type 'readonly number[]'
}
```

원본 배열을 `input.slice()` 혹은 `[...input]` 이렇게 copy를 해주면 이제 타입이 `readonly number[]`가 아니라 `number[]`가 되기 때문에 전체 메소드들을 사용할 수 있게 됩니다.

이처럼 `readonly` 키워드를 이용해서 원본에 변형을 가할 수 있는 코드에 컴파일 단계에서 타입 에러를 일으켜서 의도치 않은 에러를 사전에 방지할 수 있습니다. 💪🏻
