# 자동화 테스트 작성하기

에츠허르 다익스트라(Edsger W. Dijkstra)는 1972년 자신의 에세이 '겸손한 프로그래머' (*The Humble Programmer*)에서
"프로그램 테스트는 버그의 존재를 보여주는 데에는 매우 효율적인 방법일 수 있지만, 버그의 부재를 보여주기에는 절망적으로 부적절하다."
(*Program testing can be a very effective way to show the presence of bugs, but it is hopelessly inadequate for showing their absence.*)
라고 말했습니다. 하지만 이 말은 우리가 테스트를 할 필요가 없다는 뜻이 아닙니다!

프로그램의 정확성은 곧 '프로그램이 얼마나 우리가 의도한 대로 작동하는가'와 같습니다.
러스트는 프로그램의 정확성에 굉장히 신경을 써서 설계된 언어지만,
정확성을 증명하기란 어렵고 복잡합니다. 러스트 타입 시스템이 이 역할의
큰 부담을 해소해주고 있으나 타입 시스템만으로 모든 문제를 잡아내지는 못합니다.
따라서 러스트는 언어 자체적으로 자동화 소프트웨어 테스트 작성을 지원합니다.

전달받은 숫자에 2를 더하는 `add_two` 함수를 작성한다고 칩시다.
함수 시그니처는 매개변수로 정수를 전달받고, 결과로 정수를 반환합니다.
우리가 이 함수를 구현하고 컴파일할 때 러스트는
우리가 앞서 배운 타입 검사 및 borrow 검사를 수행합니다.
함수에 `String` 값이나 유효하지 않은 참조자가 전달될 일이 없도록 보장해주죠.
하지만 러스트는 함수가 우리 의도대로 작동하는지는 검사할 수 *없습니다.*
함수가 매개변수에 2를 더하지 않고, 10을 더하거나 50을 빼서 반환해도 모를
일입니다! 이런 경우에 테스트를 도입합니다.

`add_two` 함수에 `3` 을 전달하면 `5` 가 반환될 것임을 단언(assert)하는 테스트를 작성하고
코드를 수정할 때마다 테스트를 실행하면, 제대로 작동하던 기존 코드에
문제가 생기지 않았을지 걱정할 필요가
사라집니다.

테스트는 복잡한 기술입니다. 좋은 테스트를 작성하는 방법을 이번 장 내에서
하나부터 열까지 전부 다룰 수는 없습니다. 이번 장은 러스트의 테스트 메커니즘을 설명합니다.
테스트 작성 시 사용하는 어노테이션, 매크로를 배우고,
테스트 실행 시의 기본 동작과 실행 옵션, 유닛 테스트와
통합 테스트를 조직화하는 방법을 배워보도록 하죠.
