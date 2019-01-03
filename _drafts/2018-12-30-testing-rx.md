---
title: "Testing Rx"
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
총 5개의 값을 받았고 각 값을 받는데 1초가 걸리는 테스트를 작성하였을때, 실행되는데 걸리는 시간은 총 5초입니다. 이것은 좋지않습니다; 몇천개 까지는 아니더라도, 몇백개의 테스트가 5초안에 돌았으면 합니다. 아주 일반적인 다른 요구사항으로 timeout을 테스트 하는 것이 있습니다. 여기 1분의 timeout을 테스트 하는 코드가 있습니다.
```csharp
var never = Observable.Never<int>();
var exceptionThrown = false;
never.Timeout(TimeSpan.FromMinutes(1))
     .Subscribe(
        i => Console.WriteLine("This will never run."),
        ex => exceptionThrown = true);
Assert.IsTrue(exceptionThrown);
```
여기에는 두가지 문제가 있습니다:
1. Assert가 너무 빨리 실행되어, 테스트가 항상 실패하여 무의미 하거나,
2. 정확한 테스트를 위해 실제로 1분의 딜레이를 추가 하였습니다.

이 테스트가 유용하게 쓰이기 위해서는, 결과적으로 실행되는데 1분이 걸립니다. 실행하는데 1분이 걸리는 유닛테스트는 받아들일 수 없습니다.




## Reference
- [Introduction to Rx - Testing Rx](http://introtorx.com/Content/v1.0.10621.0/16_TestingRx.html#TestingRx)

*공부를 위해 번역한 포스트입니다. 오역은 [깃허브](http://github.com/uniqmuz/uniqmuz.github.io)나 댓글로 알려주세요 :)

