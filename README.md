# first come first served event

### 요구사항 정의

> 선착순 100명에게 할인쿠폰을 제공하는 이벤트를 진행하고자 한다.
>
> 
>
> 이 이벤트는 아래와 같은 조건을 만족하여야 한다.
>
> - 선착순 100명에게만 지급되어야 한다.
> - 101개 이상이 지급되면 안된다.
> - 순간적으로 몰리는 트래픽을 버틸 수 있어야 한다.
> - 유저는 하나의 쿠폰만 발급받을 수 있다. (중복 X)

------

### Race Condition (경쟁 상태)

두 개의 쓰레드가 공유 데이터에 access를 하고 동시에 작업을 하려고 할 때 발생하는 문제

**[예상]**

| Thread-1                    | Coupon count | Thread-2                    |
| --------------------------- | ------------ | --------------------------- |
| select count(*) from coupon | 99           |                             |
| create coupon               | 100          |                             |
|                             | 100          | select count(*) from coupon |
|                             | 100          | failed create coupon        |

Thread-1이 생성된 쿠폰의 개수를 가져가고 아직 100개가 아니므로, 쿠폰을 생성하고

Thread-2가 생성된 쿠폰의 개수를 가져갔을 때 생성된 쿠폰의 개수가 100개이므로 쿠폰을 발급하지 않음

**[동작]**

| Thread-1                    | Coupon count | Thread-2                    |
| --------------------------- | ------------ | --------------------------- |
| select count(*) from coupon | 99           |                             |
|                             | 99           | select count(*) from coupon |
| create coupon               | 100          |                             |
|                             | 101          | create coupon               |

실제로는 Thread-1이 생성된 쿠폰의 개수를 가져가고 Thread-1이 쿠폰을 생성하기 전에 Thread-2가 생성된 쿠폰의 개수를 가져감

Thread-2가 가져가는 쿠폰의 개수가 99개가 되고, 쿠폰을 생성하게 됨.

### Race Condition 해결

- synchronized 키워드 사용 (X)

  - 서버가 여러 대가 된다면 적절하지 않은 해결 방법
- MySQL, Redis를 활용한 락 구현 (X)

  - 쿠폰 개수의 정합성을 위해 발급된 쿠폰 개수를 가져오는 것 부터 생성까지 락을 걸어야 함.
  - 따라서 성능에 불이익이 있을 수 있음
- Redis의 `INCR` 명령어 사용 (△)

  - 레디스는 싱글 쓰레드 기반으로 동작하여 레이스 컨디션을 해결할 수 있음
  - `INCR `명령어는 성능이 빠른 명령어임 => O(1)
  - 따라서 `INCR`을 활용하여 발급된 쿠폰 개수를 제어한다면 성능도 빠르고 데이터 정합성을 지킬 수 있음.
- 하지만, 발급하는 쿠폰의 개수가 많아지면 많아질수록 RDB에 부하를 주게 됨. (RDB가 쿠폰 전용 DB가 아니라 다양한 곳에서 사용하고 있었다면 다른 서비스까지 영향을 줘서 장애를 일으킬 수 있음)
