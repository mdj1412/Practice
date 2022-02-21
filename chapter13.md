# 13. 수치해석

## 13.1 도입
수치해석(numerical analysis)은 직접 풀기 힘든 수학 문제를 근사적으로 푸는 알고리즘과 이들의 수치적 안정성, 오차의 범위 등을 연구하는 전산학의 한 분야로, 공학, 과학, 금융공학 등 다양한 범위에 널리 사용된다.

<br><br><br><br>

## 13.2 이분법

<p align="center">
<img width="257" alt="스크린샷 2022-02-21 오후 2 59 50" src="https://user-images.githubusercontent.com/91893721/154897940-df024c3f-32d7-4bd4-a690-025e5dbc1df4.png">
<br><br>
  
```c++
// 코드 13.1 이분법의 예제 구현

#include <cmath>
#include <algorithm>

double f(double x);

// 이분법의 예제 구현
double bisection(double lo, double hi) {
    // 반복문 불변식을 강제한다.
    if (f(lo) > 0)
        std::swap(lo, hi);
    
    // 반복문 불변식: f(lo) <= 0 < f(hi)
    while (std::fabs(hi - lo) > 2e-7) {
        double mid = (lo + hi) / 2;
        double fmid = f(mid);

        if (fmid <= 0)
            lo = mid;
        else
            hi = mid;
    }

    // 가운데 값을 반환한다.
    return (lo + hi) / 2;
}
```
<br>
[ 코드 특징 ] <br>
  
- f(lo)와 f(hi)의 부혹 다르다는 조건 대신 f(lo)는 항상 0 이하이고 f(hi)는 항상 양수라는 반복문 불변식을 유지한다.
- f(x)가 0을 반환하는 경우를 딸 처리하지 않는다. f(lo)<=0<f(hi)라는 불변식만 유지한다면 항상 답이 수렴하기 때문이다.
- bisection()은 반복문이 모두 종료한 후에는 답의 후보 구간의 중간 값을 반환한다. 반복문 불변식에 의하면 우리가 원하는 답은 [lo,hi]안에 있는데, 이 중 최대 오차를 최소화할 수 있는 답이 중간 값이기 때문이다.
<br><br><br><br>
  
  
## 절대 오차 vs 상대 오차 (절대 오차의 문제점)

- 문제점: 부동 소수점은 가수부(mantissa)라고 부르느 정수 부분과 이 변수에서 소수점의 위치를 나타내는 지수부(exponent)의 조합으로 표현되기 때문에, 표현할 수 있는 수의 집합이 제한되어 있다. <br>
- 결론: 따라서 숫자의 절대 값이 커지면 커질수록 표현할 수 있는 수들이 듬성듬성해지게 된다. <br><br>

```c++
// 코드 13.2 영원히 종료하지 않는 이분법의 예

#include <cmath>
#include <iostream>

// 종료되지 않는다.
void infiniteBisection() {
    double lo = 123456123456.1234588623046875;
    double hi = 123456123456.1234741210937500;

    while (std::fabs(hi - lo) > 2e-7)
        hi = (lo + hi) / 2.0;
    printf("finished!\n");
}
```
[ 위 코드가 문제인 이유 ] <br>
lo와 hi 사이에는 double 변수가 표현할 수 있는 값이 하나도 없다. (사용하는 컴퓨터의 아키텍처나 컴파일러에 따라 달라질 수도 있다.) <br>
따라서 (lo+hi)/2는 결국 hi와 같은 값이 된다. 그런데 hi와 lo 사이의 거리는 10^-7보다 훨씬 크기 때문에, 이 함수(while문)는 절대로 종료되지 않는다. <br><br>

<p align="center">
<img width="1000" alt="스크린샷 2022-02-21 오후 3 29 00" src="https://user-images.githubusercontent.com/91893721/154900912-06565ee2-fbcf-4f66-b18e-c848b5432cde.png">

<br>
<p align="center">
<img width="700" alt="스크린샷 2022-02-21 오후 3 48 13" src="https://user-images.githubusercontent.com/91893721/154903179-b0b1dbaa-95b1-4335-a1d7-0dbb6b14fd80.png">
<p align="center">
<img width="700" alt="스크린샷 2022-02-21 오후 3 48 01" src="https://user-images.githubusercontent.com/91893721/154903171-30a2b4ac-65c6-4f18-9674-badcba24737e.png">

  
<br>
<p align="left">
[ 또 다른 문제점 ] <br>
단 이 조건은 lo와 hi가 전부 양수라고 가정한다. 두 수가 음수인 경우, lo는 음수이고 hi는 양수인 경우 등을 모두 고려하려면 코드느 더더욱 복잡해진다. <br> 따라서 경험 많은 프로그래밍 대회 참가자들은
이와 같은 방법을 쓰지 않는다.
<br><br><br><br>
  
## 정해진 횟수만큼 반복하기
  
단순해 보이지만 가장 유용한 방법은 while문을 적당한 for문으로 대체해서 반복문이 항상 정해진 횟수만큼 실행하도록 하는 것이다. <br> 반복문을 100번 수행하면 우리가 반환하는 답은 절대 오차는 최대 |lo-hi|/(2^101
)이 된다. 2^101은 대략 서른한 자리의 수로, |lo-hi|가 대략 10^20미만의 수라면 이 오차는 항상 10^-7보다 작다는 것을 알 수 있다. <br>
따라서 큰 숫자를 다루는 경우에도 충분히 답을 구할 수 있다. 또한 이와 같은 방법은 절대로 무한 반복에 빠지지 않으며, 프로그램의 최대 수행 시간을 예상하기도 쉽다는 장점이 있다.
<br><br><br><br>
##
