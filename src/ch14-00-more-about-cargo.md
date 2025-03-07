# Cargo와 Crates.io 더 알아보기

지금까지는 빌드, 실행, 코드 테스트 등 Cargo의 가장 기본적인 기능만
사용하였지만, Cargo 는 훨씬 더 많은 일을 할 수 있습니다. 이번 장에서는
아래 목록의 기능을 수행하는 몇가지 고급 기능들을 알아보도록 하겠습니다:

* 릴리즈 프로필을 통한 빌드 커스터마이징하기
* [crates.io](https://crates.io)<!-- ignore --> 에 라이브러리 배포하기
* 대규모 작업을 위한 작업공간 (workspace) 구성하기
* [crates.io](https://crates.io)<!-- ignore --> 로부터 바이너리 설치하기
* 커스텀 명령어로 Cargo 확장하기

Cargo는 이번 장에서 다루는 것보다 더 많은 일을 할 수 있으니,
Cargo의 모든 기능에 대한 설명을 보고 싶으시다면
[공식 문서](https://doc.rust-lang.org/cargo/)를 참고하세요.
