# 추리 게임

실습 프로젝트를 통해 러스트를 사용해 봅시다. 이번 장은 실제 프로젝트에서 몇몇
일반적인 Rust 개념이 어떻게 활용되는지를 소개하려 합니다. 이 과정에서 `let`,
`match`, 메소드, 연관함수(assiciated functions), 외부 크레이트(external crates)
등의 활용 방법을 배울 수 있습니다. 이런 개념들은 다음 장들에서 더 자세히 다뤄질
것입니다. 이번 장에서는 여러분이 직접 기초적인 내용을
실습합니다.

우리는 고전적인 입문자용 프로그래밍 문제인 추리 게임을 구현해 보려 합니다.
먼저 프로그램은 1~100 사이의 임의의 정수를 생성합니다.
다음으로 플레이어가 프로그램에 추리한 정수를 입력합니다.
프로그램은 입력받은 추리값이 정답보다 높거나 낮은지를 알려줍니다.
추리값이 정답이라면 축하 메세지를 보여주고 종료됩니다.

## 새로운 프로젝트를 준비하기

새로운 프로젝트를 준비하기 위해 1장에서 생성했던 디렉토리인 _projects_ 로 이동하고
아래 예제처럼 Cargo를 이용하여 새로운 프로젝트를 생성합니다.

```console
$ cargo new guessing_game
$ cd guessing_game
```

첫 명령문인 `cargo new`는 프로젝트의 이름 (`guessing_game`)을
첫번째 인자로 받습니다. 두번째 명령문은 작업 디렉토리를 새로운 프로젝트의
디렉토리로 변경합니다.

생성된 *Cargo.toml* 파일을 살펴봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

1장에서 보았듯이 `cargo new`는 여러분을 위해 “Hello, world!” 프로그램을 생성합니다.
*src/main.rs* 파일을 살펴보면 다음과 같습니다.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

이제 이 "Hello, world!" 프로그램을 `cargo run` 명령문을 이용하여 컴파일하고 실행해
봅시다.

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

`run` 명령어는 이번 실습 프로젝트처럼 빠르게 반복(iteration)을
하고 싶을 때 유용합니다. 우리는 다음 iteration으로 넘어가기 전 빠르게
각 iteration을 테스트하고 싶습니다.

*src/main.rs* 를 다시 열어 두세요. 이 파일에 모든 코드를 작성할 것입니다.

## 추리값을 처리하기

프로그램의 첫 부분은 사용자 입력 요청, 입력값의 처리 후
입력값이 기대하던 형식인지 검증합니다. 첫 시작으로 플레이어가
추리한 값을 입력받을 수 있게 할 것입니다.
Listing 2-1의 코드를 *src/main.rs* 에 작성하세요.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

<span class="caption">Listing 2-1: 사용자가 추리한 값을 입력 받아 그대로
출력하는 코드</span>

이 코드에 담긴 다양한 정보를 하나씩 살펴 보겠습니다.
사용자 입력을 받고 결과값을 표시하기 위해서는 `io`
(input/output) 라이브러리를 스코프로 가져와야 합니다.
`io` 라이브러리는 `std` 라고 불리는 표준 라이브러리에 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

기본적으로 러스트는 모든 프로그램의 스코프로 가져오는 표준 라이브러리에
정의된 아이템 세트를 가지고 있습니다. 이 세트를 *프렐루드 (prelude)* 라고
부르며, 이와 관련한 것은 [표준 라이브러리 문서][prelude]에서 찾아볼 수 있습니다.

만약 여러분이 원하는 타입이 프렐루드에 없다면
`use`문을 활용하여 명시적으로 그 타입을 가져와야 합니다.
`std::io`는 사용자의 입력을 받는 것을 포함하여 `io`와 관련된
기능들을 제공합니다.

1장에서 보았듯이 `main` 함수는 프로그램의
진입점입니다:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` 문법은 새로운 함수를 선언하며, 괄호 `()`는 인자가 없음을 나타내고
중괄호 `{`는 함수 본문의 시작을 나타냅니다.

1장에서 배웠듯이 `println!`은 string을 화면에 표시하는
매크로입니다:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

이 코드는 게임에 대한 설명과 사용자의 입력을 요청하는
글자를 표시합니다.

### 값을 변수에 저장하기

다음으로 우리는 다음 코드처럼 사용자의 입력값을 저장할 *변수(variable)* 를 생성할 수 있습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

이제 프로그램이 점점 흥미로워지고 있습니다! 이 짧은 라인에서 여러 일들이 벌어집니다.
변수를 만드는 데에는 `let` 구문을 사용합니다. 다음 코드도 변수를 선언하는 예시입니다:

```rust,ignore
let apples = 5;
```

이 라인은 `apples`라는 변수를 선언하고 5라는 값을 묶어 넣습니다.
러스트에서 변수는 기본적으로 불변 (immutable) 인데, 이는 변수에
어떤 값을 집어 넣으면 그 값이 안 바뀔 것이란 뜻입니다. 이 개념에 대한
자세한 내용은 3장의 [“변수와 가변성 (mutable)”][variables-and-mutability]<!-- ignore -->
절에서 논의할 예정입니다. 변수의 값을 변할 수 있도록 하려면 변수명
앞에 `mut`를 추가합니다:

```rust,ignore
let apples = 5; // 불변
let mut bananas= 5; // 가변
```

> Note: `//` 문법은 현재 위치부터 라인의 끝까지 주석임을 나타냅니다.
> 러스트는 주석의 모든 내용을 무시합니다. 더 자세한 내용은 3장에서
> 설명할 예정입니다.

추리 게임 프로그램으로 다시 돌아와보면, 이제 `let mut guess`가 `guess`라는
이름의 가변성 변수임을 알 수 있습니다. 등호(`=`)는 지금 해당 변수에 어떤 값을
묶어넣고 싶다는 것을 뜻합니다. 등호의 오른쪽에는 `guess`에 묶고자 하는 값이
있는데, 이번 예시에서는 함수 `String::new`의 결과값인
새로운 `String` 인스턴스가 묶이는 대상이 됩니다.
[`String`][string]<!-- ignore -->은 표준 라이브러리에서
제공하는 확장 가능한(growable) UTF-8 인코딩의 문자열 타입입니다.

`::new`에 있는 `::`는 `new`가 `String` 타입의 *연관 함수 (associated function)*
임을 나타냅니다. 연관함수는 임의의 타입 상에 구현된 함수고, 위의 경우에는
`String` 타입 위에 만들어진 함수입니다. 이 `new` 함수는 비어있는 새 스트링을
생성합니다. 어떤 새로운 값을 만드는 함수 이름으로서 `new`는 흔히 사용되는 이름이기
때문에, 여러 타입에서 `new` 함수를 찾아볼 수 있을 겁니다.

요약하자면 `let mut guess = String::new();` 라인은
새로운 빈 `String` 인스턴스와 연결된 가변변수를 생성합니다.

### 사용자 입력 받기

프로그램에 첫번째 라인에 `use std::io;` 를 이용하여
표준 라이브러리의 input/output 기능을 포함한 것을 떠올려 보세요.
이제 `io`의 연관함수인 `stdin`을 호출하는데, 이것이 사용자의 입력을
다루도록 해줄 것입니다:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

만약 프로그램 시작점에 `use std::io`를 통해 `io` 라이브러리를 가져오지
않았더라도, 함수 호출 시 `std::io::stdin`처럼 작성하는 것으로 이 함수를
이용할 수 있습니다. `stdin` 함수는 터미널의 표준 입력의 핸들(handle)을
나타내는 타입인 [`std::io::Stdin`][iostdin]<!-- ignore -->의 인스턴스를
돌려줍니다.

코드의 다음 부분인 `.read_line(&mut guess)`는 사용자로부터
입력을 받기 위해 표준 입력 핸들에서 [`read_line`][read_line]<!-- ignore -->
메소드를 호출합니다. 여기에 `read_line`의 인자로서
`&mut guess`를 넘겨서 사용자 입력이 저장될 스트링이
무엇인지 알려줍니다. `read_line`의 전체 기능은 사용자가 표준 입력 장치에
입력할 때마다 입력된 문자들을 받아서 스트링에 추가하는 것이므로
스트링을 인자로 넘겨준 것입니다. 메소드가 스트링의 내용물을 바꿀 수
있기 때문에 이 스트링 인자는 가변이어야 합니다.

`&`는 코드의 여러 부분에서 데이터를 여러 번 메모리로 복사하지 않고
접근하기 위한 방법을 제공하는 *참조자* 임을 나타냅니다.
참조자는 복잡한 특성으로서 러스트의 큰 이점 중 하나가
참조자를 사용함으로써 얻는 안전성과 용이성입니다.
이 프로그램을 작성하기 위해 참조자의 자세한 내용을 알 필요는 없습니다.
4장에서 참조자에 대해 전체적으로 설명할 것입니다.
지금 당장은 참조자가 변수처럼 기본적으로 불변임을 알기만 하면 됩니다.
따라서 가변으로 바꾸기 위해 `&guess`가 아니라
`&mut guess`로 작성해야 합니다.

<!-- Old heading. Do not remove or links may break. -->
<a id="handling-potential-failure-with-the-result-type"></a>

### `Result` 타입으로 잠재적 실패 다루기

아직 이 라인에 대해 다 설명하지 않았습니다. 코드의 세번째 라인에 대해서
논의하는 중이지만 실은 논리적으로 한 줄짜리 코드의 일부일 뿐입니다.
다음 부분은 아래의 메소드입니다:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

위 코드를 아래처럼 쓸 수도 있습니다.

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

하지만 하나의 긴 라인은 가독성이 떨어지므로 라인을 나누는
것이 좋습니다. `.method_name()` 문법으로 어떤 메소드를 호출할 때는
줄바꿈과 공백 문제로 긴 라인을 쪼개는 것이 보통 현명한 선택입니다.
이제 이 라인이 뭘 하는지 살펴봅시다.

이전에 언급한 것처럼 `read_line`은 우리가 인자로 넘긴 문자열에 사용자가
입력한 것을 저장할 뿐만 아니라 하나의 `Result` 값을 돌려 줍니다.
[`Result`][result]<!-- ignore --> 는 *enum*이라고 부르기도 하는
[*열거형 (enumeration*][enums]<!-- ignore --> 인데, 여러 개의 가능한 상태 중
하나의 값이 될 수 있는 타입입니다. 이러한 가능한 상태값을 *배리언트 (variant)* 라고 부릅니다.

[6장][enums]<!-- ignore -->] 에서 열거형에 대해 더 자세히 다루겠습니다.
`Result` 타입의 목적은 에러 처리용 정보를 담아내기 위한 것입니다.

`Result`의 배리언트는 `Ok` 혹은 `Err`입니다. `Ok`는 처리가 성공했음을
나타내며 내부적으로 성공적으로 생성된 결과를 가지고 있습니다.
`Err`는 처리가 실패했음을 나타내고 그 이유에 대한 정보를 가지고
있습니다.

`Result` 타입의 값들은 다른 타입들처럼 메소드들을 가지고 있습니다.
`Result` 인스턴스는 [`expect` 메소드][expect]<!-- ignore -->를
가지고 있습니다. 만약 `Result` 인스턴스가 `Err`일 경우 `expect`
메소드는 프로그램이 작동을 멈추게 하고 `expect`에 인자로 넘겼던
메세지를 출력하도록 합니다. 만약 `read_line` 메소드가 `Err`를
돌려줬다면 그 에러는 운영체제로부터 생긴 에러일 경우가 많습니다.
만약 `Result`가 `Ok` 값이라면 `expect`는 `Ok`가 가지고 있는
결과값을 돌려주어 사용할 수 있도록 합니다. 이 경우 결과값은
사용자가 표준 입력으로 입력했던 바이트의 개수입니다.

만약 `expect`를 호출하지 않는다면 컴파일은 되지만 경고가 나타납니다.

```text
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

러스트는 `read_line`가 돌려주는 `Result` 값을 사용하지 않았음을 경고하며 일어날 수 있는
에러를 처리하지 않았음을 알려줍니다.

이 경고를 없애는 옳은 방법은 에러 처리용 코드를 작성하는 것이지만
우리의 경우에는 문제가 발생했을 때 프로그램이 멈추길 원하므로 `expect`를 사용할
수 있습니다. [9장][recover]<!-- ignore -->에서 에러로부터 복구하는 방법에 대해
배우게 될 겁니다.

### `println!` 변경자(placeholder)를 이용한 값 출력

지금까지 작성한 코드에서 닫는 중괄호 말고도 살펴봐야 하는 코드가 하나 더 있습니다. 내용은
아래와 같습니다.

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

이 라인은 사용자가 입력한 값을 담고 있는 문자열을 출력합니다.
`{}`는 자리표시자 (placeholder) 입니다: `{}`를 어떤 위치에 값을
자리하게 하는 작은 집게발이라고 생각하면 됩니다. 어떤 변수의 값을 출력할 때라면
해당 변수 이름이 이 중괄호 안에 넣을 수 있습니다. 어떤 표현식의 결과값을
출력할 때는 빈 중괄호들를 형식 문자열에 위치시키고, 그 뒤에 콤마로 구분된
표현식들을 나열하여 각 중괄호에 순차적으로 출력하도록 할 수 있습니다.
어떤 변수와 표현식 결과값을 한번의 `println!` 호출로 출력한다면 아래와
같은 형태가 됩니다:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

이 코드는 `x = 5 and y = 12`을 출력합니다.

### 첫번째 부분을 테스트하기

추리 게임의 처음 부분을 테스트 해 봅시다. `cargo run`을 통해 실행할 수 있습니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```text
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

지금까지 게임의 첫번째 부분을 작성했습니다.
우리는 입력값을 받고 그 값을 출력했습니다.

## 비밀번호를 생성하기

다음으로 사용자가 추리하기 위한 비밀번호를 생성해야 합니다. 게임을 다시
하더라도 재미있도록 비밀번호는 매번 달라야 합니다. 게임이 너무 어렵지 않도록
1에서 100 사이의 임의의 수를 사용하겠습니다. 러스트는 아직 표준 라이브러리에
임의의 값을 생성하는 기능이 없습니다. 하지만 러스트 팀에서는
해당 기능을 가지고 있는 [`rand` 크레이트 (crate)][randcrate]를
제공합니다.

### 크레이트(Crate)를 사용하여 더 많은 기능 가져오기

크레이트는 러스트 코드의 묶음(package)임을 기억하세요. 우리가 만들고 있는
프로젝트는 실행이 가능한 *바이너리 크레이트 (binary crate)* 입니다.
`rand` crate는 자체적으로 실행될 수는 없고 다른 프로그램에서 사용되기 위한
용도인 *라이브러리 크레이트 (library crate)* 입니다.

Cargo의 외부 크레이트 조정은 정말 멋진 부분입니다. `rand`를 사용하는
코드를 작성하기 전에 *Cargo.toml* 을 수정하여 `rand` 크레이트를 의존
리스트에 추가해야 합니다. 파일을 열고 Cargo가 여러분을 위해 생성한
`[dependencies]` 절의 시작 바로 아래에 다음 내용을 추가하세요. 여기
적혀 있는대로 `rand`를 버전 숫자와 함께 정확하게 기입하지 않으면 이
튜토리얼의 예제 코드는 동작하지 않을 수도 있습니다.

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

*Cargo.toml* 파일에서 어떤 섹션의 헤더 이후의 모든 내용은
그 절에 포함되며 이는 다음 절이 나타날 때까지 동일합니다.
`[dependencies]`에서는 여러분의 프로젝트가 의존하고 있는 외부 크레이트와
각각의 요구 버전을 Cargo에게 알려주게 됩니다. 지금의 경우 우리는 `rand`
크레이트의 유의적 버전인 `0.8.5`을 지정했습니다. Cargo는 버전을 명시하는
표준에 해당하는 [Semantic Versioning][semver]<!-- ignore -->
(때때로 *SemVer* 라고 불리는) 을 이용합니다. 지정자 `0.8.5`는 실제로는
`^0.8.5`의 축약형인데, 이는 최소 `0.8.5` 이상이지만 `0.9.0` 아래의
버전을 의미합니다.

Cargo는 이 버전들이 `0.8.5`와 호환되는 공개 API를 갖추고 있다고
간주하며, 이러한 버전 지정은 이 장의 코드와 함께 컴파일이 되도록 하는
최신 패치판을 받게 될 것임을 보장합니다. 0.9.0 버전 혹은 그 이상의 버전은
이후의 예제에서 사용할 때 같은 API를 사용하는 것을 보장하지 못합니다.

이제 Listing 2-2처럼 코드 수정 없이 프로젝트를
빌드 해 봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
  Downloaded libc v0.2.127
  Downloaded getrandom v0.2.7
  Downloaded cfg-if v1.0.0
  Downloaded ppv-lite86 v0.2.16
  Downloaded rand_chacha v0.3.1
  Downloaded rand_core v0.6.3
   Compiling libc v0.2.127
   Compiling getrandom v0.2.7
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.16
   Compiling rand_core v0.6.3
   Compiling rand_chacha v0.3.1
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
```

<span class="caption">Listing 2-2: rand 크레이트를 의존성으로 추가한 후
`cargo build` 를 실행한 결과</span>

여러분은 다른 버전명이 보이거나 (하지만 SemVer 덕분에 현재 코드와 호환될 것입니다)
다른 라인들이 보이거나 (운영 체제에 따라서 달라질 수 있습니다) 라인의 순서가
다르게 보일 수 있습니다.

외부 의존성을 포함시키기 되면, Cargo는 [Crates.io][cratesio]로부터
데이터의 복사본인 *레지스트리 (registry)* 에서 해당 의존성이 필요로 하는
모든 것들의 최신 버전을 가져옵니다. Crates.io는 러스트 생태계의 개발자들이
다른 사람들도 이용할 수 있도록 러스트 오픈소스를 공개하는 곳입니다.

레지스트리를 업데이트하면 Cargo는 `[dependencies]` 절을 확인하고
아직 다운로드되지 않은 크레이트들을 다운 받습니다. 우리의 경우에는
`rand`만 의존한다고 명시했지만 Cargo는 `rand`가 동작하기 위해 의존하고
있는 다른 크레이트들도 가져옵니다. 이것들을 다운받은 후 러스트는 컴파일하여
의존성이 해결된 프로젝트를 컴파일합니다.

만약 아무것도 변경하지 않고 `cargo build`를 실행한다면 `Finished`
줄 외엔 어떠한 결과도 얻지 못할 것입니다. Cargo는 이미 의존 패키지들을
다운받고 컴파일했음을 알고 있고 여러분이 *Cargo.toml* 을
변경하지 않은 것을 알고 있습니다. 또한 Cargo는 코드가
변경되지 않은 것도 알고 있기에 코드도 다시 컴파일하지 않습니다.
아무것도 할 일이 없기에 그냥 종료될 뿐입니다.

만약 여러분이 *src/main.rs* 파일을 열어 사소한 변경을 하고
저장한 후 다시 빌드를 한다면 한 라인이 출력됨을 확인할 수 있습니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
```

이 라인은 Cargo가 *src/main.rs* 의 사소한 변경을 반영하여 빌드를 업데이트 했음을
보여줍니다. 의존 패키지가 변경되지 않았으므로 Cargo는 이미 다운받고 컴파일된 것들을
재사용할 수 있음을 알고 있습니다.

#### 재현 가능한 빌드를 보장하는 *Cargo.lock*

Cargo는 여러분 뿐만이 아니라 다른 누구라도 여러분의 코드를 빌드할 경우
같은 산출물이 나오도록 보장하는 방법을 가지고 있습니다.
Cargo는 여러분이 다른 의존성을 추가하지 전까지는 여러분이 명시한 의존
패키지만을 사용합니다. 예를 들어 `rand` 크레이트의 다음 버전인 0.8.6에서 중요한
결함이 고쳐졌지만 당신의 코드를 망치는 변경점 (regression) 이 있다고 칩시다.
이러한 문제를 해결하기 위해서 러스트는 여러분이 `cargo build`를 처음
실행할 때 *Cargo.lock* 파일을 생성하여 *guessing_game* 디렉토리에
가지고 있습니다.

여러분이 처음 프로젝트를 빌드할 때 Cargo는 기준을 만족하는
모든 의존 패키지의 버전을 확인하고 *Cargo.lock* 에 이를 기록합니다.
만약 여러분이 미래에 프로젝트를 빌드할 경우 Cargo는 모든 버전들을
다시 확인하지 않고 *Cargo.lock* 파일이 존재하는지 확인하여
그 안에 명시된 버전들을 사용합니다. 이는 여러분이 재현가능한
빌드를 자동으로 가능하게 합니다. 즉 여러분의 프로젝트는
*Cargo.lock* 덕분에 당신이 명시적으로 업그레이드하지 않는
이상 `0.8.5`를 이용합니다. *Cargo.lock* 파일이 재현 가능한 빌드에 있어
중요하기 때문에, 흔히 프로젝트의 코드와 함께 소스 제어 도구에서 검사됩니다.

#### 크레이트를 새로운 버전으로 업그레이드하기

만약 당신이 *정말* 크레이트를 업데이트하고 싶은 경우를 위해 Cargo는 `update` 명령어를
제공합니다. 이것은 *Cargo.lock* 파일을 무시하고 *Cargo.toml* 에 여러분이 명시한
요구사항에 맞는 최신 버전을 확인합니다. 확인이 되었다면 Cargo는 해당 버전을
*Cargo.lock* 에 기록합니다. 하지만 Cargo는 기본적으로 0.8.5보다 크고 0.9.0보다
작은 버전을 찾을 것입니다. 만약 `rand` 크레이트가 새로운 버전 0.8.6과 0.9.0
두 가지를 배포했다면 여러분이 `cargo update`를 실행했을 때 다음의 메세지를 볼
것입니다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.5.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
    Updating rand v0.8.5 -> v0.8.6
```

Cargo는 0.9.0 버전을 무시합니다. 이 시점에서 여러분은 *Cargo.lock*
파일에서 변경이 일어난 것과 앞으로 사용될 `rand` 크레이트의 버전이 0.8.6 임을
확인할 수 있습니다. 0.9.0이나 0.9.*x*에 해당하는 버전을 사용하고 싶다면
*Cargo.toml* 을 다음과 같이 업데이트해야 합니다.

```toml
[dependencies]
rand = "0.9.0"
```

다음번에 여러분이 `cargo build`를 실행하면 Cargo는 가용 가능한 크레이트들의
레지스트리를 업데이트할 것이고 여러분의 `rand`요구사항을 새롭게 명시한 버전에
따라 재계산할 것입니다.

[Cargo][doccargo]<!-- ignore -->와 [그의 생태계][doccratesio]<!-- ignore -->에
대해 더 많은 것들은 14장에서 다뤄지지만 지금 당장은 이 정도만 알면 됩니다.
Cargo는 라이브러리의 재사용을 쉽게 하여
러스트 사용자들이 많은 패키지들과 결합된 더 작은 프로젝트들을
작성할 수 있도록 도와줍니다.

### 임의의 숫자를 생성하기

`rand`를 사용하여 추리할 임의의 숫자를 생성해봅시다. 다음 단계는
*src/main.rs* 를 Listing 2-3처럼 업데이트하면 됩니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

<span class="caption">Listing 2-3: 임의의 숫자를 생성하기 위해 필요한 코드</span>

먼저 `use rand::Rng;` 라인을 추가합니다. `Rng`는 난수 생성기를 구현한
메소드들을 정의한 트레잇 (trait) 이며 해당 메소드들을 이용하기 위해서는 반드시
스코프 내에 있어야 합니다. 10장에서 트레잇에 대해 더 자세히 다룰 것입니다.

다음으로 우리는 중간에 두 개의 라인을 추가합니다.
첫번째 줄에서는 `rand::thread_rng` 함수를 호출하여 OS가 시드(seed)를 정하고
현재 스레드에서만 사용되는 특별한 난수 생성기를 돌려줍니다.
다음으로 우리는 `gen_range` 메소드를 호출합니다.
이 메소드는 `Rng` 트레잇에 정의되어 있으므로 `use rand::Rng;`
구문을 통해 스코프로 가져올 수 있습니다. `gen_range` 메소드는
범위 표현식을 인자로 받아서 해당 범위 내 임의의 숫자를 생성합니다.
여기서 사용하고자 하는 범위 표현식은 `start..=end`이고 이는
상한선과 하한선을 포함하므로, 1부터 100 사이의 숫자를
생성하려면 `1..=100`을 넘겨야 합니다.

> Note: 크레이트에서 트레잇과 메소드, 함수중 어떤 것을 호출해야
> 할지 모를 수도 있으므로, 각 크레이트는 사용법을 담고 있는
> 문서를 갖추고 있습니다. Cargo의 다른 멋진 기능은
> `cargo doc --open` 명령어를 사용하여 의존하는 크레이트의
> 문서를 로컬에서 모두 빌드한 다음, 브라우저에서 열 수 있다는 것입니다.
> `rand` 크레이트의 다른 기능이 궁금하시면, `cargo doc --open`을
> 실행하고, 왼쪽 사이드바에서 `rand`를 클릭하여 알 수
> 있습니다.

코드에 추가한 두 번째 라인은 비밀번호를 표시합니다.
이 라인은 우리가 프로그램을 개발 중 테스트 할 때 유용하지만
최종 버전에서는 삭제할 것입니다. 프로그램이 시작하자마자 답을 출력한다면
게임으로서는 부족하니까요!

이제 프로그램을 몇 번 실행해 봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

매 실행마다 다른 숫자면서 1부터 100 사이의 숫자가 나타나야 합니다.
잘 하셨습니다!

## 비밀번호와 추리값을 비교하기

이제 우리는 입력값과 임의의 정수를 가지고 있음으로 비교가 가능합니다.
Listing 2-4는 그 단계를 보여주고 있습니다.
이 코드는 아직 컴파일 되지 않습니다. 그 이유는 곧 설명할 예정입니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

<span class="caption">Listing 2-4: 두 숫자를 비교한 결과 처리하기</span>

먼저 `use`를 구문을 하나 더 사용하여 표준 라이브러리로부터
`std::cmp::Ordering`이라는 타입을 가져옵니다. `Ordering`은 열거형이고
`Less`, `Greater`, `Equal`이라는 배리언트들을 가지고 있습니다.
이들은 여러분이 어떤 두 값을 비교할 때 나올 수 있는 세 가지 결과입니다.

그리고 나서 우리는 `Ordering` 타입을 이용하는 다섯 줄을
마지막에 추가 했습니다. `cmp` 메소드는 두 값을 비교하며
비교 가능한 모든 것들에 대해 호출할 수 있습니다.
이 메소드는 비교하고 싶은 것들의 참조자를 받습니다.
여기서는 `guess`와 `secret_number`를 비교하고 있습니다.
`cmp`는 `Ordering` 열거형을 돌려줍니다. 우리는
[`match`][match]<!-- ignore --> 표현문을 이용하여
`cmp`가 `guess`와 `secret_number`를 비교한 결과인
`Ordering`의 값에 따라 무엇을 할 것인지 결정할 수 있습니다.

`match` 표현식은 *arm* 으로 이루어져 있습니다.
하나의 arm은 하나의 *패턴* 과 `match` 표현식에서 주어진 값이 패턴과 맞는다면
실행할 코드로 이루어져 있습니다. 러스트는 `match`에게 주어진 값을
arm의 패턴에 맞는지 순서대로 확인합니다. `match` 생성자와 패턴들은
여러분의 코드가 마주칠 다양한 상황을 표현할 수 있도록 하고 모든 경우의 수를
처리했음을 확신할 수 있도록 도와주는 강력한 특성들입니다.
이 기능들은 6장과 18장에서 각각 더 자세히 다뤄집니다.

예제에서 사용된 `match` 표현식에 무엇이 일어날지 한번 따라가
봅시다. 사용자가 50을 예측했다고 하고 비밀번호가 38이라
합시다.

50과 38을 비교하면 `cmp` 메소드의 결과는 `Ordering::Greater` 입니다.
`match` 표현식은 `Ordering::Greater`를 값으로 받아서
각 arm의 패턴을 확인합니다. 처음으로 마주하는 arm의 패턴인
`Ordering::Less`는 `Ordering::Greater`와
매칭되지 않으므로 첫번째 arm은 무시하고 다음으로 넘어갑니다.
다음 arm의 패턴인 `Ordering::Greater`는 *확실히*
`Ordering::Greater`와 매칭합니다! arm과 연관된 코드가 실행될
것이고 `Too big!`가 출력될 것입니다. 이 경우 마지막 arm은
확인할 필요가 없으므로 `match` 표현식은
끝납니다.

하지만 Listing 2-4의 코드는 컴파일되지 않습니다. 한번 시도해 봅시다:

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

에러의 핵심은 *일치하지 않는 타입* 이 있다고 알려 주는 것입니다.
러스트는 강한 정적 타입 시스템을 가지고 있습니다. 하지만 타입 추론도 수행합니다.
만약 `let guess = String::new()`를 작성한다면 러스트는 `guess`가
`String`타입이어야 함을 추론할 수 있으므로 타입을 적으라고 하지 않습니다.
반대로 `secret_number`는 정수형입니다. 러스트의 숫자 타입 몇 가지가 1과 100 사이의
값을 가질 수 있습니다. `i32`는 32비트 정수, `u32`는 32비트의 부호없는 정수,
`i64`는 64비트의 정수이며 그 외에도 비슷합니다. 러스트는 기본적으로
우리가 다른 정수형임을 추론할 수 있는 다른 타입 정보를 제공하지 않는다면
숫자들을 `i32`으로 생각합니다. 이 에러의 원인은 러스트가
문자열과 정수형을 비교할 수 없기 때문입니다.

최종적으로 우리는 추리값을 정수형으로 비교하기 위해 입력으로 받은
`String`을 정수로 바꾸고 싶을 것입니다. 아래와 같은 한 줄을 `main` 함수에
추가하여 그렇게 합니다:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

이 라인은 다음과 같습니다:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

우리는 `guess` 변수를 생성했습니다. 잠깐, 이미 프로그램에서 `guess`라는
이름의 변수가 생성되지 않았나요? 그렇긴 하지만 러스트는 이전에 있던
`guess`의 값을 가리는 것을 허락합니다. *쉐도잉 (shadowing)* 은,
이를테면 `guess_str`과 `guess`와 같은 두 개의 고유한 변수를 만들도록 강제하기
보다 `guess` 라는 변수 이름을 재사용하도록 해줍니다. [3장][shadowing]<!-- ignore -->에서
더 자세한 이야기를 다루겠지만, 지금은 어떤 한 타입의 값을 다른 타입으로
바꾸고 싶을때 자주 사용되는 기능이라고만 알아두도록 하죠.

이 새로운 변수에 `guess.trim().parse()` 표현식을 묶습니다. 표현식 내에서
`guess`는 입력값을 가지고 있던 `String`을 참조합니다. `String` 인스턴스의
`trim` 메소드는 처음과 끝 부분의 공백문자들을 제거하는데, 이는 숫자형 데이터만
저장할 수 있는 `u32`와 문자열을 비교할 수 있게 하기 위해 꼭 필요합니다.
사용자들은 추리값을 입력한 뒤 `read_line`을 끝내기 위해
<span class="keystroke">enter</span>키를 반드시 눌러야 하고, 이것이
개행문자를 문자열에 추가시킵니다. 예를 들어 사용자가 <span class="keystroke">5</span>를
누르고 <span class="keystroke">enter</span>키를 누르면 `guess`는
`5\n`처럼 됩니다. `\n`은 “새줄”을 나타냅니다.
(Windows에서 <span class="keystroke">enter</span>는 캐리지 리턴과
개행, 즉 `\r\n`을 발생시킵니다.) `trim` 메소드는 `\n` 혹은 `\r\n`을
제거하고 `5`만 남도록 처리합니다.

[문자열의 `parse` 메소드][parse]<!-- ignore -->는
문자열을 숫자형으로 파싱합니다. 이 메소드는 다양한 종류의 정수형을
변환하므로 우리는 `let guess: u32`처럼 정확한 타입을 명시해야 합니다.
`guess` 뒤의 콜론(`:`)은 변수의 타입을 명시했음을 의미합니다.
러스트는 몇몇 내장된 정수형을 가지고 있습니다. `u32`은 부호가 없는
32비트의 정수입니다. 이 타입은 작은 양수를 표현하기에는 좋은 선택입니다.
[3장][integers]<!-- ignore -->에서 다른 숫자형에 대해 배울 것입니다.

추가로 이 예제 프로그램의 `u32` 명시와 `secret_number`와의 비교는
러스트가 `secret_number`의 타입 또한 `u32`이어야 한다고 유추할 것임을
의미합니다. 이제 이 비교는 같은 타입의 두 값의 비교가
됩니다.

`parse` 메소드의 호출은 에러가 발생하기 쉽습니다. 만약 `A👍%`과 같은
문자열이 포함되어 있다면 정수로 바꿀 방법이 없습니다.
[“`Result` 타입으로 잠재된 실패 다루기”](#handling-potential-failure-with-the-result-type)<!-- ignore-->
에서 `read_line`와 비슷하게 `parse` 메소드는 실패할 경우를 위해
`Result` 타입을 결과로 돌려 줍니다. 우리는 이 `Result`를
`expect` 메소드를 사용하여 같은 방식으로 처리합니다.
만약 `parse` 메소드가 문자열에서 정수로 파싱을 실패하여
`Err` `Result` variant를 돌려준다면 `expect` 호출은 게임을 멈추고
우리가 명시한 메세지를 출력합니다. 만약 `parse` 메소드가 성공적으로
문자열을 정수로 바꾸었다면 `Result`의 `Ok` variant를 돌려 받으므로
`expect`에서 `Ok`에서 얻고 싶었던 값을 결과로 받게
됩니다.

이제 프로그램을 실행해 봅시다:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

좋습니다! 추리값 앞에 빈칸을 넣더라도 프로그램은
추리값이 76임을 파악 했습니다. 추리값이 맞을 때나 너무 클 경우,
혹은 너무 작은 경우 등 여러 종류의 입력값으로
여러 시나리오를 검증해 봅시다.

우리는 게임의 대부분이 동작하도록 처리 했지만 사용자는 한 번의 추리만 가능합니다.
반복문을 추가하여 변경해 봅시다!

## 반복문을 이용하여 여러 번의 추리 허용

`loop` 키워드는 무한루프를 제공합니다. 루프를 추가하여 사용자들에게 숫자를
추리할 기회를 더 주겠습니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

우리는 추리값을 입력 받는 코드부터 모든 코드들을 반복문 내로 옮겼습니다.
각각의 라인이 4간격 더 들여쓰기 되어 있음을 확실히 하고 프로그램을 다시
실행 해 보세요. 이제 프로그램이 영원히 다른 추리값을 요청하고, 새로운
문제가 만들어졌군요. 사용자가 이 프로그램을 종료할 수 없어요!

사용자는 <span class="keystroke">ctrl-C</span> 단축키를 이용하여 프로그램을
멈출 수 있습니다. 하지만 [“비밀번호를 추리값과 비교하기”](#comparing-the-guess-to-the-secret-number)<!--
ignore -->에서 `parse` 메소드에 대해 논의할 때 언급한 방법으로
이 만족할 줄 모르는 괴물에게서 빠져나올 수 있습니다.
만약 사용자가 숫자가 아닌 답을 적는다면 프로그램이 멈춥니다.
사용자의 종료 허용을 위하여 이를 아래와 같이 활용할 수 있습니다:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread 'main' panicked at 'Please type a number!: ParseIntError { kind: InvalidDigit }', src/main.rs:28:47
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

`quit`를 입력하면 게임은 확실히 끝나지만, 눈치채신 것처럼 숫자 아닌 다른 입력값들도
마찬가지입니다. 하지만 이것은 최소한의 차선책입니다. 우리는 정답을 입력할 경우 자동으로
게임이 끝나도록 하고 싶습니다.

### 정답 이후에 종료하기

사용자가 정답을 맞췄을 때 게임이 종료되도록 `break`문을 추가합니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

`break`문을 `You win!` 이후에 추가하여 사용자가 비밀번호를 맞췄을 때 프로그램이
반복문을 끝내도록 합니다. 반복문이 `main`의 마지막 부분이므로 반복문의 종료는
프로그램의 종료를 의미합니다.

### 잘못된 입력값 처리하기

사용자가 숫자가 아닌 값을 입력했을 때 프로그램이 종료되는 동작을 더 다듬어
숫자가 아닌 입력은 무시하여 사용자가 계속 입력할 수 있도록 해 봅시다.
`guess`가 `String`에서 `u32`로 변환되는 라인을 수정하면 됩니다.
Listing 2-5에서 보이는 대로입니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

<span class="caption">Listing 2-5: 숫자가 아닌 추측일 입력했을 때 프로그램을
종료하는 대신 다른 추측을 요구하는 코드</span>

`expect` 메소드 호출을 `match` 표현식으로 바꾸어 에러 발생 시 크래시가 나지 않고
에러를 처리하도록 합니다. `parse` 메소드가 `Result` 타입을
돌려주는 것과 `Result`는 `Ok`나 `Err` variants를 가진 열거형임을 떠올리세요.
`cmp` 메소드의 `Ordering` 결과를 처리했을 때처럼 여기서 `match` 표현식을
사용하고 있습니다.

만약 `parse`가 성공적으로 문자열에서 정수로 변환했다면 결과값을 가진
`Ok` 를 돌려줍니다. `Ok`는 첫번째 arm의 패턴과 매칭하게 되고
`match` 표현식은 `parse` 가 생성한 `num`값을 돌려줍니다.
그 값은 우리가 생성하고 있던 새로운 `guess`과 묶이게
됩니다.

만약 `parse`가 문자열을 정수로 바꾸지 *못했다면*
에러 정보를 가진 `Err`를 돌려줍니다. `Err`는 첫번째 arm의 패턴인
`Ok(num)`과 매칭하지 않지만 두 번째 arm의 `Err(_)` 와 매칭합니다.
`_`은 모든 값과 매칭될 수 있습니다. 이 예시에서는 `Err`내에 무슨
값이 있던지에 관계없이 모든 `Err`를 매칭하도록 했습니다.
따라서 프로그램은 두 번째 arm의 코드인 `continue`를 실행하며,
이는 `loop`의 다음 반복으로 가서 또 다른 추리값을 요청하도록 합니다.
따라서, 프로그램은 효율적으로 `parse`에서 발생할지도 모르는
모든 에러를 무시합니다!

이제 우리가 원하는대로 프로그램이 작동해야 합니다. `cargo run`을 실행해 봅시다.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 4.45s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

멋집니다! 마지막에 조금 값을 조정하여 우리는 추리 게임을 끝냈습니다. 프로그램이
여전히 비밀번호를 출력하고 있다는 것을 떠올리세요. 테스트 때는 괜찮지만 게임을
망치게 됩니다. 비밀번호를 출력하는 `println!`을 삭제합니다.
Listing 2-6은 최종 코드를 보여줍니다.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

<span class="caption">Listing 2-6: 추리 게임의 완성된 코드</span>

이 시점에서 여러분은 성공적으로 추리 게임을 만들었습니다! 축하합니다!

## 요약

이 프로젝트는 `let`, `match`, 메소드, 연관함수, 외부 크레이트 사용과
같은 많은 새로운 러스트 개념들을 소개하기 위한 실습이었습니다. 다음
장들에서는 이 개념들의 세부적인 내용을 배울 것입니다. 3장은 대부분의
프로그래밍 언어들이 가지고 있는 변수, 데이터 타입, 함수를 소개하고
러스트에서의 사용법을 다룹니다. 4장에서는 다른 프로그래밍 언어와 차별화된
러스트의 특성인 소유권을 다룹니다. 5장에서는 구조체와 메소드 문법을
다루며 6장에서는 열거형에 대해 다룹니다.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: http://doc.crates.io
[doccratesio]: http://doc.crates.io/crates-io.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
