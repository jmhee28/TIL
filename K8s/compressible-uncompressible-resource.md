# Kubernetes 압축가능 자원 vs 압축불가능 자원 (Compressible vs Non-compressible)

## 1) 개념

쿠버네티스에서 말하는 **압축가능(Compressible)** / **압축불가능(Non-compressible)** 자원은,
자원이 부족할 때 시스템이 **“성능을 깎아서 버틸 수 있냐”** vs **“바로 실패(종료)로 이어지냐”**의 차이다.

---

## 2) 압축가능 자원(Compressible) = 부족하면 “느려진다”

### 대표: CPU

* 컨테이너가 CPU를 더 쓰려 해도 `limits.cpu`를 넘으면 **throttling(쓰로틀링)** 으로 제한됨
* 노드가 바쁘면 CPU 시간을 덜 받아 **지연(latency) 증가 / 처리량(throughput) 감소**
* 보통 CPU 부족은 **즉시 종료가 아니라 성능 저하**로 나타남

✅ 한 줄: **CPU는 부족하면 “속도를 제한해서” 버틴다.**

---

## 3) 압축불가능 자원(Non-compressible) = 부족하면 “터진다/죽는다”

### 대표: Memory

* 메모리는 이미 할당된 사용량을 “조금 덜 쓰게” 강제로 줄이기 어려움
* 컨테이너가 `limits.memory`를 넘어서면 → **OOMKilled**(커널이 강제 종료)
* JVM 내부에서도 Heap 상한(Xmx) 넘으면 → `OutOfMemoryError: Java heap space`

✅ 한 줄: **메모리는 부족하면 “압축”이 아니라 “실패(종료)”로 간다.**

---

## 4) 운영 관점 정리

* CPU 문제 → **쓰로틀링/느려짐**(성능 튜닝, limit 조정, HPA, 부하 분산)
* Memory 문제 → **OOMKilled/OOM**(limit 상향, 힙/오프힙/스레드/캐시 점검, 누수 확인)

---

## 5) 초간단 암기

* **CPU(압축가능)**: 넘치면 **느려짐(Throttling)**
* **Memory(압축불가능)**: 넘치면 **죽음(OOMKilled/OOM)**
