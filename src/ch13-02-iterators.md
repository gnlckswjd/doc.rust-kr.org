## 반복자로 일련의 아이템들 처리하기

반복자 패턴은 일련의 아이템들에 대해 순서대로 어떤 작업을 수행할 수
있도록 해줍니다. 반복자는 각 아이템들을 순회하고 언제 시퀀스가
종료될 지 결정하는 로직을 담당합니다. 반복자를 사용하면, 그런 로직을
다시 구현할 필요가 없습니다.

러스트에서의 반복자는 *게으른데*, 이는 반복자를 사용하기 위해 이를 써버리는
메소드를 호출하기 전까지 아무런 동작을 하지 않는다는 의미입니다. 예를 들면,
Listing 13-10의 코드는 `Vec<T>` 에 정의된 `iter` 메소드를 호출함으로써
벡터 `v1`에 있는 아이템들에 대한 반복자를 생성합니다. 이 코드 자체로는 어떤
유용한 동작도 하지 않습니다.

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

<span class="caption">Listing 13-10: 반복자 생성하기</span>

반복자는 `v1_iter` 변수에 저장됩니다. 일단 반복자를 만들면, 다양한 방법으로
사용할 수 있습니다. 3장의 Listing 3-5에서는 각 아이템에 대해 어떤 코드를
실행하기 위해 `for` 루프를 사용하여 어떤 배열에 대한 반복을 수행했습니다.
내부적으로는 암묵적으로 반복자를 생성한 다음 소비하는 것이었지만, 지금까지는
이게 정확히 어떻게 동작하는지에 대해서 대충 넘겼습니다.

Listing 13-11의 예제에서는 `for` 루프에서 반복자를 사용하는 부분으로부터
반복자 생성을 분리했습니다. `v1_iter`에 있는 반복자를 사용하여 `for`
루프가 호출되면, 반복작의 각 원소가 루프의 한 순번마다 사용되는데, 여기서는
각각의 값을 출력합니다.

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

<span class="caption">Listing 13-11: `for` 루프에서 반복자 사용하기</span>

표준 라이브러리에서 반복자를 제공하지 않는 언어에서는, 아마도 변수를
인덱스 0으로 시작하고, 그 변수를 벡터에서 값을 꺼내오기 위한 인덱스로
사용하고 색인해서 값을 가져오는데 사용하고, 루프 안에서 벡터가 가진
아이템의 전체 갯수에 다다를 때까지 그 변수값을 중가시키는 것으로 동일한
기능을 작성할 것입니다.

반복자는 여러분을 위해 그러한 모든 로직을 처리하며, 엉망이 될 가능성이 
있는 반복되는 코드를 줄여 줍니다. 반복자는 벡터처럼 인덱스를 사용할 수 있는
자료구조 뿐만 아니라, 많은 다른 종류의 시퀀스에 대해 동일한 로직을 사용할 수
있도록 더 많은 유연성을 제공 합니다. 반복자가 어떻게 그렇게 하는지 살펴봅시다.

### `Iterator`트레잇과 `next` 메서드

모든 반복자는 표준 라이브러리에 정의된 `Iterator`라는 이름의 트레잇을
구현합니다. 트레잇의 정의는 아래처럼 생겼습니다:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

이 정의가 몇몇 새로운 문법을 사용하는 것을 주목하세요: `type Item`과
`Self::Item`은 이 트레잇에 대한 *연관 타입 (associated type)*을 정의합니다.
연관 타입에 대해서는 19장에서 더 깊이 이야기하겠습니다. 지금 여러분이 알아야 할
것은 `Iterator` 트레잇을 구현하려면 `Item` 타입도 함께 정의되어야 하며,
이 `Item` 타입이 `next` 메소드의 반환 타입으로 사용된다는 것을 이 코드가
알려준다는 것입니다. 바꿔 말하면, `Item` 타입은 반복자로부터 반환되는 타입이
되겠습니다.

`Iterator` 트레잇은 구현하려는 이에게 딱 하나의 메소드 정의를 요구합니다: 바로
`next` 메소드인데, 이 메소드는 `Some`으로 감싼 반복자의 한 아이템을 반환하고,
반복자가 종료될 때는 `None` 을 반환합니다.

반복자의 `next` 메소드를 직접 호출할 수 있습니다; Listing 13-12는
벡터로부터 생성된 반복자에 대하여 `next`를 반복적으로 호출했을 때 어떤
값들이 반환되는지 보여줍니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

<span class="caption">Listing 13-12: 반복자의 `next` 메서드
호출하기</span>

`v1_iter`를 가변으로 만들 필요가 있음을 주의하세요: 반복자에 대한 `next`
메소드 호출은 반복자 내부의 상태를 변경하여 반복자가 현재 시퀀스의 어디에
있는지 추적합니다. 바꿔 말하면, 이 코드는 반복자를 *소비합니다*, 혹은 다
써 버립니다. `next` 에 대한 각 호출은 반복자로부터 하나의 항목을 소비합니다. 
`for` 루프를 사용할 때는 `v1_iter` 를 변경할 수 있도록 만들 필요가 없는데, 
루프가 `v1_iter` 의 소유권을 갖고 내부적으로 변경 가능하도록 만들기 때문입니다.

또한 `next` 호출로 얻어온 값들은 벡터 내의 값들에 대한 불변
참조자라는 점도 주의하세요. `iter` 메소드는 불변 참조자에 대한
반복자를 만듭니다. 만약 `v1`의 소유권을 얻어서 반환하도록 하고
싶다면, `iter` 대신 `into_iter` 를 호출할 수 있습니다. 비슷하게,
가변 참조자에 대한 반복자가 필요하면, `iter` 대신 `iter_mut`을
호출할 수 있습니다.

### 반복자를 소비하는 메소드들

`Iterator` 트레잇에는 표준 라이브러리에서 기본 구현을 제공하는
여러가지 메소드들이 있습니다; 이 메소드들은 표준 라이브러리 API
문서의 `Iterator` 트레잇에 대한 부분을 살펴보면 찾을 수 있습니다.
이 메소드들 중 일부는 정의 부분에서 `next` 메소드를 호출하는데,
이것이 `Iterator` 트레잇을 구현할 때 `next` 메소드를 구현해야만
하는 이유 입니다.

`next`를 호출하는 메소드들을 *소비 어댑터 (consuming adaptor)*라고 하는데,
호출하면 반복자를 써버리기 때문에 그렇습니다. 한 가지 예로 `sum` 메소드가 있는데,
이는 반복자의 소유권을 가져온 다음 반복적으로 `next`를 호출하는 방식으로
순회하며, 따라서 반복자를 소비합니다. 전체를 순회하면서 현재의 합계값에
각 아이템을 더하고 순회가 완료되면 합계를 반환합니다. Listing 13-13은
`sum` 메소드 사용 방식을 보여주는 테스트입니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

<span class="caption">Listing 13-13: `sum` 메소드를 호출하여 반복자의
모든 아이템에 대한 합계 구하기</span>

`sum`은 반복자를 소유하여 호출하므로, `sum`을 호출한 이후에는 `v1_iter`의
사용이 허용되지 않습니다.

### 다른 반복자를 생성하는 메소드들

*반복자 어댑터 (iterator adaptor)*는 `Iterator` 트레잇에 정의된 메소드로
반복자를 소비하지 않습니다. 대신 원본 반복자의 어떤 측면을 바꿔서 다른
반복자로 제공합니다.

Listing 13-14 shows an example of calling the iterator adaptor method `map`,
which takes a closure to call on each item as the items are iterated through.
The `map` method returns a new iterator that produces the modified items. The
closure here creates a new iterator in which each item from the vector will be
incremented by 1:
Listing 13-14는 반복자 어댑터 메소드인 `map`을 호출하는 예를 보여주는데,
클로저를 인자로 받아서 각 아이템에 대해 호출하여 아이템 전체를 순회합니다.
`map` 메소드는 수정된 아이템들을 생성하는 새로운 반복자를 반환합니다.
여기서의 클로저는 벡터의 각 항목에서 1이 증가된 새로운 반복자를
만듭니다:

<span class="filename">Filename: src/main.rs</span>

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

<span class="caption">Listing 13-14: 반복자 어댑터 `map`을 호출하여
새로운 반복자 생성하기</span>

하지만 이 코드는 다음과 같은 경고를 발생시킵니다:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Listing 13-14의 코드는 아무것도 하지 않습니다; 넘겨진 클로저는 결코 호출되지
않습니다. 위 경고는 이유가 무엇인지 상기시켜 줍니다: 반복자 어댑터는 게으르고,
반복자를 여기서 소비할 필요가 있다는 것을요.

이 경고를 수정하고 반복자를 소비하기 위해서 `collect` 메소드를 사용할
것인데, 12장의 Listing 12-1에서 `env::args`와 함께 사용했었지요. 이
메소드는 반복자를 소비하고 결과값을 모아서 콜렉션 데이터 타입으로 만들어
줍니다.

Listing 13-15에서는 `map`을 벡터에 대해 호출하여 얻은 반복자를 순회하면서
결과를 모읍니다. 이 벡터는 원본 벡터로부터 1씩 증가된 아이템들을 담고 있는
상태가 될 것 입니다.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

<span class="caption">Listing 13-15: `map`을 호출하여 새로운 반복자를
생성한 다음 `collect` 메소드를 호출하여 이 반복자를 소비하고 새로운 벡터
생성하기</span>

`map`은 클로저를 인자로 받기 때문에, 각 항목에 대해 수행하고자 하는 어떤
연산이라도 기술할 수 있습니다. 이는 `Iterator` 트레잇이 제공하는 반복자 행위를
재사용하면서 클로저가 일부의 동작을 커스터마이징할 수 있게 해주는 방법을 보여주는
훌륭한 예입니다.

반복자 어댑터의 호출을 연결시키면 복잡한 동작을 읽기 쉬운 방식으로 수행할
수 있습니다. 하지만 모든 반복자는 게으르므로, 반복자 어댑터를 호출한 결과를
얻기 위해서는 소비 어댑터 중 하나를 호출해야만 합니다.

### 환경을 캡쳐하는 클로저 사용하기

많은 반복자 어댑터들은 클로저를 인자로 사용하고, 보통 클로저의
인자에 명시되는 클로저는 자신의 환경을 캡처하는 클로저일
것입니다.

예를 들기 위해 클로저 인자를 사용하는 `filter` 메소드를 사용해보겠습니다.
이 클로저는 반복자로부터 아이템을 받아서 `bool`을 반환합니다. 만일 클로저가
`true`를 반환하면, 그 값은 `filter`에 의해 생성된 반복자에 포함되게 됩니다.
클로저가 `false`를 반환하면 해당 값은 포함시키지 않습니다.

리스트 13-13에서는 환경으로부터 `shoe_size`를 캡처하는 클로저를 가지고
`filter`를 사용하여 `Shoe` 구조체 인스턴스들의 콜렉션을 순회합니다.
이는 명시된 크기의 신발만 반환해줄 것입니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

<span class="caption">Listing 13-16: `shoe_size`를 캡처하는 클로저로
`filter` 메소드 사용하기</span>

`shoes_in_size` 함수는 매개변수로 신발들의 벡터에 대한 소유권과 신발
크기를 받습니다. 이 함수는 지정된 크기의 신발들만을 담고 있는 벡터를
반환합니다.

`shoes_in_size`의 본문에서는 `into_iter`를 호출하여 이 벡터의 소유권을
갖는 반복자를 생성합니다. 그다음 `filter`를 호출하여 앞의 반복자를
새로운 반복자로 바꾸는데, 새로운 반복자에는 클로저가 `true`를 반환하는
원소들만 담겨있게 됩니다.

클로저는 환경에서 `shoe_size` 매개변수를 캡처하고 각 신발의
크기와 값을 비교하여 지정된 크기의 신발만 유지하도록 합니다.
마지막으로, `collect`를 호출하면 적용된 반복자에 의해 반환된 값을
벡터로 모으고, 이 벡터가 함수에 의해 반환됩니다.

이 테스트는 `shoes_in_size`를 호출했을 때 지정된 값과 동일한 크기인
신발들만 돌려받는다는 것을 보여 줍니다.
