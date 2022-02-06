---
title: 함수형 도메인 모델링 - ADT
date: "2022-02-06T21:47:00+09:00"
series: 함수형 도메인 모델링
order: 1
---

<!-- series-toc -->

도메인 모델링에 쓸 수 있는 유용한 함수형 프로그래밍 기법들이 있는데 오늘은 그 중 대수적 자료형(Algebraic Data Type, 이하 ADT)을 소개합니다.

## 예제: 결제 시스템의 할인 유형

가상의 결제 시스템을 예로 들어보겠습니다. 두 가지 할인 유형이 있는데, 하나는 구매금액에 비례해서 퍼센트 단위로 할인을 해 주고, 나머지 하나는 정해진 액수만큼 할인을 해 줍니다.

```text
- 비율
  - 퍼센트 (정수)
- 정액
  - 액수 (실수)
  - 통화 (통화코드)
```

## 안티패턴: 테이블 주도 설계

요즘은 다른 선택지가 많아졌지만 웹 프로그래밍을 할 때는 여전히 RDBMS를 영속성을 위해 쓰는 것이 일반적입니다. ERD를 먼저 그리고 모델을 설계하다 보면 아래같은 구조가 나오는 경우가 많습니다.

```scala
enum DiscountType:
  case Percentage   // 퍼센트
  case FixedAmount  // 정액할인

class Discount(
  var discountType: DiscountType,
  var percentage: Option[Int],
  var fixedAmount: Option[BigDecimal],
  var fixedAmountCurrency: Option[Currency]
)
```

흔히 볼 수 있는 형태고 많은 경우 이대로도 잘 동작하지만, 이런저런 문제가 있습니다.

### 테이블 주도 모델의 문제점 1. 복잡도 증가

```scala
enum DiscountType:
  case Percentage
  case FixedAmount   

class Discount(
  var discountType: DiscountType,
  var percentage: Option[Int],
  var fixedAmount: Option[BigDecimal],
  var fixedAmountCurrency: Option[Currency]
)
```

```scala
C(Discount) = C(discountType) * C(percentage) * 
              C(fixedAmount) * C(fixedAmountCurrency)
            = C(DiscountType) C(Option[Int]) *
              C(Option[BigDecimal]) * C(Option[Currency])
            = 2 * 2 * 2 * 2
            = 16
```

`Discount`가 가질 수 있는 경우의 수는 16개

### 테이블 주도 모델의 문제점 2. 잘못된 상태를 표현 가능

```scala{1-6,10-11,15}
val valid = new Discount(
  discountType = FixedAmount,
  percentage = None,
  fixedAmount = Some(10.0),
  fixedAmountCurrency = Some(Currency.USD)
)

val invalid = new Discount(
  discountType = FixedAmount,
  percentage = Some(10),  /* 값이 없어야 함 */
  fixedAmount = None,     /* 값이 있어야 함 */
  fixedAmountCurrency = Some(Currency.USD)
)

valid.discountType = Percentage // ???
```

```scala{3,6}
def discount(subtotal: Money, d: Discount): Money =
  if (d.discountType == Percentage) {
    d.fixedAmount.get // FixedAmount?
    // ...
  } else if (d.discountType == FixedAmount) {
    d.percentage.get // Percentage?
    // ...
  }
```

### 유지보수할 때 생기는 일

- 특정 할인 유형에 새로운 필드 추가
  - 계산 과정에 반영하는걸 까먹었다!
  - 다른 할인 유형 계산식에 넣었다!
- 새로운 할인 유형 추가
  - `else if` 구문 추가를 잊었다!
    - 기존 코드에도 `else if` 대신 `else`가 있었다면!?

### 대수적 자료형 (ADT)

Algebraic Data Type, 합성 타입

#### Product Type

```scala
(3, "hello", true): (Int, String, Boolean)
```

#### Sum Type <!-- .element: style="margin-top: 1.0em;" -->

```scala
enum Weekday:
  case Mon, Tue, Wed, Thu, Fri, Sat, Sun
```