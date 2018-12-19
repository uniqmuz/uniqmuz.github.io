---
title: "When is Rx appropriate?"
categories:
    - Rx
tags:
    - rx
    - reactive
toc: true
---

*Reference : [Introduction to Rx - Why Rx](http://introtorx.com/Content/v1.0.10621.0/01_WhyRx.html#WhyRx)

*오역은 [깃허브](http://github.com/uniqmuz/uniqmuz.github.io)에 이슈로 등록 부탁드립니다.

# 언제 Rx를 쓸 것인가?
Rx는 **이벤트 시퀀스**를 처리하기 위한 **자연스러운 패러다임**을 제공합니다. 시퀀스 하나에는 0개 이상의 이벤트가 포함됩니다. Rx는 이 시퀀스들을 조합할 때 가장 빛을 발합니다.

## Rx를 꼭 써야 하는 경우
아래와 같이 이벤트를 관리하는 것들이 Rx를 사용하기에 좋습니다:
- Mouse move나 button click 같은 **UI 이벤트**
- 속성 변경, 컬렉션을 업데이트, "주문 완료", "등록 승인"과 같은 도메인 이벤트
- File watcher나 system / WMI 이벤트같은 인프라스트럭쳐 이벤트
- 메세지 버스나 Websocket API의 push event 또는 low latency 미들웨어(e.q Nirvana) 같은 통합 이벤트
- StreamInsight 나 StreamBase같은 CEP(Complex Event Processing) 엔진과의 통합

흥미롭게도, SQL Server 군의 일부인 Microsoft의 CEP 제품 StreamInsight는 데이터 스트리밍 이벤트에 대한 쿼리를 만드는데 LINQ를 사용합니다.

Rx는 또한 offloading목적으로 **동시성**을 도입하고 관리하기에 매우 적합합니다. 즉, 주어진 일련의 작업을 동시에 수행하여 현재 스레드로 부터 자유로워집니다. 이것은 반응형 UI를 유지관리 하는데 주로 사용됩니다.

동작중인 데이터를 모델링하려고 하는경우, IEnumerable<T>가 존재한다면 Rx의 사용을 고려해 보아야합니다. IEnumerable<T>가 동작중인 데이터를 모델링 할 수 있을 때(yield return 과 같은 lazy evalutaion을 사용할때), 그것은 아마 확장할 수 없을 것입니다.
IEnumerable<T>를 순환하는것은 스레드를 소비/블록 하게 될 것입니다. 
Rx의 non-blocking 특성인 IOberservable<T>이나 .NET 4.5의 async 특성을 고려하여 사용하여야 합니다.

## Rx를 사용 가능한 경우
Rx는 **비동기 호출**에도 사용될 수 있습니다. 이것들은 실제로`effectively` 한 이벤트에 대한 시퀀스입니다.
- Task 나 Task<T>의 결과
- FileStream의 BeginRead/EndRead와 같은 [APM][APM_link] (Asynchronous Programming Model) 메서드 호출의 결과

[TPL][TPL_link] (Task Parallel Library)나 Dataflow 또는 async 키워드(.NET 4.5)를 사용하는 것이 비동기 메서드를 구성하는 더 자연스러운 방법임을 알 수 있습니다. Rx가 이러한 시나리오를 확실히 도울 수 있지만, 만약 더 적합한 다른 프레임워크가 있다면 먼저 고려해 보아야합니다.

병렬 계산을 수행하거나 스케일링을 할 목적으로 동시성을 도입하고 관리하려는 경우, Rx가 사용될 수 있지만 적합하지는 않습니다. TPL이나 C++ AMP가 이러한 집중적인 병렬 계산에 더 적합합니다.

## Rx를 사용할 수 없음
Rx와 IObservable<T>는 **IEnumerable<T>을 대체할 수 없습니다.**
- 단지 코드 베이스가 "더 많은 Rx"가 되도록 이미 존재하는 IEnumerable<T> 값을 IOberservable<T>로 변환하는 것
- 메세지 큐, MSMQ 나 JMS같은 큐는 기본적으로 트랜잭션성을 가지며 정의상으로도 순차적입니다. 여기에는 이미 IEnumerable<T>이 녹아들어있습니다.

[APM_link]: https://docs.microsoft.com/ko-kr/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm

[TPL_link]: 
https://docs.microsoft.com/ko-kr/dotnet/standard/parallel-programming/task-parallel-library-tpl