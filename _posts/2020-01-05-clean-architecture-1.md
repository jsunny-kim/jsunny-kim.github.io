---
title:  "클린 아키텍처 (1)"
category: posts
tags:
  - 클린 아키텍처
comments: true
last_modified_at: 2020-01-05T21:40:00
---

# 01. 소개

> 프로그램을 동작하게 만들기는 어려운 일이 아니다.

## 1장. 설계와 아키텍처란?

- 설계(`design`)
  - **저수준의 구조** 또는 결정사항 등을 의미한다.
- 아키텍처(`architecture`)
  - 저수준의 세부사항과는 분리된 **고수준의 무언가**를 의미한다.

> 소프트웨어 설계란, 고수준(구조)에서 저수준(세부사항)으로 향하는 의사결정의 연속이다.

### 목표는?

> 소프트웨어 아키텍처의 목표는 필요한 **시스템을 만들고 유지보수하는 데 투입되는 비용을 최소화** 하는 데 있다.

### 사례 연구

> 🐰 빨리가는 유일한 방법은 **제대로** 가는 것이다.

## 2장. 두가지 가치에 대한 이야기

- 모든 소프트웨어 시스템이 이해관계자에게 제공해야 하는 두 가치
  - **행위**(`behavior`) : 개발자가 제공해야하는 가치가 행위 뿐이라고 착각해서는 절대 안된다!
    1. 기능명세서 및 요구사항 문서 구체화
    2. 요구사항에 만족하는 코드 작성
    3. 요구사항을 위반하는 경우, 디버깅 및 버그 수정
  - **구조**(`structure`)
    - 소프트웨어(`soft`+`ware`)가 가진 본연의 목적? 변경하기 쉬워야 한다.
    - 변경하는데 드는 비용?
      1. 변경사항의 범위(`scope`)에 비례해야 맞다.
      2. 변경사항의 형태(`shape`)와는 관련이 없어야 한다.
         - 아키텍처는 형태에 독립적이어야 하고, 그럴수록 더 실용적이다.

### 더 높은 가치

> **기능의 동작여부 < 아키텍처의 유연성**

### 아이젠하워 매트릭스

| ①<br />중요함<br />긴급함                        | ② **아키텍처**<br />중요함<br />긴급하지 않음 |
| ----------------------------------------- | --------------------------------------------- |
| ③ **행위**<br />중요하지 않음<br />긴급함 | ④<br />중요하지 않음<br />긴급하지 않음              |

> 긴급한 문제보다 중요한 문제가 더 높은 우선순위를 가진다.

### 아키텍처를 위해 투쟁하라

> 소프트웨어 개발자인 나도 이해관계자임을 명심하자!