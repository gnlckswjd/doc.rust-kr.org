## `Box<T>`를 사용하여 힙에 있는 데이터 가리키기

가장 직관적인 스마트 포인터는 *박스 (box)* 인데, `Box<T>`로 쓰여지는
타입입니다. 박스는 스택이 아니라 힙에 데이터를 저장할 수 있도록 해줍니다.
스택에 남는 것은 힙 데이터를 가리키는 포인터입니다. 스택과 힙의
차이에 대해 다시 보고싶다면 4장을 참조하세요.

박스는 스택 대신 힙에 데이터를 저장한다는 점 외에는, 성능 측면에서의 오버헤드가
없습니다. 하지만 여러 추가 기능도 없습니다. 박스는 아래와 같은 상황에서
가장 자주 쓰이게 됩니다:

* 컴파일 타임에는 크기를 알 수 없는 타입이 있는데, 정확한 크기를 요구하는 맥락
  내에서 그 타입의 값을 사용하고 싶을 때
* 커다란 데이터를 가지고 있고 소유권을 옮기고 싶지만 그렇게 했을 때 데이터가
  복사되지 않을 것이라고 보장하고 싶을 때
* 어떤 값을 소유하고 이 값의 구체화된 타입보다는 특정 트레잇을 구현한 타입이라는
  점만 신경 쓰고 싶을 때

첫 번째 상황은 [“박스로 재귀적 타입 가능하게
하기”](#enabling-recursive-types-with-boxes)<!-- ignore -->절에서 보여주겠습니다.
두 번째 경우, 방대한 양의 데이터의 소유권 옮기기는 긴 시간이 소요될 수 있는데
이는 그 데이터가 스택 상에서 복사되기 때문입니다. 이러한 상황에서 성능을
향상시킬 목적으로 박스 안의 힙에 그 방대한 양의 데이터를 저장할 수 있습니다.
그러면 작은 양의 포인터 데이터만 스택 상에서 복사되고, 이 포인터가
참조하는 데이터는 힙 상에서 한 곳에 머물게 됩니다. 세 번째 경우는
*트레잇 객체 (trait object)* 라고 알려진 것이고, 17장의 [“트레잇 객체를
사용하여 서로 다른 타입의 값 허용하기”][trait-objects]<!-- ignore -->절
전체가 이 주제만으로 채워져 있습니다. 그러니 여기서 배운 것을 17장에서 다시
적용하게 될 것입니다!

### `Box<T>`을 사용하여 힙에 데이터 저장하기

`Box<T>`에 대한 사용례를 논의하기 전에, 먼저 문법 및 `Box<T>` 내에
저장된 값의 사용법을 다루겠습니다.

Listing 15-1은 박스를 사용하여 힙에 `i32` 값을 저장하는 방법을 보여줍니다:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

<span class="caption">Listing 15-1: 박스를 사용하여 `i32` 값을 힙에
저장하기</span>

변수 `b`를 정의하여 `5`라는 값을 가리키는 `Box` 값을 갖도록 했는데, 여기서
`5`는 힙에 할당됩니다. 이 프로그램은 `b = 5`를 출력할 것입니다; 이 경우,
박스 안에 있는 데이터는 마치 이 데이터가 스택에 있는 것처럼 접근
가능합니다. 다른 어떤 소유값과 마찬가지로, `b`가 `main`의 끝에
도달하는 것처럼 어떤 박스가 스코프를 벗어날 때, 할당은 해제될 것입니다.
할당 해제는 (스택에 저장된) 박스와 이것이 가리키고 있는 (힙에 저장된)
데이터 모두에게 일어납니다.

단일 값을 힙에 집어넣는 것은 그다지 유용하지는 않으므로, 이 같은 방식의
박스 사용은 자주 쓰이지 않을 것입니다. 단일 `i32`의 저장 공간은
기본적으로 스택이고, 이러한 값을 스택에 저장하는 것이 대부분의 경우에
더 적합합니다. 박스를 쓰지 않으면 허용되지 않았을 타입을 박스로 정의하는
경우를 살펴봅시다.

### 박스로 재귀적 타입 가능하게 하기

*재귀적 타입 (recursive type)* 의 값은 자신 안에 동일한 타입의 또다른 값을
담을 수 있습니다. 컴파일 타임에 러스트는 어떤 타입이 얼만큼의 공간을 잡아먹는지
알아야 하기 때문에 재귀적 타입은 문제를 일으킵니다. 재귀적 타입의 값 중첩은
이론적으로 무한히 계속될 수 있으므로, 러스트는 이 값에 얼만큼의 공간이
필요한지 알 수 없습니다. 박스는 알려진 크기를 갖고 있으므로, 재귀적 타입의
정의에 박스를 집어 넣어서 재귀적 타입을 가능하게 할 수 있습니다.

재귀적 타입의 예제로, *cons list*를 탐구해 봅시다. 이것은 함수형
프로그램 언어에서 흔히 발견되는 데이터 타입입니다. 여기서 정의할
cons list 타입은 재귀를 제외하면 직관적입니다; 따라서 여기서
작업할 예제의 개념은 재귀적 타입을 포함하는 더 복잡한 경우에
직면하더라도 유용할 것입니다.

#### Cons List에 대한 더 많은 정보

*cons list*는 Lisp 프로그래밍 언어 및 그의 파생 언어들로부터 유래된 데이터
구조로서 중첩된 쌍으로 구성되며, 연결 리스트(linked list)의 Lisp
버전입니다. 이 이름은 Lisp의  (“생성 함수 (construct function)”의 줄임말인)
`cons` 함수에서 유래되었는데, 이 함수는 두 개의 인자로부터 새로운 쌍을 생성합니다.
`cons`에 어떤 값과 다른 쌍으로 구성된 쌍을 넣어 호출함으로써 재귀적인 쌍으로
이루어진 cons list를 구성할 수 있습니다.

예를 들어, 다음은 1, 2, 3 리스트를 담고 있는 cons list를 각각의 쌍을 괄호로
묶어서 표현한 의사 코드입니다:

```text
(1, (2, (3, Nil)))
```

cons list 내의 각 아이템은 두 개의 요소를 담고 있습니다: 현재 아이템의
값과 다음 아이템이지요. 리스트의 마지막 아이템은 다음 아이템 없이 `Nil`
이라 불리는 값을 담고 있습니다. cons list는 `cons` 함수를 재귀적으로
호출함으로써 만들어집니다. 재귀의 기본 케이스를 의미하는 표준 이름이 바로
`Nil` 입니다. 6장의 “null” 혹은 “nil” 개념과 동일하지 않다는 점을 주의하세요.
그것들은 값이 유효하지 않거나 없음을 말합니다.

cons list는 러스트에서 흔히 사용되는 데이터 구조는 아닙니다. 러스트에서
아이템 리스트를 쓰는 대부분의 경우에는 `Vec<T>`가 더 나은 선택입니다.
그와는 다른, 더 복잡한 재귀적 데이터 타입들은 다양한 상황에서 유용*하기는*
하지만, 이 장에서는 cons list로 시작하여 박스가 어떤 식으로 별로 주의를
기울이지 않고도 재귀적 데이터 타입을 정의하도록 하는지 탐구하겠습니다.

Listing 15-2는 cons list를 위한 열거형 정의를 담고 있습니다. `List` 타입이
알려진 크기를 가지고 있지 않고 있기 때문에 이 코드는 아직 컴파일이 안된다는
점을 유의하세요. 이것이 여기서 보여주려는 것입니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

<span class="caption">Listing 15-2: `i32` 값의 cons list 데이터 구조를
표현하는 열거형 정의에 대한 첫 번째 시도</span>

> Note: 이 예제의 목적을 위해 오직 `i32` 값만 담는 cons list를
> 구현하는 중입니다. 10장에서 논의했던 것처럼, 제네릭을 이용하면
> 임의의 타입 값을 저장할 수 있는 cons list 타입을 정의할 수도
> 있습니다.

`List` 타입을 이용하여 리스트 `1, 2, 3`을 저장하는 것은 Listing 15-3의
코드처럼 보일 것입니다:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

<span class="caption">Listing 15-3: `List` 열거형을 이용하여 리스트 `1, 2, 3`
저장하기</span>

첫 번째 `Cons` 값은 `1`과 또다른 `List` 값을 갖습니다. 이 `List` 값은
`2`와 또다른 `List` 값을 갖는 `Cons` 값입니다. 그 안의 `List` 값에는 `3`과
`List` 값을 갖는 `Cons`가 하나 더 있는데, 여기서 마지막의 `List`은 `Nil`로서,
리스트의 끝을 알리는 비재귀적인 variant입니다.

만일 Listing 15-3 코드의 컴파일을 시도하면, Listing 15-4과 같은
에러를 얻습니다:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

<span class="caption">Listing 15-4: 재귀적 열거형을 정의하는 시도를 했을 때 얻게
되는 에러</span>

이 에러는 이 타입이 “무한한 크기다”라고 말해줍니다. 그 원인은 재귀적인 variant를
이용하여 `List`를 정의했기 때문입니다: 즉 이것은 자신의 또다른 값을 직접 갖습니다.
결과적으로, 러스트는 `List` 값을 저장하는데 필요한 크기가 얼마나 되는지 알아낼 수
없습니다. 왜 이런 에러가 생기는지 쪼개서 봅시다. 먼저, 러스트가 비재귀적인 타입의
값을 저장하는데 필요한 용량이 얼마나 되는지 결정하는 방법을 살펴봅시다.

#### 비재귀적 타입의 크기 계산하기

6장에서 열거형 정의에 대해 논의할 때 Listing 6-2에서 정의했던 `Message`
열거형을 상기해봅시다:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

`Message` 값을 할당하기 위해 필요한 공간의 양을 결정하기 위해서, 러스트는 각
variant들의 내부를 보면서 어떤 variant가 가장 많은 공간을 필요로 하는지를
알아봅니다. 러스트는 `Message::Quit`가 어떠한 공간도 필요 없음을, `Message::Move`는
두 개의 `i32` 값을 저장하기에 충분한 공간이 필요함을 알게 되고, 그런 식으로
진행됩니다. 하나의 variant만 사용될 것이기 때문에, `Message` 값이 필요로 하는
가장 큰 공간은 varient 중에서 가장 큰 것을 저장하는데 필요한 공간입니다.

러스트가 Listing 15-2의 `List` 열거형과 같은 재귀적 타입이 필요로 하는 공간을
결정하는 시도를 할 때는 어떤 일이 일어날지 위의 경우와 대조해보세요. 컴파일러는
`Cons` variant를 살펴보기 시작하는데, 이는 `i32` 타입의 값과 `List` 타입의 값을
갖습니다. 그러므로 `Cons`는 `i32`의 크기에 `List` 크기를 더한 만큼의 공간을
필요로 합니다. `List` 타입이 얼마나 많은 메모리를 차지하는지 알아내기 위해서,
컴파일러는 그것의 variants를 살펴보는데, 이는 `Cons` variant로 시작됩니다.
`Cons` variant는 `i32` 타입의 값과 `List` 타입의 값을 갖고, 이 과정은 Figure
15-1에서 보는 바와 같이 무한히 계속됩니다.

<img alt="An infinite Cons list" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 15-1: 무한한 `Cons` variant를 가지고 있는
무한한 `List`</span>

#### `Box<T>`를 이용하여 알려진 크기를 가진 재귀적 타입 만들기

러스트는 재귀적으로 정의된 타입을 위하여 얼마큼의 공간을 할당하는지 알아낼 수
없으므로, 컴파일러는 에러와 함께 아래와 같은 유용한 제안을 제공합니다:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

이 제안에서 “간접 (indirection)”이란, 값을 직접 저장하는 대신
데이터 구조를 바꿔 값을 가리키는 포인터를 저장하는 식으로 값을
간접적으로 저장해야 함을 의미합니다.

`Box<T>`가 포인터이기 때문에, 러스트는 언제나 `Box<T>`가 필요로 하는 공간이
얼마인지 알고 있습니다: 포인터의 크기는 그것이 가리키고 있는 데이터의 양에
따라 변경되지 않습니다. 이는 `Cons` variant 내에 또 다른 `List` 값을
직접 넣는 대신 `Box<T>`를 넣을 수 있음을 의미합니다. `Box<T>`는 `Cons`
variant 안이 아니라 힙에 있을 다음의 `List` 값을 가리킬 것입니다.
개념적으로는 여전히 다른 리스트들을 담은 리스트로 만들어진 리스트지만,
이 구현은 이제 아이템을 다른 것 안쪽에 넣는 것이 아니라 그 다음 위치에 놓는
형태에 더 가깝습니다.

Listing 15-2의 `List` 열거형의 정의와 Listing 15-3의 `List` 사용법을
Listing 15-5의 코드로 바꿀 수 있고, 이것은 컴파일될 것입니다:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

<span class="caption">Listing 15-5: 알려진 크기를 갖도록 하기 위해
`Box<T>`를 이용한 `List` 정의</span>

`Cons` variant에는 `i32`와 박스의 포인터 데이터를 저장할 공간을 더한 크기만큼이
필요합니다. `Nil` variant는 아무런 값도 저장하지 않으므로, `Cons` variant에 비해
공간을 덜 필요로 합니다. 이제는 어떠한 `List` 값이라도 `i32`의 크기와 박스의 포인터
데이터 크기를 더한 값만큼만 차지한다는 것을 알게 되었습니다. 박스를 이용하는 것으로
무한한 재귀적 연결을 깨뜨렸고, 따라서 컴파일러는 `List` 값을 저장하는데 필요한
크기를 알아낼 수 있습니다. Figure 15-2는 `Cons` variant가 이제 어떻게 생겼는지를
보여주고 있습니다:

<img alt="A finite Cons list" src="img/trpl15-02.svg" class="center" />

<span class="caption">Figure 15-2: `Cons`가 `Box`를 들고 있기 때문에
무한한 크기가 아니게 된 `List`</span>

박스는 그저 간접 및 힙 할당만을 제공할 뿐입니다; 이들은 다른 어떤 특별한
능력들, 다른 스마트 포인터 타입들에서 보게 될 능력 같은 것들은 없습니다.
또한 이들은 이러한 특별한 능력들이 초래하는 성능적인 오버헤드도 가지고
있지 않으므로, 필요한 기능이 간접 하나인 cons list와 같은 경우에는
유용할 수 있습니다. 17장에서 박스에 대한 더 많은 사용례도 살펴볼
예정입니다.

`Box<T>` 타입은 `Deref` 트레잇을 구현하고 있기 때문에 스마트 포인터이며,
이는 `Box<T>` 값이 참조자와 같이 취급되도록 허용해줍니다. `Box<T>` 값이
스코프 밖으로 벗어날 때, 박스가 가리키고 있는 힙 데이터도 마찬가지로
정리되는데 이는 `Drop` 트레잇의 구현 때문에 그렇습니다. 이 두 트레잇이
이 장의 나머지에서 다루고자 하는 다른 스마트 포인터 타입에 의해 제공되는
기능들보다도 심지어 더 중요할 것입니다. 이 두 트레잇에 대하여 더 자세히
탐구해 봅시다.

[trait-objects]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
