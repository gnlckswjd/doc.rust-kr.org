## `Sync`와 `Send` 트레잇을 이용한 확장 가능한 동시성

흥미롭게도, 러스트 언어는 *매우* 적은 숫자의 동시성 기능을 갖고 있습니다.
이 장에서 여지껏 이야기한 거의 모든 동시성 기능들이 언어의 부분이 아니라
표준 라이브러리의 영역이었습니다. 동시성 처리를 위한 옵션은 언어 혹은
표준 라이브러리에만 국한되지 않습니다; 여러분만의 동시성 기능을 작성하거나
다른 이들이 작성한 것을 이용할 수 있습니다.

그러나, 두 개의 동시성 개념은 언어에 내재되어 있습니다: 바로 `std::marker`
트레잇인 `Sync`와 `Send`입니다.

### `Send`를 사용하여 스레드 사이에 소유권 이전을 허용하기

`Send` 마커 트레잇은 `Send`가 구현된 타입의 소유권이 스레드 사이에서
이전될 수 있음을 나타냅니다. 거의 대부분의 러스트 타입이 `Send`이지만,
몇 개의 예외가 있는데, 그 중 `Rc<T>`도 있습니다: 이것은 `Send`가 될 수
없는데 그 이유는 여러분이 `Rc<T>` 값을 복제하여 다른 스레드로 복제본의
소유권 전송을 시도한다면, 두 스레드 모두 동시에 참조 카운트 값을 갱신할지도
모르기 때문입니다. 이러한 이유로, `Rc<T>`는 여러분이 스레드-안전성 성능
저하를 지불하지 않아도 되는 단일 스레드의 경우에 사용되도록
구현되었습니다.

따라서 러스트의 타입 시스템과 트레잇 바운드는 우발적으로 스레드 간에
`Rc<T>` 값을 불안전하게 보내질 수 없도록 보장해줍니다. Listing 16-14를
시도할 때는 `` 트레잇 `Send`가 `Rc<Mutex<i32>>` 에 대해 구현되지 않았습니다 ``
라는 에러를 얻었습니다. `Send`가 구현된 `Arc<T>`로 바꿨을 때는 코드가 컴파일
되었습니다.

또한 전체가 `Send` 타입으로 구성된 모든 타입은 자동적으로 `Send`로 마킹됩니다.
로우 포인터 (raw pointer)를 빼고 거의 모든 기초 타입이 `Send`인데, 이는
19장에서 다루겠습니다.

### `Sync`를 사용하여 여러 스레드로부터의 접근을 허용하기 

`Sync` 마커 트레잇은 `Sync`가 구현된 타입이 여러 스레드로부터 안전하게 참조
가능함을 나타냅니다. 바꿔 말하면, 만일 `&T` (`T`의 불변 참조자) 가 `Send`이면,
즉 참조자가 다른 스레드로 안전하게 보내질 수 있다면, `T`는 `Sync`합니다.
`Send`와 유사하게, 기초 타입들은 `Sync`하고, 또한 전체가 `Sync`한 타입들로
구성된 타입 또한 `Sync`합니다.

스마트 포인터 `Rc<T>`는 `Send`가 아닌 이유와 동일한 이유로 또한
`Sync`하지도 않습니다. (15장에서 이야기한) `RefCell<T>` 타입과
연관된 `Cell<T>` 타입의 가족들도 `Sync`하지 않습니다.
`RefCell<T>`가 런타임에 수행하는 빌림 검사 구현은 스레드-안전하지
않습니다. 스마트 포인터 `Mutex<T>`는 `Sync`하고 여러분이 [“여러 스레드 사이에서
`Mutex<T>` 공유하기”][sharing-a-mutext-between-multiple-threads]<!-- ignore -->절에서
본 것처럼 여러 스레드에서 접근을 공유하는데 사용될 수 있습니다.

### `Send`와 `Sync`를 손수 구현하는 것은 안전하지 않습니다

`Send`와 `Sync` 트레잇들로 구성된 타입들이 자동적으로 `Send` 될 수 있고
`Sync`하기 때문에, 이 트레잇들은 손수 구현하지 않아도 됩니다. 이들은 심지어
마커 트레잇으로서 구현할 어떠한 메소드도 없습니다. 이들은 그저 동시성과
관련된 불변성을 강제하는데 유용할 따름입니다.

이 트레잇들을 손수 구현하는 것은 안전하지 않은 (unsafe) 러스트 코드 구현을
수반합니다. 19장에서 안전하지 않은 러스트 코드에 대하여 이야기 하겠습니다;
지금으로서 중요한 정보는 `Send`와 `Sync`하지 않은 요소들로 구성된 새로운
동시적 타입을 만드는 것이 안전성 보장을 유지하기 위해 신중한 고려가 필요하다는
점입니다. [러스토노미콘][nomicon]에 이러한 보장과 유지하는 방법 대한 더 많은
정보가 있습니다.

## 정리

여기가 이 책에서 동시성에 대해 보게될 마지막은 아닙니다: 20장의 프로젝트에서는
여기서 다룬 작은 예제보다 더 실질적인 상황에서 이번 장에서 다룬 개념들을
이용하게 될 것입니다.

일찍이 언급한 것처럼, 러스트가 동시성을 처리하는 방법이 언어의 매우
작은 부분이기 때문에, 많은 동시성 솔루션이 크레이트로 구현됩니다.
이들은 표준 라이브러리보다 더 빠르게 진화하므로, 현재 가장 최신
기술의 크레이트를 온라인으로 검색해서 멀티스레드 상황에서 사용해
보세요.

러스트 표준 라이브러리는 메세지 패싱을 위해 채널을 제공하고, 동시적
맥락에서 사용하기 안전한 `Mutex<T>`와 `Arc<T>` 같은 스마트 포인터
타입들을 제공합니다. 타입 시스템과 빌림 검사기는 이 솔루션을 이용하는
코드가 데이터 경쟁 혹은 유효하지 않은 참조자로 끝나지 않을 것을 보장합니다.
일단 코드가 컴파일된다면, 다른 언어에서는 흔한 추적하기 어려운 버그 없이
여러 스레드 상에서 행복하게 동작하므로 안심할 수 있습니다.
동시성 프로그래밍은 더이상 두려워할 개념이 아닙니다: 앞으로 나아가
겁없이 여러분의 프로그램을 동시적으로 만드세요!

다음으로는 러스트 프로그램이 점차 커짐에 따라서 문제를 모델링하고 솔루션을
구조화하는 자연스러운 방법에 대해 이야기할 것입니다. 더불어 객체 지향 프로그래밍으로부터
친숙할지도 모를 개념들과 러스트의 관용구가 어떻게 연관되어 있는지 다루겠습니다.

[sharing-a-mutext-between-multiple-threads]:
ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
