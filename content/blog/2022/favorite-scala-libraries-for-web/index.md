---
title: (WIP) 웹 서버에서 자주 쓰는 스칼라 라이브러리 소개
date: "2022-02-05T06:57:00+09:00"
description: 2022년 버전
---

자주 쓰는 유용한 스칼라 라이브러리 몇 개를 소개합니다. 스칼라 생태계가 조금씩 커지면서 비슷한 목적의 라이브러리가 여러 개 나왔는데(특히 JSON), 스칼라를 처음 접할 때 어느 것이 많이 쓰이고 활발하게 관리되고 있는지 알기 어렵기 때문에 여기에서 조금이나마 안내가 되었으면 합니다.

소개할 라이브러리들은 [Scripting](#scripting) 분류 외에는 대부분 순수 함수형입니다. 함수형 프로그래밍의 장점은 [What Functional Programming Is, What it Isn't, and Why it Matters](https://noelwelsh.com/posts/what-and-why-fp/)에서 잘 설명하고 있습니다.

## 목차

```toc
tight: true
```

## JSON

웹 프로그래밍에서 가장 자주 다루는 데이터 포맷 중 하나입니다. 정적 타입 함수형 프로그래밍에서는 JSON포맷을 직접 다루기보다 케이스 클래스로 정의한 자료형을 타입클래스를 이용해 JSON 형식으로 상호 변환하는 방법을 많이 씁니다.

```scala
import io.circe._, io.circe.generic.auto._, io.circe.parser._, io.circe.syntax._

sealed trait Foo
case class Bar(xs: Vector[String]) extends Foo
case class Qux(i: Int, d: Option[Double]) extends Foo

val foo: Foo = Qux(13, Some(14.0))

val json = foo.asJson.noSpaces
println(json) // {"Qux":{"i":13,"d":14.0}}

val decodedFoo = decode[Foo](json)
println(decodedFoo) // Right(Qux(13,Some(14.0)))
```

출처: [Circe 공식 문서](https://circe.github.io/circe/)

정적 타입이기 때문에 파이썬 딕셔너리나 자바스크립트 오브젝트를 쓰는 것에 비해 안정성이 높고 유지보수하기 쉬우며, 매번 수동으로 매핑을 작성할 필요가 없기 때문에 생산성도 높습니다. 또한 매퍼 자동 생성(타입클래스 유도derivation)이 컴파일 타임에 일어나기 때문에 런타임 리플렉션에 의존하는 자바 Jackson 처럼 컴파일은 잘 됐는데 정작 런타임에 맞는 매퍼가 없어서 문제를 겪는 경우도 드뭅니다.

일반적인 웹 애플리케이션 작성 시에는 위의 특성이 도움이 도움이 됩니다. 하지만 REPL환경에서 명세가 없는 JSON 데이터를 탐험하거나, 복잡한 비즈니스 로직 없이 단순히 하나의 JSON 데이터를 다른 JSON 형식으로 변환하기만 하는 경우에는 케이스 클래스를 선언하는 것이 부담스러울 때도 있습니다. 이럴 때는 후술할 [uJson](#uJson)을 이용해 마치 정적 타입 파이썬처럼 프로그래밍할 수도 있습니다.

### [Circe](https://circe.github.io/circe/)

Cats 기반의 함수형 JSON 라이브러리입니다. 사용방법이 간단하고 [Tapir](#tapir), [http4s](#http4s) 등 많은 라이브러리에서 통합을 지원하고 있어 JSON을 다룰 때 우선 고려하는 편입니다.

팀에서는 러닝커브를 낮추기 위해 기술 스택을 Cats + [Cats Effect](#cats-effect)에서 [ZIO](#zio) 기반으로 옮기고 있어서 향후에는 cats 의존성이 필요없는 [ZIO Json](#zio-json)을 더 많이 사용하게 될지도 모르겠습니다.

### [ZIO Json](https://github.com/zio/zio-json)

[ZIO](#zio) 기반의 JSON 라이브러리입니다. 함수형 이펙트 시스템인 ZIO가 JSON과 무슨 상관인가 싶을 수 있지만 대용량 JSON 데이터를 스트리밍 방식으로 읽고 쓸 때 성능상 유리한 점이 있습니다.

인코딩/디코딩 시 중간 AST를 거치지 않기 때문에 circe에 비해 성능이 우수하며, 임의의 큰 JSON 페이로드를 이용한 DoS 공격에도 강하다고 합니다. 디코딩 실패 시 [jq](https://stedolan.github.io/jq/) 형식의 에러 메시지를 보여주어 어디에서 문제가 생겼는지 알기 쉽게 해줍니다.

코드베이스가 복잡하지는 않지만 아직 개발 초기 단계이기 때문에 인터페이스가 바뀔 여지가 있어 프로덕션에 도입하기 전에 좀더 지켜보고 있습니다.

## SQL

### [Doobie](https://tpolecat.github.io/doobie/)

### [Quill](https://getquill.io/)

## API Development

### [Tapir](https://github.com/softwaremill/tapir)

### [http4s](https://http4s.org/)

### [ZIO Http](https://dream11.github.io/zio-http/)

## Functional Effect

### [ZIO](https://zio.dev/)

### [Cats Effect](https://typelevel.org/cats-effect/)

## Streaming

### [ZIO Stream](https://zio.dev/next/datatypes/stream/)

### [FS2](https://fs2.io/)

## Binary Data

### [Scodec](http://scodec.org/)

## GraphQL

### [Caliban](https://ghostdogpr.github.io/caliban/)

## Scripting

### [Ammonite](https://ammonite.io/)

### [Scala CLI](https://scala-cli.virtuslab.org/)

### [Requests-Scala](https://github.com/com-lihaoyi/requests-scala)

### [uJson](https://com-lihaoyi.github.io/upickle/#uJson)