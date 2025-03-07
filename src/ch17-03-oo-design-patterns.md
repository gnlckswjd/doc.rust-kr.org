## 객체 지향 디자인 패턴 구현하기

*상태 패턴 (state pattern)* 은 객체 지향 디자인 패턴입니다. 이
패턴의 핵심은 어떤 값이 내부적으로 가질 수 있는 상태 집합을 정의한다는
것입니다. 상태는 *상태 객체 (state object)* 의 집합으로 표현되며,
값의 동작은 상태에 기반하여 변경됩니다. 여기서는 상태를 저장하는 필드가
있는 블로그 게시물 구조체의 예를 살펴보려하는데, 이 상태는 "초안 (draft)",
"검토 (review)", 혹은 "게시 (published)" 집합의 상태 객체가 될 것입니다.

상태 객체들은 기능을 공유합니다: 당연히 러스트에서는 객체와
상속보다는 구조체와 트레잇을 사용합니다. 각 상태 객체는 자신의
동작 및 다른 상태로 변경되어야 할 때의 시기를 담당합니다.
상태 객체를 보유한 값은 상태의 서로 다른 행동 혹은 상태 간의
전이가 이뤄지는 때에 대해 아무것도 모릅니다.

상태 패턴을 사용하면 프로그램의 사업적 요구사항들이 변경될 때,
상태를 보유한 값의 코드 혹은 그 값을 사용하는 코드는 변경될
필요가 없다는 이점이 있습니다. 상태 객체 중 하나의 내부 코드를
갱신하여 그 규칙을 바꾸거나 혹은 상태 객체를 더 추가하기만
하면 됩니다.

우선 좀 더 전통적인 객체 지향 방식으로 상태 패턴을 구현한 다음,
러스트에 좀 더 자연스러운 접근법을 사용해 보겠습니다. 상태 패턴을
사용하여 블로그 게시물 작업 흐름을 점진적으로 구현하는 방법을
자세히 살펴봅시다.

최종적인 기능은 다음과 같을 것입니다:

1. 블로그 게시물은 빈 초안으로 시작합니다.
2. 초안이 완료되면 게시물의 검토가 요청됩니다.
3. 게시물이 승인되면 게시됩니다.
4. 오직 게시된 블로그 게시물만이 출력될 내용물을 반환하므로, 승인되지 않은 게시물이
   실수로 게시되는 것을 방지할 수 있습니다.

게시물에 시도된 그 외의 변경사항은 어떤 영향도 미치지 않습니다. 예를 들어,
만약 검토를 요청하기도 전에 블로그 게시물 초안을 승인하려는 시도를 했다면,
그 게시물은 게시되지 않은 초안으로 남아있어야 합니다.

Listing 17-11은 이 작업 흐름을 코드의 형태로 보여줍니다: 이는 `blog`라는 이름의
라이브러리 크레이트에 구현하게 될 API를 사용하는 예제입니다. 아직 `blog` 크레이트를
구현하지 않았으므로 컴파일되지 않습니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:all}}
```

<span class="caption">Listing 17-11: `blog` 크레이트에 원하는 요구
동작들을 보여주는 코드</span>

사용자가 `Post::new`를 통해 새로운 블로그 게시물의 초안을 작성할 수 있도록
허용하고 싶습니다. 블로그 게시물에는 텍스트의 추가가 허용되도록 하고 싶습니다.
만약 승인 전에 게시물의 내용을 즉시 얻어오는 시도를 하면, 해당 게시물이 아직
초안이기 때문에 텍스트를 가지고 올 수 없어야 합니다. 이를 확인할 목적으로 코드에
`assert_eq!`를 추가했습니다. 이를 위한 훌륭한 단위 테스트는 블로그 게시물 초안이
`content` 메소드에서 빈 문자열을 반환하는지 확인하는 것이겠지만, 이 예제에서는
테스트를 작성하지는 않겠습니다.

다음으로 게시물의 검토를 요청하는 것을 활성화하고, 검토를 기다리는 동안에는
`content`가 빈 문자열을 반환하도록 하려고 합니다. 게시물이 승인을 받은
시점에는 게시가 되어야 하므로, `content`의 호출되었을 때 게시물의 글이
반환될 것입니다.

이 크레이트에서 상호작용 하고 있는 유일한 타입이 `Post` 타입임을
주목하세요. 이 타입은 상태 패턴을 사용하며 게시물이 가질 수 있는 초안,
검토 대기, 게시됨을 나타내는 세 가지 상태 객체 중 하나가 될 값을
보유할 것입니다. 어떤 상태에서 다른 상태로 변경되는 것은 `Post` 타입
내에서 내부적으로 관리됩니다. 이 상태들은 라이브러리 사용자가 `Post`
인스턴스에서 호출하는 메소드에 대해 응답하여 변경되지만, 사용자가 상태
변화를 직접 관리할 필요는 없습니다. 또한, 사용자는 검토 전에 게시물이
게시되는 것 같은 상태와 관련된 실수를 할 수 없습니다.

### `Post`를 정의하고 초안 상태의 새 인스턴스 생성하기

라이브러리 구현을 시작해 봅시다! 어떤 내용물을 담고 있는 공개된
`Post` 구조체가 필요하다는 것을 알고 있으므로, Listing 17-12와
같이 구조체의 정의와 `Post`의 인스턴스를 만들기 위한 연관 공개
함수 `new`를 정의로 시작하겠습니다. `Post`에 대한 모든 상태
객체가 가져야하는 동작이 정의된 비공개 `State` 트레잇도
만들겠습니다.

그 다음 `Post`는 상태 객제를 담기 위해 `Option<T>`로 감싸진 `Box<dyn State>`
트레잇 객체를 `state`라는 비공개 필드로 가지게 될 것입니다. `Option<T>`가 왜
필요한지는 곧 보게 될겁니다.

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-12/src/lib.rs}}
```

<span class="caption">Listing 17-12: `Post` 구조체, 새로운 `Post`
인스턴스를 만드는 `new` 함수, `State` 트레잇, 그리고 `Draft` 구조체의
정의</span>

`State` 트레잇은 서로 다른 게시물 상태들이 공유하는 동작을 정의합니다.
상태 객체는 `Draft`, `PendingReview`, 그리고 `Published`이며, 모두
`State` 트레잇을 구현할 것입니다. 지금은 트레잇에 아무 메소드도 없고,
`Draft` 상태가 게시물이 시작되도록 원하는 상태이므로 `Draft` 상태만
정의하는 것으로 시작하겠습니다.

새로운 `Post`를 생성할 때는 이것의 `state` 필드에 `Box`를 보유한 `Some`
값을 설정합니다. 이 `Box`는 `Draft` 구조체의 새 인스턴스를 가리킵니다.
이렇게 하면 `Post`의 새 인스턴스가 생성될 때마다 초안으로 시작되는 것이
보장됩니다. `Post`의 `state` 필드가 비공개이기 때문에, 다른 상태로 `Post`를
생성할 방법은 없습니다! `Post::new` 함수에서는 `content` 필드를 새로운,
비어있는 `String`로 설정합니다.

### 게시물 콘텐츠의 텍스트 저장하기

Listing 17-11에서 `add_text`라는 메소드를 호출하고 여기에 `&str`을
전달하여 블로그 게시물의 콘텐츠 텍스트로 추가할 수 있길 원한다는
것을 보았습니다. 나중에 `content` 필드의 데이터를 읽는 방식을
제어할 수 있는 메소드를 구현할 수 있도록 `content` 필드를 `pub`으로
노출시키는 대신 메소드로 구현합니다. `add_text` 메소드는 매우
직관적이므로, Listing 17-13의 구현을 `impl Post` 블록에
추가해봅시다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-13/src/lib.rs:here}}
```

<span class="caption">Listing 17-13: `content`에 텍스트를 추가하기
위한 `add_text` 메소드 구현하기</span>

`add_text` 메소드는 가변 참조자 `self`를 취하는데, 이는 `add_text`를
호출하고 있는 해당 `Post` 인스턴스가 변경되기 때문입니다. 그 다음
`content`의 `String`에서 `push_str`을 호출하고 `text`를 인자로 전달해
저장된 `content`에 추가합니다. 이 동작은 게시물의 상태에 의존적이지
않으므로, 상태 패턴의 일부가 아닙니다. `add_text` 메소드는 `state`
필드와 전혀 상호작용을 하지 않지만, 지원하고자 하는 동작의
일부입니다.

### 초안 게시물의 내용이 비어있음을 보장하기

`add_text`를 호출하고 게시물에 어떤 콘텐츠를 추가한 이후일지라도, 여전히
`content` 메소드가 빈 스트링 슬라이스을 반환하길 원하는데, 그 이유는 
Listing 17-11의 7번째 줄처럼 게시물이 여전히 초안 상태이기 때문입니다. 당장은
이 요건을 만족할 가장 단순한 것으로 `content` 메소드를 구현해놓으려고 합니다:
언제나 빈 스트링 슬라이스를 반환하는 것으로요. 나중에 게시물이 게시될 수 있도록
게시물의 상태를 변경하는 기능을 구현하게 되면 이 메소드를 변경하겠습니다.
지금까지는 게시물이 오직 초안 상태만 가능하므로, 게시물 컨텐츠는 항상 비어 있어야
합니다. Listing 17-14는 이 껍데기 구현을 보여줍니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-14/src/lib.rs:here}}
```

<span class="caption">Listing 17-14: 항상 비어있는 스트링 슬라이스를
반환하는 `Post`의 `content` 메소드에 대한 껍데기 구현</span>

`content` 메소드를 추가함으로써, Listing 17-11의 7번째 줄까지는
의도한대로 작동됩니다.

### 게시물에 대한 검토 요청이 게시물의 상태를 변경합니다

다음에는 게시물의 검토를 요청하는 기능을 추가하고자 하는데, 이는 게시물의 상태를
`Draft`에서 `PendingReview`로 변경해야 합니다. Listing 17-15가 이 코드를 보여줍니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-15/src/lib.rs:here}}
```

<span class="caption">Listing 17-15: `Post`와 `State` 트레잇에
`request_review` 메소드를 구현하기</span>

`Post`에게 `self`에 대한 가변 참조자를 받는 `request_review`라는 이름의
공개 메소드가 제공되었습니다. 그런 다음 `Post`의 현재 상태에 대해 내부
메소드 `request_review`를 호출하고, 이 두번째 `request_review` 메소드는
현재의 상태를 소비하고 새로운 상태를 반환합니다.

`State` 트레잇에 `request_review` 메소드가 추가되었습니다; 트레잇을
구현하는 모든 타입은 이제 `request_review` 메소드를 구현할 필요가 있을
것입니다. 메소드의 첫 인자가 `self`, `&self`, 나 `&mut self`가 아니라
`self:Box<Self>`라는 점을 주목하세요. 이 문법은 메소드가 오직 그 타입을
보유한 `Box`에 대해서 호출될 경우에만 유효함을 뜻합니다. 이 문법은
`Box<Self>`의 소유권을 가져와서 `Post`의 예전 상태를 무효화하여 `Post`의
상태 값이 새로운 상태로 변환될 수 있도록 합니다.

이전 상태를 소비하려면 `request_review` 메소드가 상태 값의 소유권을
가져올 필요가 있습니다. 여기서 `Post`의 `state` 필드 내 `Option`이 중요한
역할을 합니다: 러스트는 구조체 내에 값이 없는 필드를 허용하지 않기 때문에,
`take` 메소드를 호출하여 `state` 필드 밖으로 `Some` 값을 빼내고 그
자리에 `None`을 남깁니다. 이렇게 하면 `state` 값을 빌리지 않고 `Post`
밖으로 옮길 수 있습니다. 그런 다음 게시물의 `state` 값을 이 작업의
결과물로 설정합니다.

`state` 값의 소유권을 얻기 위해서는 `self.state = self.state.request_review();`
처럼 직접 설정하지 않고 `state`를 임시로 `None`으로 설정할 필요가
있습니다. 이는 `Post`가 새 상태로 변환된 후 이전 `state` 값을
사용할 수 없음을 보장합니다.

`Draft`의 `request_review` 메소드는 새 `PendingReview` 구조체의 새로운,
박스로 감싸진 인스턴스를 반환하는데, 이는 게시물이 검토를 기다리는 상태를
나타냅니다. `PendingReview` 구조체도 `request_review` 메소드를 구현하지만
어떤 변환도 수행하지 않습니다. 오히려 자기 자신을 반환하는데, 이미
`PendingReview` 상태인 게시물에 대한 검토를 요청하면 `PendingReview` 상태를
유지해야 하기 때문입니다.

이제 상태 패턴의 장점을 확인할 수 있습니다: `Post`의 `request_review`
메소드는 `state` 값에 관계없이 동일합니다. 각 상태는 자신의
규칙을 담당합니다.

`Post`의 `content` 메소드는 빈 스트링 슬라이스를 반환하도록 그대로
놔두겠습니다. 이제는 `Draft` 상태에 있는 `Post` 뿐만 아니라 `PendingReview`
상태에 있는 `Post`도 있습니다만, `PendingReview` 상태에서도 동일한 동작이
필요합니다. Listing 17-11은 이제 10번째 줄까지 동작합니다!

<!-- Old headings. Do not remove or links may break. -->
<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

### `content`의 동작을 변경하는 `approve` 메소드 추가하기

`approve` 메소드는 `request_review` 메소드와 유사할 것입니다: 이것은
Listing 17-16과 같이 현재 상태가 승인되었을 때 갖게 되는 값으로 `state`를
설정하게 됩니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-16/src/lib.rs:here}}
```

<span class="caption">Listing 17-16: `Post`와 `State` 트레잇에
`approve` 메소드 구현하기</span>

`State` 트레잇에 `approve` 메소드를 추가하고 `State`를 구현하는
새 구조체 `Published` 상태도 추가합니다.

`Draft`의 `approve` 메소드를 호출하면 `PendingReview`의 `request_review`가
동작하는 것과 유사하게 `approve`가 `self`를 반환하므로 아무 효과가
없습니다. `PendingReview`에서 `approve`를 호출하면 박스로 포장된
`Published` 구조체의 새 인스턴스가 반환됩니다. `Published` 구조체는
`State` 트레잇을 구현하고, `request_review`와 `approve` 메소드 양 쪽
모두의 경우 게시물이 `Published` 상태를 유지해야 하므로 자기 자신을
반환합니다.

이제 `Post`의 `content` 메소드를 갱신해야 합니다. `content`로부터 반환된
값이 `Post`의 현재 상태에 의존적이길 원하므로, Listing 17-17과 같이
`Post`가 자신의 `state`에 정의된 `content` 메소드에게 위임 (delegate)
하도록 할 것입니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-oop/listing-17-17/src/lib.rs:here}}
```

<span class="caption">Listing 17-17: `Post`의 `content` 메소드가
`State`의 `content` 메소드에게 위임하도록 갱신하기</span>

목표는 `State`를 구현하는 구조체들 안에서 이 모든 규칙을 유지하는 것이기
때문에, `state`의 값에 `content` 메소드를 호출하고 게시물 인스턴스
(즉 `self`) 를 인자로 넘겨줍니다. 그러면 `state` 값의 `content`
메소드를 사용하여 얻어낸 값이 반환됩니다.

`Option`의 `as_ref` 메소드가 호출되었는데 `Option` 값에 대한 소유권이 아니라
그에 대한 참조자가 필요하기 때문입니다. `state`는 `Option<Box<dyn State>>`이므로,
`as_ref`가 호출되면 `Option<&Box<dyn State>>`가 반환됩니다. `as_ref`를
호출하지 않는다면, 함수 매개변수의 `&self`로부터 빌려온 `state`를 이동시킬
수 없기 때문에 에러가 발생했을 것입니다.

그 다음은 `unwrap`이 호출되는데, `Post`의 메소드가 완료되면
`state`에 언제나 `Some` 값이 들어있음을 보장한다는 것을 알고
있으므로 패닉이 발생하지 않을 것입니다. 이는 9장의
[“여러분이 컴파일러보다 더 많은 정보를 가진 경우”][more-info-than-rustc]<!-- ignore -->
절에서 다루었던, 컴파일러는 이해할 수 없지만 `None` 값이
절대 불가능함을 알고 있는 경우 중 한가지
입니다.

이 시점에서 `&Box<dyn State>`의 `content`가 호출되면, `&`와 `Box`에
역참조 강제가 적용되어, `content` 메소드는 궁극적으로 `State`
트레잇을 구현하는 타입에서 호출될 것입니다. 이는 즉 `State` 트레잇
정의에 `content`를 추가해야 함을 뜻하고, Listing 17-18처럼
가지고 있는 상태에 따라 어떤 내용물을 반환할지에 대한 로직을
여기에 넣을 것입니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-18/src/lib.rs:here}}
```

<span class="caption">Listing 17-18: `State` 트레잇에 `content` 메소드
추가하기</span>

`content` 메소드에 대하여 빈 스트링 슬라이스를 반환하는 기본 구현이
추가되었습니다. 이는 즉 `Draft`와 `PendingReview` 구조체에 대한 `content`는
구현할 필요가 없음을 뜻입니다. `Published` 구조체는 `content` 메소드를
오버라이딩하고 `post.content`의 값을 반환할 것입니다.

10장에서 설명한 것처럼 이 메소드에 대한 라이프타임 명시가 필요하다는 것에
주의하세요. `post`에 대한 참조자를 인자로 취하고 있고 해당 `post`의
일부분에 대한 참조자를 반환하는 중이므로, 반환되는 참조자의 라이프타임은
`post` 인자의 라이프타임과 관련이 있습니다.

그리고 이제 끝났습니다—이제 Listing 17-11의 모든 코드가 작동합니다!
블로그 게시물의 작업 흐름을 상태 패턴을 통해 구현해냈습니다. 규칙과 관련된
로직들은 `Post` 전체에 흩어져 있는 것이 아닌 상태 객체 안에서만 존재합니다.

> #### 열거형 쓰면 안되나요?
>
> 서로 다른 가능한 게시물 상태를 variant로 하는 `enum`을 쓰지 않는 이유가
> 궁금하실지도 모르겠습니다. 그것도 확실히 가능한 솔루션이니, 한번 시도해보고
> 그 결과를 비교해서 어떤 쪽이 더 나은지 확인해보세요! 열거형 사용의 단점
> 하나는 열거형 값을 검사하는 모든 위치에서 가능한 모든 variant를 처리하기
> 위하여 `match` 표현식 혹은 이와 유사한 표현식이 필요하다는 점입니다. 이는
> 지금의 트레잇 객체 솔루션에 비해 더 반복적일 수 있습니다.

### 상태 패턴의 기회비용

게시물이 각 상태에서 가져야 하는 다양한 종류의 동작을 캡슐화하기
위해서 러스트로 객체 지향 상태 패턴을 충분히 구현할 수 있음을
보았습니다. `Post`의 메소드는 이런 다양한 동작에 대해서 전혀 알지 못합니다.
코드를 구조화한 방식에 따라, 게시된 게시물이 작동할 수 있는 서로 다른
방식을 알기 위해서는 단 한 곳만 보면 됩니다: 바로 `Published` 구조체에서
`State` 트레잇을 구현한 내용말이죠.

만약 상태 패턴을 사용하지 않는 다른 구현을 만들려면, 대신
`Post`의 메소드나 심지어 `main` 코드에서 `match` 표현식을 사용하여
게시물의 상태를 검사하고 이에 따라 해야 할 행동을 변경할 수도
있겠습니다. 이는 게시된 상태의 게시물의 모든 결과들에 대해
이해하기 위해서 여러 곳을 살펴봐야 한다는 것을 뜻합니다! 이는
상태를 더 많이 추가할수록 각 `match` 표현식에 또다른 갈래를 추가가
필요하게 됩니다.

상태 패턴을 이용하면 `Post`의 메소드와 `Post`를 사용하는 곳에서는
`match` 표현식을 사용할 필요가 없고, 새로운 상태를 추가하려면 그저 새로운
구조체와 구조체에 대한 트레잇 메소드들을 구현하면 됩니다.

상태 패턴을 사용하는 구현은 더 많은 기능을 추가하는 확장이 쉽습니다.
상태 패턴을 사용하는 코드를 유지하는 것이 간단하다는 것을 확인해보려면,
다음 몇 가지 제안 사항을 시도해보세요:

* 게시물의 상태를 `PendingReview`에서 `Draft`로 변경하는 `reject` 메소드
  추가하기
* 상태를 `Published`로 변경하려면 `approve`의 호출이 두 번 필요하도록 하기
* 게시물이 `Draft` 상태일 때는 사용자들에게 텍스트 콘텐츠 추가만 하용하기.
  힌트: 상태 객체가 콘텐츠에 관한 변경은 담당하지만 `Post`를 수정할
  책임은 없도록 하기

상태 패턴의 한가지 단점은, 상태가 상태 간의 전환을 구현하기
때문에, 일부 상태가 서로 결합되어 있다는 것입니다. 만약
`PendingReview`와 `Published` 사이에 `Scheduled`와 같은 또다른
상태를 추가하면, `PendingReview`의 코드를 변경하여 `Scheduled`로
대신 전환되도록 해야 합니다. 새로운 상태의 추가할 때 `PendingReview`가
변경될 필요가 없었다면 작업량이 줄어들겠지만, 이는 다른 디자인
패턴으로의 전환을 의미할 겁니다.

또다른 단점은 일부 로직이 중복된다는 점입니다. 일부 중복을 제거하기
위해서 `State` 트레잇의 `request_review`와 `approve` 메소드가
`self`를 반환하도록 기본 구현을 만드는 시도를 할 수도 있습니다; 하지만
이는 트레잇이 구체적인 `self`가 정확히 무엇인지 모르기 때문에 객체
안전성을 위반할 수 있습니다. `State`가 트레잇 객체로 사용될 수 있기를
원하므로, 해당 메소드들이 객체 안전성을 지킬 필요가 있습니다. 

`Post`의 `request_review`와 `approve` 메소드의 유사한 구현들도
그 밖의 중복에 포함됩니다. 두 메소드 모두 `Option`의 `state` 필드
값에 대해 동일한 메소드의 구현을 위임하며, `state` 필드의 새 값을
결과로 설정합니다. 이 패턴을 따르는 `Post`의 메소드가 많다면,
매크로를 정의하여 반복을 없애는 것도 좋을 수 있겠습니다
(19장의 [“매크로”][macros]<!-- ignore -->절을 살펴보세요).

객체 지향 언어에서 정의된 상태 패턴을 그대로 구현하는
것으로는 러스트의 강점을 최대한 활용하지 못합니다. 유효하지
않은 상태와 전환을 컴파일 타임 에러로 만들 수 있도록 `blog`
크레이트에 적용할 수 있는 변경 사항 몇 가지를 살펴봅시다.

#### 상태와 동작을 타입처럼 인코딩하기

상태 패턴을 재고하여 다른 기회 비용 세트를 얻는 방법을 보여드리겠습니다.
상태와 전환을 완전히 캡슐화하여 외부 코드들이 이를 알 수 없도록 하는
대신, 상태를 다른 타입들로 인코딩하려고 합니다. 결과적으로 러스트의 타입
검사 시스템은 컴파일 에러를 발생시켜 게시된 게시물만 허용되는 곳에서
게시물 초안을 사용하려는 시도를 방지할 것입니다.

Listing 17-11의 `main` 첫 부분을 고려해 봅시다:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-11/src/main.rs:here}}
```

`Post::new`를 사용하여 초안 상태의 새 게시물을 생성하고 게시물의
내용에 새 글을 추가할 수 있는 기능은 계속 사용할 수 있습니다. 하지만
초안 게시물의 `content` 메소드가 빈 문자열을 반환하는 대신, 초안 게시물이
`content` 메소드를 갖지 않도록 만들려고 합니다. 이렇게 하면 초안 게시물의
내용을 얻는 시도를 할 경우, 해당 메소드가 존재하지 않는다는 컴파일 에러가
발생할 것입니다. 결과적으로, 프로덕션 환경에서 실수로 초안 게시물의 내용을
얻게 되는 일은 아예 컴파일조차 되지 않으므로 불가능해집니다.
Listing 17-19는 `Post` 구조체와 `DraftPost` 구조체의 정의와 각각의
메소드를 보여줍니다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-19/src/lib.rs}}
```

<span class="caption">Listing 17-19: `content` 메소드가 있는
`Post`와 `content` 메소드가 없는 `DraftPost`</span>

`Post`와 `DraftPost` 구조체 모두 블로그 게시물의 텍스트를 저장하는 비공개
`content` 필드를 가지고 있습니다. 이 구조체들이 더 이상 `state` 필드를 갖지 않는
이유는 상태의 인코딩을 구조체의 타입으로 옮겼기 때문입니다. `Post` 구조체는
공개된 게시물을 나타낼 것이고, `content`를 반환하는 `content` 메소드가
있습니다.

`Post::new` 함수는 여전히 있지만, `Post`의 인스턴스를 반환하는
대신 `DraftPost`를 반환합니다. `content`는 비공개이고 `Post`를 반환할
어떤 함수도 존재하지 않기 때문에, 곧바로 `Post`의 인스턴스를 생성하는
것은 불가능합니다. 

`DraftPost` 구조체에 `add_text` 메소드가 있으므로 전처럼 `content`에
텍스트를 추가할 수 있지만, `DraftPost`에는 `content` 메소드가 정의되어 있지
않다는 것을 주의하세요! 따라서 이제 프로그램은 모든 게시물이 초안 게시물로
시작되고, 초안 게시물은 자신의 콘텐츠를 표시할 수 없도록 합니다. 이러한
제약사항을 우회하려는 시도는 컴파일 에러를 발생시킬 것입니다.

#### 다른 타입으로 변환하는 것으로 전환 구현하기

그러면 게시물을 게시하려면 어떻게 해야 할까요? 초안 게시물이 게시되기
전에 검토와 승인을 받아야 하는 규칙은 적용되기를 원합니다. 검토를 기다리는
상태인 게시물은 여전히 어떤 내용도 보여줘서는 안되구요. Listing 17-20처럼
또다른 구조체 `PendingReviewPost`를 추가하고, `DraftPost`에
`PendingReviewPost`를 반환하는 `request_review` 메소드를 정의하고,
`PendingReviewPost`에 `Post`를 반환하는 `approve` 메소드를 정의하여
위의 제약사항들을 구현해봅시다:

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch17-oop/listing-17-20/src/lib.rs:here}}
```

<span class="caption">Listing 17-20: `DraftPost`의 `request_review`를
호출하여 생성되는 `PendingReviewPost` 및 `PendingReviewPost`를 게시된
`Post`로 전환하는 `approve` 메소드</span>

`request_review`와 `approve` 메소드는 `self`의 소유권을 가져와서
`DraftPost`와 `PendingReviewPost`의 인스턴스를 소비하고 이들을
각각 `PendingReviewPost`와 게시된 `Post`로 변환시킵니다. 이렇게 하면
`request_review`를 호출한 후 등등에는 `DraftPost` 인스턴스가
남아있지 않게 될겁니다. `PendingReviewPost` 구조체에는 `content`
메소드가 정의되어 있지 않기 때문에, 그 콘텐츠를 읽으려는 시도는 `DraftPost`에서와
마찬가지로 컴파일 에러를 발생시킵니다. `content` 메소드가 정의된
게시된 `Post` 인스턴스를 얻을 수 있는 유일한 방법은 `PendingReviewPost`의
`approve` 메소드를 호출하는 것이고, `PendingReviewPost`를 얻을 수
있는 유일한 방법은 `DraftPost`의 `request_review`를 호출하는 것이므로,
이제 블로그 게시물의 작업 흐름을 타입 시스템으로 인코딩했습니다.

또한 `main`에도 약간의 수정이 필요합니다. `request_review`와 `approve`
메소드는 호출되고 있는 구조체를 변경하는 것이 아니라 새 인스턴스를
반환하기 때문에, 더 많은 `let post =` 쉐도잉 할당을 추가하여
반환되는 인스턴스를 보관해야 합니다. 또한 초안과 검토 중인 게시물의
내용이 빈 문자열이라고 단언할 수도 없고, 단언할 필요도 없습니다: 이 상태에서
게시물이 콘텐츠를 사용 시도하는 코드는 더 이상 컴파일되지 않습니다.
Listing 17-12에 갱신된 `main` 코드가 있습니다:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch17-oop/listing-17-21/src/main.rs}}
```

<span class="caption">Listing 17-21: 새로운 블로그 게시물 작업 흐름
구현을 사용하기 위한 `main` 수정</span>

`main`에서 `post`의 다시 대입하기 위해 필요한 이 변경사항은 즉 이
구현이 더이상 객체 지향 상태 패턴을 잘 따르지 않는다는 것을 의미합니다:
즉 상태간의 변환이 더 이상 `Post`의 구현체 내에 모두 캡슐화되지
않습니다. 하지만, 타입 시스템과 컴파일 타임에 일어나는 타입
검사로 인해 유효하지 않은 상태는 이제 불가능해졌습니다!
이는 게시되지 않은 게시물의 내용이 보여진다거나 하는 특정
버그들이 프로덕션에 적용되기 전에 발견될 것임을 보장합니다.

이번 절의 시작점에서 제안되었던 작업들을 Listing 17-21의 `blog`
크레이트에서 그대로 시도해보면서 이 버전의 코드 디자인에 대해
어떻게 생각하는지 살펴보세요. 일부 작업은 이번 디자인에서 이미
완료되었을 수도 있음을 알려드립니다. 

러스트에서 객체 지향 디자인 패턴의 구현이 가능할지라도, 러스트에서는
상태를 타입 시스템으로 인코딩하는 다른 패턴도 사용할 수 있음을
확인했습니다. 이 패턴들은 서로 다른 기회비용을 갖고 있습니다.
여러분이 객체 지향 패턴에 매우 익숙할 수도 있지만, 문제를 다시
생각하여 러스트의 기능을 활용하면 컴파일 타임에 일부 버그를 방지하는
등의 이점을 얻을 수 있습니다. 소유권 같은 객체 지향 언어에는
없는 특정 기능으로 인해 객체 지향 패턴이 항상 러스트에서 최고의
해결책이 되지는 못합니다.

## 정리

이 장을 읽은 후 러스트가 객체 지향 언어라고 생각하든 그렇지 않든,
여러분은 이제 트레잇 객체를 사용하여 일부 객체 지향 기능을 러스트
내에서 사용할 수 있다는 것을 알게 되었습니다. 동적 디스패치는
약간의 실행 성능과 맞바꿔 여러분의 코드에 유연성을 줄 수 있습니다.
이 유연성을 사용하여 여러분의 코드의 유지보수에 도움이 되는
객체 지행 패턴을 구현할 수 있습니다. 러스트에는 또한 소유권과 같은
객체 지향 언어들에는 없는 다른 기능도 있습니다. 객체 지향 패턴이
항상 러스트의 강점을 활용하는 최고의 방법은 아니겠지만, 사용할 수 있는
옵션입니다.

다음으로는 패턴을 살펴볼 것인데, 이는 높은 유연성을 제공하는 러스트의
또다른 기능 중 하나입니다. 이 책 전체에 걸쳐 패턴을 간단히 살펴보긴
했지만 아직 모든 기능을 살펴본건 아닙니다. 가볼까요!

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch19-06-macros.html#macros
