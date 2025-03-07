## 트레잇으로 공통된 동작을 정의하기

*트레잇 (trait)* 은 특정 타입이 가지고 있고 다른 타입과 공유할 수 있는
**기능**을 정의합니다. 트레잇을 사용하면 공통된 기능을 추상적으로
정의할 수 있습니다. *트레잇 바운드(trait bound)*를 이용하면 어떤 제네릭
타입 자리에 특정한 동작을 하는 타입이 올 수 있음을 명시할 수 있습니다.

> Note: 트레잇은 다른 언어에서 *인터페이스(interface)* 라고 부르는 기능과
> 유사하지만, 약간 차이가 있습니다.

### 트레잇 정의하기

어떤 타입의 동작은 우리가 해당 타입에서 호출할 수 있는 메소드로 구성됩니다.
만약 다양한 타입에서 공통된 메소드를 호출할 수 있다면, 우린 이 타입들이
동일한 동작을 공유한다고 표현할 수 있을 겁니다. 트레잇 정의는 메소드 시그니처를
그룹화하여 특정 목적을 달성하는 데 필요한 일련의 동작을 정의하는 것입니다.

예를 들어 봅시다.
다양한 종류 및 분량의 텍스트를 갖는 여러 가지 구조체가 있다고 치죠.
`NewsArticle` 구조체는 특정 지역에서 등록된 뉴스 기사를 저장하고,
`Tweet` 구조체는 최대 280자의 콘텐츠와 메타데이터(해당 트윗이 새 트윗인지,
리트윗인지, 다른 트윗의 대답인지)를 저장합니다.

`NewsArticle` 이나 `Tweet` 인스턴스에 저장된 데이터를 종합해 보여주는
종합 미디어 라이브러리 크레이트 `aggregator`를 만든다고 가정합시다.
이를 위해서는 각 타입의 요약 정보를 얻어와야 하는데, 이는 인스턴스에
`summarize` 메소드를 호출하여 가져오고자 합니다. Listing 10-12는
이 동작을 `Summary` 트레잇 정의로 표현합니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

<span class="caption">Listing 10-12: `summarize` 메소드가 제공하는
'요약' 동작으로 구성된 `Summary` 트레잇</span>

`trait` 키워드 다음 트레잇의 이름 `Summary` 를 작성해
트레잇을 선언했습니다. 또한 트레잇을 `pub`로 선언하여 몇몇
예제에서 볼 것처럼 이 크레이트에 의존하는 다른 크레이트들이
이 트레잇을 사용할 수 있도록 하였습니다. 중괄호 안에는 이 트레잇을
구현할 타입의 동작을 묘사하는 메소드 시그니처를 선언했는데,
위의 경우는 `fn summarize(&self) -> String` 입니다.

메소드 시그니처 뒤에는 중괄호로 시작하여
메소드를 구현하는 대신 세미콜론을 집어넣었습니다.
이 트레잇을 구현할 각각의 타입이 직접 메소드에 맞는 동작을 제공하도록 하기 위함입니다.
이제 `Summary` 트레잇을 갖는 모든 타입은 정확히 이와 같은 시그니처의
`summarize` 메소드를 가지고 있음을 러스트 컴파일러가 보장해줄 겁니다.

트레잇은 본문에 여러 메소드를 가질 수 있습니다.
메소드 서명은 한 줄에 하나씩 나열되며, 각 줄은 세미콜론으로 끝납니다.

### 특정 타입에 트레잇 구현하기

`Summary` 트레잇의 메소드 시그니처를 원하는 대로 정의했으니,
우리의 종합 미디어 내 각 타입에 `Summary` 트레잇을 구현해봅시다.
Listing 10-13은 헤드라인, 저자, 지역 정보를 사용하여 `summarize`의 반환 값을
만드는 `Summary` 트레잇을 `NewsArticle` 구조체에 구현한 모습입니다.
`Tweet` 구조체에는 트윗 내용이 이미 280자로 제한되어있음을 가정하고,
사용자명과 해당 트윗의 전체 텍스트를 가져오도록 `summarize` 를
정의했습니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

<span class="caption">Listing 10-13: `NewsArticle`과 `Tweet` 타입에
`Summary` 트레잇 구현</span>

어떤 타입에 대한 트레잇을 구현하는 것은 평범한 메소드를
구현하는 것과 유사합니다. 다른 점은 `impl` 뒤에 구현하고자
하는 트레잇 이름을 적고, 그다음 `for` 키워드와 트레잇을
구현할 타입명을 명시한다는 점입니다. `impl` 블록 내에는
트레잇 정의에서 정의된 메소드 시그니처를 집어넣되,
세미콜론 대신 중괄호를 사용하여 메소드 본문에 우리가
원하는 특정한 동작을 채워 넣습니다.

라이브러리가 `NewsArticle`과 `Tweet`에 대한 `Summary` 트레잇을 구현했으니,
크레이트 사용자는 `NewsArticle`과 `Tweet` 인스턴스에 대하여 보통의 메소드를
호출하는 것과 같은 방식으로 트레잇 메소드를 호출할 수 있습니다. 유일한 차이점은
크레이트 사용자가 타입 뿐만 아니라 트레잇도 스코프로 가져와야 한다는 점입니다.
아래에 바이너리 크레이트가 `aggregator` 라이브러리 크레이트를 사용하는 방법에
대한 예제가 있습니다:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

이 코드는 `1 new tweet: horse_ebooks: of course, as you probably already
know, people`를 출력합니다.

`aggregator` 크레이트에 의존적인 다른 크레이트들 또한 `Summary` 트레잇을 스코프로
가져와서 자신들의 타입에 대해 `Summary`를 구현할 수 있습니다. 트레잇 구현에는 한 가지
제약사항이 있는데, 이는 트레잇이나 트레잇을 구현할 타입 둘 중 하나는 반드시 우리 크레이트
내의 것이어야 해당 타입에 대한 트레잇을 구현할 수 있다는 점입니다. 예를 들어, 우리가
만든 `aggregator` 크레이트 기능의 일부로서 `Tweet` 타입에 표준 라이브러리 트레잇인
`Display` 등을 구현할 수 있습니다. `Tweet` 타입이 우리가 만든 `aggregator` 크레이트의
타입이기 때문입니다. 또한 `aggregator` 크레이트 내에서 `Vec<T>` 타입에 `Summary`
트레잇을 구현할 수도 있습니다. 마찬가지로 `Summary` 트레잇이 우리가 만든 `aggregator`
크레이트의 트레잇이기 때문입니다.

하지만 외부 타입에 외부 트레잇을 구현할 수는 없습니다. 예를 들어, 우리가 만든
`aggregator` 크레이트에서는 `Vec<T>`에 대하여 `Display` 트레잇을 구현할 수 없습니다.
`Vec<T>`, `Display` 둘다 우리가 만든 크레이트가 아닌 표준 라이브러리에 정의되어
있기 때문입니다. 이 제약은 프로그램의 특성 중 하나인 *일관성(coherence)*,
보다 자세히는 *고아 규칙(orphan rule)* 에서 나옵니다 (부모 타입이 존재하지
않기 때문에 고아 규칙이라고 부릅니다). 이 규칙으로 인해 다른 사람의 코드는
여러분의 코드를 망가뜨릴 수 없으며 반대의 경우도 마찬가지입니다. 이 규칙이
없다면 두 크레이트가 동일한 타입에 동일한 트레잇을 구현할 수 있게 되고, 러스트는
어떤 구현체를 이용해야 할지 알 수 없습니다.

### 기본 구현

타입에 트레잇을 구현할 때마다 모든 메소드를 구현할 필요는 없도록
트레잇의 메소드에 기본 동작을 제공할 수도 있습니다.
이러면 특정한 타입에 트레잇을 구현할 때 기본 동작을 유지할지
오버라이드(override)할지 선택할 수 있습니다.

Listing 10-14는 Listing 10-12에서 `Summary` 트레잇에
메소드 시그니처만 정의했던 것과는 달리 `summarize` 메소드에
기본 문자열을 명시하였습니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

<span class="caption">Listing 10-14: `summarize` 메소드의 기본 구현을
포함하는 `Summary` 트레잇 정의하기</span>

`NewsArticle` 인스턴스에 기본 구현을 사용하려면
`impl Summary for NewsArticle {}` 처럼 비어있는 `impl` 블록을 명시합니다.

`impl` 블록을 비워도 `NewsArticle` 인스턴스에서 `summarize` 메소드를 호출할 수 있습니다.
`NewsArticle` 에 `summarize` 메소드를 직접적으로 정의하지는 않았지만,
`NewsArticle` 은 `Summary` 트레잇을 구현하도록 지정되어 있으며,
`Summary` 트레잇은 `summarize` 메소드의 기본 구현을 제공하기 때문입니다:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

이 코드는 `New article available! (Read more...)`를 출력합니다.

기본 구현을 생성한다고 해서 Listing 10-13 코드의 `Tweet`
의 `Summary` 구현을 변경할 필요는 없습니다. 기본 구현을
오버라이딩하는 문법과, 기본 구현이 없는 트레잇 메소드를
구현하는 문법은 동일하기 때문입니다.

기본 구현 내에서 트레잇 내 다른 메소드를 호출할 수도 있습니다.
호출할 다른 메소드가 기본 구현을 제공하지 않는 메소드여도 상관없습니다.
이 점을 이용해, 트레잇의 일부 구현만 명시하는 것만으로도 트레잇의 다양한
기능을 제공받을 수 있습니다. 예시로 알아봅시다.
`Summary` 트레잇에 `summarize_author` 메소드를 추가하고,
`summarize` 메소드의 기본 구현 내에서 `summarize_author` 메소드를 호출하도록
만들어보았습니다:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

이 `Summary`를 어떤 타입에 구현할 때는 `summarize_author` 만
정의하면 됩니다:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

`summarize_author` 를 정의하고 나면 `Tweet` 인스턴스에서 `summarize` 를 호출할 수 있습니다.
이러면 `summarize` 기본 구현이 우리가 정의한 `summarize_author` 메소드를 호출할 겁니다.
우린 `summarize_author` 만 구현하고 추가적인 코드를 전혀 작성하지 않았지만,
`Summary` 트레잇은 우리에게 `summarize` 메소드의 기능도 제공해주는 것을
알 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

이 코드는 `1 new tweet: (Read more from @horse_ebooks...)` 를 출력합니다.

단, 어떤 메소드를 오버라이딩하는 구현을 하면 해당 메소드의 기본 구현을
호출할 수는 없다는 점을 기억해두세요.

### 매개변수로서의 트레잇

트레잇을 정의하고 구현하는 방법을 알아보았으니, 트레잇을 이용하여 어떤 함수가
다양한 타입으로 작동하게 만드는 법을 알아봅시다. Listing 10-13에서 `NewsArticle`,
`Tweet` 타입에 구현한 `Summary` 트레잇을 사용하여, `item` 매개변수의
`summarize` 메소드를 호출하는 `notify` 함수를 정의하겠습니다. 이때 `item`
매개변수의 타입은 `Summary` 트레잇을 구현하는 어떤 타입입니다. 이렇게 하기
위해서는 아래와 같이 `impl Trait` 문법을 사용합니다:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

`item` 매개변수의 구체적 타입을 명시하는 대신 `impl` 키워드와 트레잇 이름을 명시했습니다.
이 매개변수는 명시된 트레잇을 구현하는 타입이라면 어떤 타입이건 전달받을 수 있습니다.
`notify` 본문 내에서는 `item` 에 `summarize` 등
`Summary` 트레잇의 모든 메소드를 호출할 수 있습니다.
`notify` 는 `NewsArticle` 인스턴스로도, `Tweet` 인스턴스로도 호출할 수 있습니다.
만약 `notify` 함수를 `Summary` 트레잇을 구현하지 않는
`String`, `i32` 등의 타입으로 호출하는 코드를 작성한다면 컴파일 에러가 발생합니다.

<!-- Old headings. Do not remove or links may break. -->
<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### 트레잇 바운드 문법

`impl Trait` 문법은 간단하지만, 이는 *트레잇 바운드 (trait bound)*로 알려진, 좀 더
길다란 형식의 문법 설탕(syntax sugar)입니다; 트레잇 바운드는 다음과 같이 생겼습니다:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

앞서 본 예시와 동일한 코드지만, 더 장황합니다.
트레잇 바운드는 꺾쇠괄호 안의 제네릭 타입 매개변수 선언에 붙은 콜론(`:`) 뒤에
위치합니다.

`impl Trait` 문법은 편리하고 단순한 상황에는 코드를 더 간결하게 만들어 주는
반면, 트레잇 바운드 문법은 더 복잡한 상황을 표현할 수 있습니다. 예를 들어,
`Summary` 를 구현하는 두 매개변수를 전달받는 함수를 구현할 때, `impl Trait`
문법으로 표현하면 다음과 같은 모양이 됩니다:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

`item1` 과 `item2` 가 (둘 다 `Summary`를 구현하는 타입이되) 서로 다른
타입이어도 상관없다면 `impl Trait` 문법 사용도 적절합니다. 하지만 만약
두 매개변수가 같은 타입으로 강제되어야 한다면, 이는 아래와 같이 트레잇
바운드를 사용해야 합니다:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

`item1` 및 `item2` 매개 변수의 타입으로 지정된 제네릭 타입 `T`는
함수를 호출할 때 `item1`, `item2` 인자 값의 구체적인 타입이
반드시 동일하도록 제한합니다.

#### `+` 구문으로 트레잇 바운드를 여럿 지정하기

트레잇 바운드는 여러 개 지정할 수도 있습니다. `notify` 에서 `item`의 `summarize`
메소드뿐만 아니라 출력 포매팅까지 사용하고 싶다고 가정해 봅시다: `notify` 정의에서
`item` 이 `Display`, `Summary`를 모두 구현해야하도록 지정해야합니다. `+` 문법을
사용하면 트레잇을 여러 개 지정할 수 있습니다:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` 구문은 제네릭 타입의 트레잇 바운드에도 사용할 수 있습니다:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

두 트레잇 바운드가 지정됐으니, `notify` 본문에서는 `item` 의 `summarize` 메소드를
호출할 수도 있고 `item` 을 `{}` 으로 포매팅할 수도 있습니다.

#### `where` 조항으로 트레잇 바운드 정리하기

트레잇 바운드가 너무 많아지면 문제가 생깁니다.
제네릭마다 트레잇 바운드를 갖게 되면, 여러 제네릭 타입 매개변수를 사용하는 함수는
함수명과 매개변수 사이에 너무 많은 트레잇 바운드 정보를 담게 될 수 있습니다.
이는 가독성을 해치기 때문에, 러스트는 트레잇 바운드를 함수 시그니처 뒤의
`where` 조항에 명시할 수 있도록 대안을 제공합니다.
즉, 다음과 같이 작성하는 대신:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

다음과 같이 `where` 조항을 사용할 수 있습니다:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

트레잇 바운드로 도배되지 않은 평범한 함수처럼
함수명, 매개변수 목록, 반환형이 붙어 있으니, 함수 시그니처를
읽기 쉬워집니다.

### 트레잇을 구현하는 타입을 반환하기

`impl Trait` 문법을 트레잇을 구현하는 어떤 타입의 값을 반환하는 데
사용할 수도 있습니다:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

반환 타입에 구체적인 타입명이 아닌 `impl Summary` 를 작성하여
`returns_summarizable` 함수는 `Summary` 트레잇을 구현하는 어떤 타입을 반환함을 명시했습니다.
이 경우 `returns_summarizable` 는 `Tweet` 을 반환하지만,
이 함수를 호출하는 쪽의 코드에서는 구체적인 타입을 알 필요가 없습니다.

구현되는 트레잇으로 반환 타입을 명시하는 기능은 13장에서
다룰 클로저 및 반복자 문법에서 굉장히 유용합니다. 클로저와
반복자는 컴파일만 아는 타입이나, 직접 명시하기에는 굉장히
긴 타입을 생성합니다. `impl Trait` 문법을 사용하면 굉장히
긴 타입을 직접 작성할 필요 없이 `Iterator` 트레잇을 구현하는
어떤 타입이라고만 간결하게 명시할 수 있습니다.

하지만, `impl Trait` 문법을 쓴다고 해서 다양한 타입을 반환할 수는 없습니다.
다음은 반환형을 `impl Summary`로 명시하고 `NewsArticle`, `Tweet` 중 하나를 반환하는 코드 예시입니다.
이 코드는 컴파일할 수 없습니다:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

`NewsArticle`, `Tweet` 중 하나를 반환하는 행위는 컴파일러 내에서
`impl Trait` 문법이 구현되는 방식으로 인해 제한됩니다.
함수가 이렇게 동작하도록 만드는 방법은 17장의
["트레잇 객체를 사용하여 다른 타입 간의 값 허용하기"][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore -->
절에서 알아볼
예정입니다.

### 트레잇 바운드를 사용해 조건부로 메소드 구현하기

제네릭 타입 매개변수를 사용하는 `impl` 블록에 트레잇 바운드를 이용하면,
명시된 트레잇을 구현하는 타입에만 메소드를 구현할 수도 있습니다.
예를 들어, Listing 10-15의 `Pair<T>` 타입은 항상 새로운 `Pair<T>` 인스턴스를
반환하는 `new` 함수를 구현합니다 (5장의 [“메소드 정의”][methods]<!-- ignore -->절에서
다룬 것처럼 `Self`는 `impl` 블록에 대한 타입의 별명이고, 지금의 경우에는
`Pair<T>`라는 점을 상기합시다). 하지만 그 다음의 `impl` 블록에서는,
어떤 `T` 타입이 비교를 가능하게 해주는 `PartialOrd` 트레잇과 출력을
가능하게 만드는 `Display` 트레잇을 모두 구현하는 타입인 경우에 대해서만
`cmp_display` 메소드를 구현하고 있습니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

<span class="caption">Listing 10-15: 트레잇 바운드를 이용해 제네릭 타입에
조건부로 메소드 구현하기</span>

타입이 특정 트레잇을 구현하는 경우에만 해당 타입에 트레잇을 구현할 수도 있습니다.
트레잇 바운드에 맞는 모든 타입에 트레잇을 구현하는 것을
*포괄 구현(blanket implementations)* 이라 하며,
이는 러스트 표준 라이브러리 내에서 광범위하게 사용됩니다.
예를 들어, 표준 라이브러리는 `Display` 트레잇을 구현하는 모든 타입에 `ToString` 트레잇도 구현합니다.
표준 라이브러리 내 `impl` 블록은 다음과 유사합니다:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

우리가 `Display` 트레잇이 구현된 모든 타입에 `to_string()`
메소드(`ToString` 트레잇에 정의된)를 호출할 수 있는 건
표준 라이브러리의 이 포괄 구현 덕분입니다.
예를 들어, 정수는 `Display`를 구현하므로 `String` 값으로 변환할 수 있습니다.

```rust
let s = 3.to_string();
```

포괄 구현은 트레잇 문서 페이지의 "구현체(Implementors)" 절에
있습니다.

트레잇과 트레잇 바운드를 사용하면 제네릭 타입 매개변수로
코드 중복을 제거하면서 '특정 동작을 하는 제네릭 타입'
이 필요하다는 걸 컴파일러에게 전달할 수 있습니다.
컴파일러는 트레잇 바운드를 이용하여 우리가 코드에서
사용한 구체적인 타입들이 알맞은 동작을 제공하는지 검사합니다.
동적 타입 언어에선 해당 타입이 정의하지 않은 메소드를 호출하면
런타임에 에러가 발생합니다. 하지만 러스트는 컴파일타임에 에러를
제공하여 코드를 실행하기도 전에 문제를 해결하도록 강제합니다.
따라서 우린 런타임에 해당 동작을 구현하는지를 검사하는 코드를
작성할 필요가 없습니다. 컴파일 타임에 이미 다 확인했기 때문이죠.
러스트는 제네릭의 유연성과 성능 둘 다 놓치지 않습니다.

[using-trait-objects-that-allow-for-values-of-different-types]: ch17-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
