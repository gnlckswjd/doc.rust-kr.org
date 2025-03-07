## 트레잇 객체를 사용하여 다른 타입의 값 허용하기

8장에서 벡터의 제약사항 중 하나는 딱 하나의 타입에 대한 원소만 보관할 수
있다는 것임을 언급했습니다. Listing 8-10에서 정수, 부동소수점, 그리고 문자를
보관하기 위한 variant들을 가지고 있는 `SpreadsheetCell` 열거형을 정의하는
해결 방안을 만들었습니다. 즉 각 칸마다 다른 타입의 데이터를 저장할 수 있으면서도
여전히 그 칸들의 한 묶음을 대표하는 벡터를 가질 수 있었습니다. 이는 교환 가능한
아이템들이 코드를 컴파일할 때 알 수 있는 고정된 타입의 집합인 경우 완벽한
해결책입니다.

하지만, 때로는 우리의 라이브러리 사용자가 특정 상황에서 유효한 타입의
집합을 확장할 수 있도록 하길 원할 때가 있습니다. 이를 어떻게 달성할 수 있는지
보이기 위해, 아이템들의 리스트에 대해 반복하고 각 아이템에 대해 `draw` 메소드를
호출하여 이를 화면에 그리는 예제 그래픽 사용자 인터페이스(GUI) 도구를 만들어
보겠습니다 - GUI 도구들에게 있어서는 흔한 방식이죠. 우리가 만들 라이브러리 크레이트는
`gui`라고 호명되고 GUI 라이브러리 구조를 포괄합니다. 이 크레이트는 사용자들이
사용할 수 있는 몇 가지 타입들, `Button`이나 `TextField` 들을 포함하게
될 수 있습니다. 또한 `gui` 사용자들은 그릴 수 있는 자신만의 타입을 만들고자
할 것입니다: 일례로, 어떤 프로그래머는 `Image`를 추가할지도, 또다른
누군가는 `SelectBox`를 추가할지도 모릅니다.

이번 예제에서는 완전한 GUI 라이브러리를 구현하지는 않겠지만 각 요소들이
어떻게 결합되는지 보여주고자 합니다. 라이브러리를 작성하는 시점에서는
다른 프로그래머들이 만들고자 하는 모든 타입들을 알 수 없죠. 하지만
`gui`가 다양한 타입들의 많은 값을 추적해야 하고, `draw` 메소드가 각각의
다양한 타입의 값들에 대해 호출되어야 한다는 것은 알고 있습니다. `draw`
메소드를 호출했을 때 벌어지는 일에 대해서 정확히 알 필요는 없고, 그저
그 값에 호출할 수 있는 해당 메소드가 있음을 알면 됩니다.

상속이 있는 언어로 이 작업을 하기 위해서는 `draw` 라는 이름의 메소드를
갖고 있는 `Component` 라는 클래스를 정의할 수 있습니다. 다른 클래스들, 이를테면
`Button`, `Image`, 그리고 `SelectBox` 같은 것들은 `Component`를 상속받고
따라서 `draw` 메소드를 물려받게 됩니다. 이들은 각각 `draw` 메소드를 오버라이딩하여
그들의 고유 동작을 정의할 수 있으나, 프레임워크는 모든 타입을 마치 `Component`인
것처럼 다룰 수 있고 `draw`를 호출할 수 있습니다. 하지만 러스트에는 상속이 없는
관계로, 사용자들이 새로운 타입을 정의하고 확장할 수 있도록 `gui` 라이브러리를
구조화하는 다른 방법이 필요합니다 .

### 공통된 동작을 위한 트레잇 정의하기

`gui`가 갖기를 원하는 동작을 구현하기 위해, `draw`라는 이름의 메소드가
하나 있는 `Draw`라는 이름의 트레잇을 정의할 것입니다. 그러면 *트레잇 객체
(trait object)* 를 담는 벡터를 정의할 수 있습니다. 트레잇 객체는 특정
트레잇을 구현한 타입의 인스턴스와 런타임에 해당 타입의 트레잇 메소드를
조회하는데 사용되는 테이블 모두를 가리킵니다. `&` 참조자나 `Box<T>`
스마트 포인터 같은 포인터 종류로 지정한 다음 `dyn` 키워드를 붙이고,
그 뒤에 관련된 트레잇을 특정하면 트레잇 객체를 생성할 수 있습니다.
(트레잇 객체에 포인터를 사용해야 하는 이유는 19장의
[“동적인 크기의 타입과 Sized”][dynamically-sized]<!-- ignore --> 절에서
설명하겠습니다.) 제네릭 타입이나 구체 타입 대신 트레잇 객체를 사용할 수 있습니다.
트레잇 객체를 사용하는 곳이 어디든, 러스트의 타입 시스템은 컴파일 타임에 해당
컨텍스트에서 사용된 모든 값이 트레잇 객체의 트레잇을 구현할 것을 보장합니다.
결론적으로 컴파일 타임에 모든 가능한 타입을 알 필요가 없습니다.

앞서 언급했듯 러스트에서는 다른 언어의 객체와 구분하기 위해
구조체와 열거형을 “객체”라고 부르는 것을 자제합니다. 구조체나
열거형에서는 구조체 필드의 데이터와 `impl` 블록의 동작이 분리되는 반면,
다른 언어에서는 데이터와 동작이 하나의 개념으로 결합된 것을 객체라고
명명하는 경우가 많으니까요. 트레잇 객체들은 데이터와 동작을 결합한다는
의미에서 다른 언어의 객체와 더 *비슷합니다*. 하지만 트레잇 객체는
트레잇 객체에 데이터를 추가 할 수 없다는 점에서 전통적인 객체와
다릅니다. 트레잇 객체는 다른 언어들의 객체만큼 범용적으로 유용하지는
않습니다: 그들의 명확한 목적은 공통된 동작에 대한 추상화를 가능하도록
하는 것이죠.

Listing 17-3은 `draw`라는 이름의 메소드를 갖는 `Draw `라는 트레잇을 정의하는
방법을 보여줍니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-03/src/lib.rs}}
```

<span class="caption">Listing 17-3: `Draw` 트레잇의 정의</span>

이 문법은 10장에 있는 트레잇을 정의하는 방법에서 다뤘으니 익숙하실 겁니다.
다음에 새로운 문법이 등장합니다: Listing 17-4는 `components` 라는
벡터를 보유하고 있는 `Screen`이라는 구조체를 정의합니다. `Box<dyn Draw>`
타입의 벡터인데, 이것이 트레잇 객체입니다; 이것은 `Draw` 트레잇을 구현한
`Box` 안의 모든 타입에 대한 대역입니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-04/src/lib.rs:here}}
```

<span class="caption">Listing 17-4: `Draw` 트레잇을 구현하는
트레잇 객체들의 벡터 `components`를 필드로 가지고 있는 `Screen`
구조체의 정의</span>

`Screen` 구조체에서는 Listing 17-5와 같이 `components`의 각 아이템마다
`draw` 메소드를 호출하는 `run` 메소드를 정의합니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-05/src/lib.rs:here}}
```

<span class="caption">Listing 17-5: 각 컴포넌트에 대해 `draw` 메소드를
호출하는 `Screen`의 `run` 메소드</span>

이는 트레잇 바운드가 있는 제네릭 타입 매개변수를 사용하는 구조체를
정의하는 것과는 다르게 작동합니다. 제네릭 타입 매개변수는 한 번에
하나의 구체 타입으로만 대입될 수 있는 반면, 트레잇 객체를 사용하면
런타임에 트레잇 객체에 대해 여러 구체 타입을 채워넣을 수 있습니다.
예를 들면, Listing 17-6처럼 제네릭 타입과 트레잇 바운드를 사용하여
`Screen` 구조체를 정의할 수도 있을 겁니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-06/src/lib.rs:here}}
```

<span class="caption">Listing 17-6: 제네릭과 트레잇 바운드를 사용한
`Screen` 구조체와 `run` 메소드의 대체 구현</span>

이렇게하면 전부 `Button` 타입이거나 전부 `TextField` 타입인 컴포넌트의
리스트를 가지는 `Screen` 인스턴스로 제한됩니다. 동일 유형의 콜렉션만 사용한다면
제네릭과 트레잇 바운드를 사용하는 것이 바람직한데, 왜냐하면 그 정의들은 컴파일 타임에
단형성화 (monomorphize) 되어 구체 타입으로 사용되기 때문입니다.

반면 트레잇 객체를 사용하는 메소드를 이용할 경우, 하나의 `Screen` 인스턴스가
`Box<Button>`는 물론 `Box<TextField>`도 담을 수 있는 `Vec<T>`를 보유할
수 있습니다. 이것이 어떻게 작동하는지 살펴보고 런타임 성능에 미치는 영향에 대해
설명하겠습니다.

### 트레잇 구현하기

이제 `Draw` 트레잇을 구현하는 몇가지 타입을 추가하겠습니다. `Button` 타입을
제공해 보겠습니다. 다시금 말하지만, 실제 GUI 라이브러리를 구현하는 것은
이 책의 범위를 벗어나므로, `draw`에는 별다른 구현을 하지 않을 겁니다.
구현이 어떻게 생겼을지 상상해보자면, `Button` 구조체에는 Listing 17-7에서
보는 바와 같이 `width`, `height` 그리고 `label` 항목들이 있을 것입니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-07/src/lib.rs:here}}
```

<span class="caption">Listing 17-7: `Draw` 트레잇을 구현하는
`Button` 구조체</span>

`Button`의 `width`, `height` 및 `label` 필드는 다른 컴포넌트의
필드와는 다를 것입니다; 예를 들어 `TextField` 타입은 이 필드들에 추가로
`placeholder` 필드를 가질 수 있습니다. 화면에 그리고자 하는 각각의
타입은 `Draw` 트레잇을 구현하겠지만 해당 타입을 그리는 방법을 정의하기
위하여 `draw` 메소드 내에 서로 다른 코드를 사용하게 될 것이고, `Button`도
여기서 그렇게 하고 있습니다 (앞서 언급한 것처럼 실질적인 GUI 코드는 없지만요).
예를 들어, `Button` 타입은 추가적인 `impl` 블록에 사용자가 버튼을 클릭했을
때 어떤 일이 벌어질지와 관련된 메소드들을 포함할 수 있습니다. 이런 종류의
메소드는 `TextField`와 같은 타입에는 적용할 수 없죠.

우리의 라이브러리를 사용하는 누군가가 `width`, `height` 및 `options`
필드가 있는 `SelectBox` 구조체를 구현하기로 결정했다면, Listing 17-8과 같이
`SelectBox` 타입에도 `Draw` 트레잇을 구현합니다:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-08/src/main.rs:here}}
```

<span class="caption">Listing 17-8: `gui`를 사용하고 `Draw` 트레잇을
`SelectBox` 구조체에 구현한 또 다른 크레이트</span>

이제 라이브러리 사용자는 `main` 함수를 작성하여 `Screen` 인스턴스를
만들 수 있습니다. `Screen` 인스턴스에는 `SelectBox`와 `Button`를
`Box<T>` 안에 넣어 트레잇 객체가 되도록 함으로써 이들을 추가할 수 있습니다.
그러면 `Screen` 인스턴스 상의 `run` 메소드를 호출할 수 있는데, 이는 각 컴포넌트들에
대해 `draw`를 호출할 것입니다. Listing 17-9는 이러한 구현을 보여줍니다:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-09/src/main.rs:here}}
```

<span class="caption">Listing 17-9: 트레잇 객체를 사용하여 동일한 트레잇을
구현하는 서로 다른 타입들의 값 저장하기</span>

이 라이브러리를 작성할 때는 누군가 `SelectBox` 타입을 추가할 수도
있다는 것을 몰랐지만, 우리의 `Screen` 구현체는 새로운 타입에 대해서도
동작하고 이를 그려낼수 있는데, 그 이유는 `SelectBox`가 `Draw` 타입을 구현했기
때문이고, 이는 `draw` 메소드가 구현되어 있음을 의미합니다.

이러한 개념, 즉 값의 구체적인 타입이 아닌 값이 응답하는 메시지만
고려하는 개념은 동적 타입 언어의 *오리 타이핑 (duck typing)* 이란
개념과 유사합니다: 만약 오리처럼 걷고 오리처럼 꽥꽥거리면, 그것은
오리임에 틀림없습니다! Listing 17-5에 있는 `Screen`에 구현된 `run`을 보면,
`run`은 각 컴포넌트가 어떤 구체적 타입인지 알 필요가 없습니다.
이 함수는 컴포넌트가 `Button`의 인스턴스인지 혹은 `SelectBox`의
인스턴스인지 검사하지 않고 그저 각 컴포넌트의 `draw` 메소드를 호출할
뿐입니다. `components` 벡터에 담기는 값의 타입을 `Box<dyn Draw>`로
특정하는 것으로 `draw` 메소드를 호출할 수 있는 값을 필요로 하는
`Screen`을 정의했습니다.

트레잇 객체와 러스트의 타입 시스템을 사용하여 오리 타이핑을 사용하는 코드와
유사한 코드를 작성할 때의 장점은 런타임에 어떤 값이 특정한 메소드를 구현했는지
여부를 검사하거나 값이 메소드를 구현하지 않았는데 어쨌든 호출한 경우 에러가
발생할 것을 걱정할 필요가 없다는 겁니다. 트레잇 객체가 요구하는 트레잇을
해당 값이 구현하지 않았다면 러스트는 컴파일하지 않을 겁니다.

예를 들어, Listing 17-10은 `String`을 컴포넌트로 사용하여 `Screen`을
생성하는 시도를 하면 어떤 일이 벌어지는지 보여줍니다:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-10/src/main.rs}}
```

<span class="caption">Listing 17-10: 트레잇 객체의 트레잇을 구현하지 않은
타입의 사용 시도</span>

`String`이 `Draw` 트레잇을 구현하지 않기 때문에 아래와 같은 에러를 얻게 됩니다:

```console
{{#include ../listings/ch17-oop/listing-17-10/output.txt}}
```

이 에러는 인자로 넘길 의도가 없었던 무언가를 `Screen`에게 넘기고 있으므로 이를
다른 타입으로 교체하던가, 아니면 `String`에 대해 `Draw`를 구현하여 `Screen`이
이것에 대해 `draw`를 호출할 수 있도록 해야한다는 것을 알려줍니다.

### 트레잇 객체는 동적 디스패치를 수행합니다

10장의 [“제네릭을 사용한 코드의 성능”][performance-of-code-using-generics]<!-- ignore -->절에서
제네릭에 트레잇 바운드를 사용했을 때 컴파일러에 의해 수행되는 단형성화
프로세스의 실행에 대한 논의를 상기해보세요: 컴파일러는 제네릭 타입 매개변수
대신 사용하는 각 구체 타입에 대한 함수와 메소드의 비제네릭 구현체를
생성합니다. 단형성화로부터 야기된 코드는 *정적 디스패치 (static dispatch)* 를
수행하는데, 이는 호출하고자 하는 메소드가 어떤 것인지 컴파일러가
컴파일 시점에 알고 있는 것입니다. 이는 *동적 디스패치 (dynamic dispatch)*
와 반대되는 개념으로, 동적 디스패치는 컴파일러가 호출하는 메소드를
컴파일 시점에 알 수 없을 경우 수행됩니다. 동적 디스패치의 경우,
컴파일러는 런타임에 어떤 메소드가 호출되는지 알아내는 코드를
생성합니다.

트레잇 객체를 사용할 때 러스트는 동적 디스패치를 이용해야 합니다. 컴파일러는
트레잇 객체를 사용중인 코드와 함께 사용될 수 있는 모든 타입을 알지 못하므로,
어떤 타입에 구현된 어떤 메소드가 호출될지 알지 못합니다. 대신 런타임에서, 러스트는
트레잇 객체 내에 존재하는 포인터를 사용하여 어떤 메소드가 호출될지 알아냅니다.
이러한 조회는 정적 디스패치 시에는 발생하지 않을 런타임 비용을 만들어냅니다.
동적 디스패치는 또한 컴파일러가 메소드의 코드를 인라인 (inline) 화하는 선택을
막아버리는데, 이것이 결과적으로 몇가지 최적화를 수행하지 못하게 합니다. 하지만,
Listing 17-5와 같은 코드를 작성하고 Listing 17-9과 같은 지원을 가능하게 하는
추가적인 유연성을 얻었으므로, 여기에는 고려할 기회비용이 있다고 하겠습니다.

[performance-of-code-using-generics]:
ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait
