## 성능 비교하기: 루프 vs. 반복자

루프와 반복자 중 무엇을 사용할지 결정하기 위해서는 어떤 쪽에 더
빠른지 알 필요가 있겠습니다: 명시적으로 `for` 루프를 사용한 `search`
함수 버전과 반복자 버전 중 말이지요.

우리는 아서 코난 도일이 쓴 *셜록 홈즈의 모험*의 전체
내용을 로딩하고 내용 중에 *the*를 찾는 벤치마크를 돌렸습니다.
아래에 루프를 사용한 `search` 버전과 반복자를 사용한 버전에
대한 벤치마크 결과가 있습니다:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

반복자 버전이 약간 더 빨랐군요! 여기서는 벤치마크 코드에 대해 설명하진 않을 것인데,
왜냐하면 여기서의 핵심은 두 버전이 동등하다는 것을 증명하는 것이 아니고,
두 구현이 성능 측면에서 얼마나 비교되는지에 대한 일반적인 감을 얻는 것이기
때문입니다.

더 종합적인 벤치마크를 위해서는 다양한 크기의 다양한 텍스트를
`contents`로 사용하고, 서로 다른 길이의 다양한 단어들을 `query`로
사용하여 모든 종류의 다른 조합으로 확인해야 합니다. 요점은 이렇습니다:
반복자는 비록 고수준의 추상화지만, 컴파일되면 대략 직접 작성한 저수준의
코드와 같은 코드 수준으로 내려갑니다. 반복자는 러스트의 *비용 없는 추상화 (zero-cost abstraction)*
중 하나이며, 그 추상을 사용하는 것은 추가적인 런타임 오버헤드가
없다는 것을 의미합니다. 최초의 C++ 디자이너이자 구현자인 비야네 스트롭스트룹이
“Foundations of C++” (2012)에서 *제로 오버헤드 (zero-overhead)*를
정의한 것과 유사합니다:

> 일반적으로 C++ 구현은 제로 오버헤드 원칙을 준수합니다: 사용하지 않는 것에
> 대해서는 비용을 지불하지 않습니다. 그리고 더 나아가서, 사용하는 것에 대해서는
> 더 나은 코드를 제공할 수 없습니다.

또 다른 예로, 다음 코드는 오디오 디코더에서 가져왔습니다. 디코딩
알고리즘은 선형 예측이라는 수학적 연산을 사용하여 이전 샘플의 선형
함수에 기반해서 미래의 값을 추정합니다. 이 코드는 반복자 체인을
사용해서 스코프에 있는 세 개의 변수로 수학 연산을 합니다: 데이터의
`buffer` 슬라이스, 12개의 `coefficients` 배열, 그리고 데이터를
쉬프트 하기 위한 `qlp_shift` 값으로 말이죠. 이 예제에서는 변수를
선언했지만 값을 주지는 않았습니다; 비록 이 코드가 문맥 밖에서는
큰 의미 없지만, 러스트가 어떻게 고수준의 개념을 저수준의 코드로
변환하는지에 대한 간결하고 실질적인 예제입니다.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

`prediction`의 값을 계산하기 위해서, 이 코드는 `coefficients`에 있는
12개의 값을 순회하면서 `zip` 메소드를 사용하여 각 계수와 `buffer`의 이전
12개의 값의 쌍을 만듭니다. 그런 다음, 각 쌍에 대하여 값들을 서로 곱하고,
모든 결과를 더한 다음, 더한 값의 비트를 `qlp_shift` 비트만큼 우측으로
쉬프트합니다.

오디오 디코더와 같은 애플리케이션에서의 계산은 종종 성능에 가장 높은 우선순위를
둡니다. 여기서는 반복자를 만들고, 두 개의 어댑터를 사용하고, 값을 소비하고
있습니다. 이 러스트 코드가 컴파일되면 어떤 어셈블리 코드가 될까요? 글쎄요,
이 글을 쓰는 시점에서는 직접 손으로 작성한 것과 같은 어셈블리 코드로 컴파일됩니다.
`coefficients`의 값들을 순회하기 위해 동반되는 어떠한 루프도 없습니다:
러스트는 12번의 반복이 있다는 것을 알고 있으므로, 루프를 "풀어(unrolls)"
놓습니다. *언롤링(Unrolling)* 은 루프 제어 코드의 오버헤드를 제거하고
대신 루프의 각 순회에 해당하는 반복되는 코드를 생성하는 최적화
방법입니다.

모든 계수들은 레지스터에 저장되는데, 이는 값에 대한 접근이 매우 빠르다는
것을 의미합니다. 런타임에 배열 접근에 대한 경계 체크가 없습니다.
러스트가 적용할 수 있는 이러한 모든 최적화들은 결과적으로 코드를
아주 효율적으로 만듭니다. 이제 이것을 알게 되었으니, 반복자와 클로저를
무서워하지 않고 사용할 수 있습니다! 이것들은 코드를 고수준으로 보이도록
하지만, 그렇게 하기 위해 런타임 성능 저하를 강제하지는 않습니다.

## 정리

클로저와 반복자는 함수형 프로그래밍 아이디어에서 영감을 받은 러스트의
기능들입니다. 이들은 고수준의 개념을 저수준의 성능으로 명확하게 표현해주는
러스트의 능력에 기여하고 있습니다. 클로저와 반복자의 구현은 런타임 성능에
그렇게 영향을 주지 않습니다. 이는 비용 없는 추상화를 제공하기 위해 노력하는
러스트 목표의 일부분입니다.

이제 우리 I/O 프로젝트의 표현력을 개선했으니, 이런 프로젝트를
세상과 공유하는데 도움을 줄 `cargo` 의 몇가지 기능들을
살펴봅시다.
