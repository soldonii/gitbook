---
description: chakra-ui 뜯어보면서 알게된 것 1
slug: til/chakra-ui-1
---

# Merge Type

`Merge` 라는 타입을 chakra-ui를 뜯어보다가 보게 되었습니다. 두 개의 interface를 하나의 인터페이스로 합쳐주는 유틸리티성 타입입니다. 실무에서 사용 니즈가 생길 수도 있을 것 같아서 제 것으로 소화하기 위해 정리해봅니다.

```typescript
type Merge<T, P> = P & Omit<T, keyof P>
```

사실 타입 정의는 전혀 복잡하지 않습니다. Merge 타입에 병합하고 싶은 두 개의 interface를 generic type으로 넣어주면 됩니다. ts playground에서 합병된 타입이 어떻게 추론되는지 보기 위해 아주 간단한 예제를 만들어봤습니다.



```typescript
type Merge<T, P> = P & Omit<T, keyof P>

type Person = {
  name: string;
  age: number;
  address: string;
}

type Company = {
  location: {
    lat: string;
    lng: string;
  }
  establishedAt: Date;
}

type Merged = Merge<Person, Company>;
```

`Merge` 타입 자체가 정의된 것을 보면, 두번째 generic(`P`) 타입을 기본으로 가지게 되고, `T`에서 `P`와 key가 겹치는 것을 제외한 나머지 key값을 추가하는 방식입니다.

그래서 만약에 키가 겹치는 type이 들어오게 되면, 두번째 generic에 정의된 type이 최종 type이 됩니다.

```typescript
type Person = {
  name: string;
  age: number;
  address: string;
}

type Company = {
  name: boolean;
  location: {
    lat: string;
    lng: string;
  }
  establishedAt: Date;
}

type Merged = Merge<Person, Company>;
```

말은 안되긴 하지만, 위처럼 `name` 프로퍼티의 타입이 서로 상충되는 경우, 두번째 generic에 들어간 `Company`에 정의된 boolean이 최종 타입으로 추론됩니다.

# Modal overlay click 감지

chakra-ui에서 Modal 컴포넌트가 어떻게 구현되었는지 보다가, 아래와 같은 코드를 보게 되었습니다. `onOverlayClick`인데, 말 그대로 modal 밖의 overlay가 클릭되었을 때 어떤 동작을 할 건지 정의된 함수입니다.

```typescript
const mouseDownTarget = useRef<EventTarget | null>(null)

const onMouseDown = useCallback((event: MouseEvent) => {
  mouseDownTarget.current = event.target
}, [])

const onOverlayClick = useCallback(
    (event: MouseEvent) => {
      event.stopPropagation()
      /**
       * Make sure the event starts and ends on the same DOM element.
       *
       * This is used to prevent the modal from closing when you
       * start dragging from the content, and release drag outside the content.
       *
       * We prevent this because it is technically not a considered "click outside"
       */
      if (mouseDownTarget.current !== event.target) return

      /**
       * When you click on the overlay, we want to remove only the topmost modal
       */
      if (!manager.isTopModal(dialogRef)) return

      if (closeOnOverlayClick) {
        onClose?.()
      }

      onOverlayClickProp?.()
    },
    [onClose, closeOnOverlayClick, onOverlayClickProp],
  )
```

chakra-ui는 내부적으로 유저가 클릭한(mousedown) element가 무엇인지 추적하는 것을 `ref`를 이용해서 하고 있었습니다. `onMouseDown` 함수를 보면, mousedown 이벤트가 일어났을 때, `mouseDownTarget.current`를 클릭된 `event.target`으로 변경하는 것을 볼 수 있습니다.

사용성을 높이기 위해서, 유저가 overlay를 클릭했을 때, 넘겨받은 함수를 실행하도록 되어 있는데, 이 때 클릭 시작은 overlay에서 했는데 드래그를 해서 이벤트 끝은 overlay가 아닌 곳에서 끝났을 경우 overlay가 닫히지 않기를 유저는 기대합니다. 그런 경우에 modal이 닫히지 않게 하기 위해 이렇게 처리를 해주었습니다.

> - event.stopPropagation()을 통해 이벤트가 더 이상 하위 dom으로 전파되지 않도록 막습니다.
> - mouseDown(클릭 시작)이 일어나는 순간 `mouseDownTarget.current`에 클릭된 element가 담기는데, 이 element와 event.target(이벤트 종료 순간의 target)이 같은 element인지를 검사합니다.
> - 같은 element가 아닐 경우 아무 동작도 하지 않기 위해 return 합니다.

이렇게 처리를 해줌으로써, 이벤트 시작과 이벤트 끝이 동일한 element인 것을 보장할 수 있고, 이것이 보장될 때에만 `onClose` 및 `onOverlayClickProp` 핸들러를 실행시켜주고 있었습니다.

나중에 디자인 시스템을 만들 때 이런 기능은 꽤 필요할 것으로 생각되어서 도움이 되었던 것 같습니다.
