---
layout: post
title: gettimeofday와 clock_gettime
key: 20191108_0
---

gettimeofday와 clock_gettime은 모두 시스템의 현재 시간을 측정할 때 쓰이는 함수이다.
주로 시각을 측정한 다음에 측정된 두 시각을 서로 빼서 함수 실행에 걸린 시간을 재는 등의 용도로 많이 사용된다.
gettimeofday는 마이크로초의 분해능을 clock_gettime은 나노초의 분해능을 제공한다.
이 외에는 큰 차이점이 없어 보이지만 사실 gettimeofday는 쓰면 안되는 함수다.
특히, 함수의 실행시간을 잰다면 더더욱이 쓰면 안된다.
그 이유를 살펴보자.
다음은 gettimeofday와 clock_gettime 함수와 각 함수에 쓰이는 구조체이다.

```C
#include <sys/time.h>

struct timeval {
        time_t      tv_sec;     /* seconds */
        suseconds_t tv_usec;    /* microseconds */
};

struct timezone {
        int tz_minuteswest;     /* minutes west of Greenwich */
        int tz_dsttime;         /* type of DST correction */
};

int gettimeofday(struct timeval *tv, struct timezone *tz);
```

gettimeofday는 현재 시스템의 시간을 timeval 구조체에 초단위, 마이크로초단위로 적고 return한다.
timezone 구조체는 일광 절약 시간제를 고려해 생긴 구조체인데 원체 쓰이질 않아 버려졌다.

```C
#include `<time.h`>

int clock_gettime(clockid_t clk_id, struct timespec &#42;tp);

struct timespec {
        time_t   tv_sec;        /* seconds */
        long     tv_nsec;       /* nanoseconds */
};
```

clock_gettime도 gettimeofday와 마찬가지로 비슷하게 동작한다.
timespec 구조체에 초단위, 나노초단위로 현재 시간을 적고 return한다.
다만 gettimeofday와 다른 인자가 하나 있는데 clk_id이다.
clk_id는 시간의 측정방식을 지정해준다.
예를 들어, clk_id 모드에는 CLOCK_REALTIME, CLOCK_MONOTONIC이 존재한다.

CLOCK_REALTIME : 현재 시간을 측정하지만 실제 현실세계의 시간인 realtime clock을  사용한다
CLOCK_MONOTONIC : 시스템의 realtime clock이 아니라 monotonic clock을 이용한다.

좀 더 자세히 설명하자면 CLOCK_REALTIME은 실제 시간을 반영하기 때문에 시간의 조정이 반복적으로 일어나는 시스템의 realtime clock을 이용해서 현재시간을 측정한다.
손목시계를 보고 달리기 선수들이 100m를 달리는 시간을 측정한다고 생각해보자.
손목시계는 시계가 갑자기 다른 시간으로 건너뛰거나 하는 일 없이 일정하게 초침이 움직이며 현재 시간을 보여준다.
따라서 선수들이 출발했을때 시간과 결승점에 골인했을때 시간의 차이를 구하면 100m를 뛰는데 걸린 시간을 구할 수 있다.
그런데 이 시계가 스마트워치라 인터넷에 연결되있어서 내 시간을 계속 보정한다고 생각해보자.
스마트워치 자체의 시간이 틀리면 이를 보정하기 위해 시간이 갑자기 건너뛰는 현상이 발생한다.
문제는 그렇게 되면 현재 시간을 정확히는 알 수 있어도 어떤 사건이 일어난 시간을 정확히 잴 수는 없는 것이다.

CLOCK_REALTIME을 이용하면 그런일이 일아난다. 
그래서 자신이 무언가의 실행시간을 잰다면 CLOCK_MONOTONIC 모드를 써야만 한다!

gettimeofday는 이러한 모드를 지정할 수도 없고 항상 CLOCK_REALTIME 모드로 동작한다.
게다가 오래전부터 버려진 timezone 구조체를 사용하는 것만 봐도 이 함수가 얼마나 오래됐는지 알 수 있다.
실제로 gettimeofday는 더 이상 유닉스 표준이 아니다.

SVr4, 4.3BSD. POSIX.1-2001 describes gettimeofday() but not settimeofday(). POSIX.1-2008 marks gettimeofday() as obsolete, recommending the use of clock_gettime(2) instead. --from gettimeofday manpage


따라서 gettimeofday는 쓰지 않는 것이 좋고 자신이 무언가의 실행시간을 잰다면 더더욱 쓰면 안된다!
내가 주장하는 게 아니라 linux manpage에 그렇게 써있다.
쓰지말라는 건 왠만하면 쓰지 말도록 하자.
