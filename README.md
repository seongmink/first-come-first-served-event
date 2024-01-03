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
