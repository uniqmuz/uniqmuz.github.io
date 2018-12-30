---
title: "Testing Rx?"
categories:
    - Rx
tags:
    - rx
    - reactive
    - translated
toc: true
---
## Testing Rx
Software를 테스트하는 것은 코드의 디버깅과 시연의 기본입니다. 기존의 수동 테스트는 그저 "프로그램을 부스는" 것을 시도하지만, 현대적인 품질 관리는 버그를 평가하고 예방하는데 도움이되는 자동화 수준을 필요로 합니다. 테스트 스페셜리스트 팀이 보편화 되는 동안, 더 많은 코더들이 자동화된 테스트 모음을 통해 품질 보증 제공하는 것을 기대하고 있습니다. 

여전히 많은 개발자들이 테스트 작성하는 것 없이 코딩하는 것을 바라지 않을 것입니다. 테스트는 그 코드가 실재로 요구사항을 만족하는지 증명되어야 하며, 회귀에 대해 안전하도록 제공 되어야하고, 심지어 코드의 문서화에도 도움 될 수 있습니다. 이 챕터는 당신이 dependency injection 및 mock이나 stub같은 test-double을 활용한 단위 테스트에 익숙하다는 전제하에 작성되었습니다. 

Rx에 대한 몇가지 흥미로운 문제가 우리의 Test-Driven 커뮤니티에서 제기되었습니다:
- 스케쥴링 및 스레딩은 일반적으로 race condition을 발생시킬 수 있기 때문에 테스트 시나리오에서 피해야합니다. 
- 테스트는 가능한 빠르게 실행되어야 합니다.
- 다수에게 Rx는 새로운 기술/라이브러리 입니다. 당연히 Rx를 마스터하기 위해, 이전 Rx코드를 리팩토링 할 수 있습니다. 이 리팩토링이 내부동작을 변경하지 않았는지 확인하기 위해서 테스트를 사용하고자 합니다. 

코드를 테스트 하고자 할 때, 느리거나 비-결정적 테스트를 도입하기를 원하지 않습니다. 사실, 이후에 false-negatives 나 false-positives를 도입할 예정입니다. Rx 라이브러리를 살펴보면, (암시적 또는 명시적으로) 스케쥴링을 포함하는 많은 메소드가 있습니다. 그래서 Rx를 효율적으로 쓰게되면 스케쥴링을 피할 수 없게됩니다. 이 LINQ 쿼리는 IScheduler를 파라미터로 허용하는 확장 메소드가 26개 이상 있음을 보여줍니다.
```csharp
var query = from method in typeof(Observable).GetMethods()
  from parameter in method.GetParameters()
  where typeof (IScheduler).IsAssignableFrom(parameter.ParameterType)
  group method by method.Name into m
  orderby m.Key
  select m.Key;
foreach (var methodName in query)
{
  Console.WriteLine(methodName);
}
```
Output:
```
Buffer
Delay
Empty
Generate
Interval
Merge
ObserveOn
Range
Repeat
Replay
Return
Sample
Start
StartWith
Subscribe
SubscribeOn
Take
Throttle
Throw
TimeInterval
Timeout
Timer
Timestamp
ToAsync
ToObservable
Window
```
이런 메소드 중 대부분은 IScheduler를 사용하지 않는 대신, 기본 인스턴스를 사용하는 오버로드를 가집니다. TDD/Test를 우선시하는 코더들은 IScheduler를 허용하는 오버로드를 선택하여 테스트에서 스케쥴링을 제어하고자 할 것입니다.

이 예제에서는 5초 동안 매 초마다 값을 게시하는 시퀀스를 만듭니다.
```csharp
var interval = Observable
.Interval(TimeSpan.FromSeconds(1))
.Take(5);
```
5개의 값을 받은 것과 1초 간격으로 실행하는데 총 5초가 걸리는 것을 보증하는 테스트 코드를 작성한다면, 잘 되지 않을 것이다;
## Reference
- [Introduction to Rx - Why Rx](http://introtorx.com/Content/v1.0.10621.0/01_WhyRx.html#WhyRx)

*오역은 [깃허브](http://github.com/uniqmuz/uniqmuz.github.io)나 댓글로 알려주세요 :)