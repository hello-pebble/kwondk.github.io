---
layout: default
title: "고성능 분산 캐시 시스템 설계 및 구현"
permalink: /portfolio/tech-focus-backend/
---

# [Portfolio] 고성능 분산 캐시 시스템 설계 및 구현

> 대규모 트래픽 환경에서 데이터베이스 부하를 줄이기 위한 계층형 캐시 아키텍처 도입기

---

## 📌 Project Overview
- **기간**: 2026.03 - 2026.05
- **기술 스택**: Spring Boot, Redis, Caffeine Cache, Docker
- **핵심 목표**: 반복적인 DB 조회 쿼리를 줄여 전체 시스템의 TPS(Transactions Per Second) 향상

---

## 🔍 Problem Discovery (문제 정의)
사용자 요청이 급증하는 특정 시간대에 MySQL의 CPU 사용률이 90% 이상 치솟으며, 응답 시간이 지연되는 현상 발생. 
분석 결과, 전체 요청의 70%가 변경이 거의 없는 고정된 데이터(상품 정보, 카테고리 등)를 반복 조회하고 있었음.

---

## 💡 Solution: Tiered Caching Strategy
단일 캐시가 아닌 **L1 (Local) - L2 (Global) 계층형 캐시** 구조를 설계하여 네트워크 비용까지 최적화.

1. **L1 Cache (Caffeine)**: 각 애플리케이션 노드의 메모리에 위치. 네트워크 I/O 없이 즉시 반환.
2. **L2 Cache (Redis)**: 분산 환경에서 여러 노드가 공유하는 캐시 서버. 데이터 정합성 유지.
3. **Cache Aside Pattern**: 데이터 조회 시 L1 -> L2 -> DB 순으로 탐색하고, 결과를 역순으로 캐싱.

---

## 🛠 Implementation Details (기술적 도전)

### 1. Cache Stampede 현상 방지
수만 명의 사용자가 동시에 만료된 캐시 데이터를 요청할 때 DB에 부하가 몰리는 현상을 방지하기 위해 **Probabilistic Early Recomputation** 로직 적용.

### 2. 데이터 정합성 보장
데이터 수정 시 Redis Pub/Sub을 활용하여 모든 WAS 노드의 L1 캐시를 즉시 무효화(Invalidation)하는 동기화 메커니즘 구현.

---

## 📈 Results (성과)
- **성능 측정**: TPS 약 **5배 향상** (1,200 -> 6,000)
- **DB 부하**: 평균 CPU 사용률 **40% 감소**
- **학습 포인트**: 분산 환경에서의 데이터 동기화 비용과 정합성 사이의 트레이드 오프를 깊이 이해함.

---

## 💻 Source Code
- [GitHub Repository Link](https://github.com/hello-pebble/distributed-cache-project)
