# 9장

### 일관성 보장

복제 데이터베이스에서는 각 노드에 쓰기 요청이 도착하는 시간이 다르기 때문에 불일치가 발생할 수 있다.

따라서 대부분의 경우에서 최종적 일관성을 제공한다. 이는 쓰기를 멈추고 특정 시간 동안 기다리면 결국 모든 읽기 요청이 같은 값을 반환한다는 의미이다.

하지만 이와 같은 보장은 매우 약한 보장인데, 그 이유는 **언제** 복제본이 수렴될 지에 대해 정해지지 않기 때문이다.

### 선형성

기본 아이디어는 시스템에 데이터 복사본이 하나만 있고 그 데이터를 대상으로 수행하는 모든 연산은 원자적인 것처럼 보이게 만드는 것이다. 다시 말해 **최신성 보장 이다.**

연산 표시를 모은 선이 항상 시간순으로 진행되어야 하고 결코 뒤로 가서는 안된다.

**직렬성** 은 선형성과 혼동하기 쉽지만 이 둘은 서로 다른 보장임
모든 트랜잭션이 여러 객체를 읽고 쓸 수 있는 상황에서 트랜잭션들의 격리 속성으로, 직렬성은 트랜잭션들이 어떤 순서에 따라 실행되는 것처럼 동작하도록 보장해주는 속성을 말한다.

**선형성**은 레지스터에 실행되는 읽기와 쓰기에 대한 최신성 보장을 말한다.

선형성이 유용한 상황

- 잠금을 사용하는 경우 (혹은 리더 선출)
- 제약 조건과 유일성 보장이 필요한 경우

부가적인 통신 채널이 존재하게 되면 타이밍에 따라 선형성을 위반하는 경우가 발생할 수 있다.

실제로 선형적인 시스템은 매우 드물다.
보통 선형성을 제거하면서 성능을 얻고자 하기 때문에, 대부분의 경우에서 선형적인 시스템이 제공되지 않는다.

### 분산 트랜잭션과 합의

합의의 목적은 **여러 노드들이 뭔가에 동의하게 만드는 것**이다.
노드가 동의하는 것이 중요한 상황들이 존재하는데, **리더 선출**, **원자적 커밋** 과 같은 것이 대표적인 예시이다.

> 합의 불가능성

어떤 노드가 죽을 위험이 있다면 항상 합의에 이를 수 있는 알고리즘은 없다는 것을 증명할 수 있음. 하지만 알고리즘이 타임아웃을 쓰는 게 허용되거나 죽은 것으로 의심되는 노드를 식별하는 다른 방법이 있다면 합의는 해결 가능하다.

단일 노드에서 트랜잭션 커밋은 데이터가 디스크에 지속성 있게 쓰여지는 순서에 결정적으로 의존한다.
즉, 트랜잭션이 커밋되거나 어보트되는 핵심적인 시점은 디스크가 커밋 레코드를 마치는 시점이다.
⇒ 그 시점 전에는 어보트 될 가능성이 있지만, 이후에는 커밋된 상태가 된다. (노드가 죽더라도)

여러 노드가 참여하는 경우에는 트랜잭션에 참여하는 다른 모든 노드도 커밋될 것이라는 확신이 있을 때에만 커밋이 되어야 한다.

### 2단계 커밋

여러 노드에 걸친 원자적 트랜잭션 커밋을 달성하는 알고리즘을 말한다.

보통 코디네이터를 사용하여 준비요청을 통해 커밋할 수 있는 상태를 물어본 뒤, 커밋 혹은 어보트 상태를 결정한다.

코디네이터에 장애가 발생하여 트랜잭션이 커밋되었는 지 여부를 확인할 수 없을 때 **in doubt** 혹은 **uncertain** 하다고 한다. 이 경우에는 어쩔 수 없이 코디네이터가 장애에서 복구될때까지 기다려야 한다.

2PC 는 코디네이터 장애 상황에서는 블록킹 상태에 빠지게 된다는 단점이 생긴다. 이에 대한 대안으로 3PC 라는 알고리즘이 존재하는데 이는 네트워크와 응답 시간에 제한이 있는 노드를 가정한다. 즉 다시말해 노드가 죽었는지 아닌지를 판별하는 신뢰성 있는 메커니즘이 존재하는 경우에만 가능한 알고리즘이다.

### XA 트랜잭션

서로 다른 데이터베이스에 걸친 2단계 커밋을 구현하는 표준.

데이터베이스는 트랜잭션이 커밋 혹은 어보트를 할때까지 잠금을 해제할 수 없다. 만약 **in doubt** 상태에 빠진다면 장애에서 회복될 때 까지 잠금을 해제할 수 없게 된다. 그러면 해당 row 에 대한 접근이 제한될 수 있다.
또한 완벽한 상태를 유지할 수 있다는 보장이 없기 때문에 **고아** 상태에 빠진 트랜잭션이 있을 수 있는데, 이 경우에는 수동으로 이를 해소하는 수 밖에 없다. 아니면 경험적으로 이 상태를 결정할 수도 있다.

내결함성을 지닌 합의 알고리즘은 다음 속성을 만족해야 한다.

- 균일한 동의 → 어떤 두 노드도 다르게 결정하지 않는다.
- 무결성 → 어떤 노드도 두 번 결정하지 않는다.
- 유효성 → 한 노드가 v 를 결정한다면, v 는 어떤 노드에서 제안된 것이다.
- 종료 → 죽지 않은 모든 노드는 결국 어떤 값을 결정한다.

여기서 내결함성에 대한 요구는 **종료 에 달려있는데, 어떤 노드가 죽더라도 다른 노드들은 여전히 결정을 내려야 하기 때문에 리더에 장애가 발생하더라도 합의 알고리즘은 계속되어야 한다는 속성이 담긴 것이다.**

에포크 번호를 정의하고 각 에포트 내에서 리더는 유일하다고 보장할 수 있다.

이 경우 현재 리더가 죽었다고 판단이 되면 새 노드를 선출하기 위해 투표를 수행한다. 
리더 선출은 에포크 번호를 증가시키며, 이 번호는 전체 순서가 있고 단조증가한다. 만약 리더 사이에 충돌이 있으면 에포크 번호가 더 높은 리더가 이긴다.