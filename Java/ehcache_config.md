# Ehcache

## Ehcache 주요 특징

* 경량의 빠른 캐시 엔진
* 확장(scalable) - 메모리 & 디스크 저장 지원, 멀티 CPU의 동시 접근에 튜닝
* 분산 지원 - 동기/비동기 복사, 피어(peer) 자동 발견
* 높은 품질 = Hibernate. Confluence, Spring 등에서 사용되고 있으며, Caia 컴포넌트에서도 Ehcache를 사용하여 캐시를 구현했다.

## 기본 사용법

Ehcache를 사용하기 위해서는 다음과 같은 작업이 필요하다.

* Ehcache 설치
* 캐시 설정 파일 작성

* CacheManager 생성
* CacheManager로부터 구한 Cache를 이용한 CRUD 작업 수행
* CacheManager 종료

## 캐시 설정

| name                            |                                                              |
| ------------------------------- | ------------------------------------------------------------ |
| maxEntriesLocalHeap             | 메모리에 저장될 수 있는 객체의 최대 갯수                     |
| maxEntriesLocalDisk             | 디스크에 생성될 최대 객체 갯수                               |
| eternal                         | 이 값이 true이면 timeout 관련 설정(timeToTdleSeconds, timeToLiveSeconds)은 무시되고, Element가 캐시에서 삭제되지 않는다. |
| overflowToDisk                  | 메모리에 저장된 객체 개수가 maxElememtsInMemory에서 지정한 값에 다다를 경우 디스크에 오버플로우 되는 객체는 저장할 지의 여부를 지정한다. |
| timeToIdleSeconds               | Element가 지정한 시간 동안 사용(조회)되지 않으면 캐시에서 제거된다. 이 값이 0인 경우 조회 관련 만료 시간을 지정하지 않는다. 기본값은 0이다. |
| timeToLiveSeconds               | Element가 존재하는 시간. 이 시간이 지나면 캐시에서 제거된다. 이 시간이 0이면 만료 시간을 지정하지 않는다. 기본값은 0이다. |
| diskPersistent                  | VM이 재 가동할 때 디스크 저장소에 캐싱된 객체를 저장할지의 여부를 지정한다. 기본값은 false이다. |
| diskExpiryThreadIntervalSeconds | Disk Expiry 스레드의 수행 시간 간격을 초 단위로 지정한다. 기본값은 120이다. |
| memoryStoreEvictionPolicy       | 객체의 개수가 maxElementsInMemory에 도달했을 떄, 메모리에서 객체를 어떻게 제거할 지에 대한 정책을 지정한다. 기본값은 LRU이다. FIFO와 LFU도 지정할 수 있다. |

## References

* http://www.ehcache.org/documentation/2.8/configuration/configuration.html

* http://javacan.tistory.com/entry/133

* http://hyeooona825.tistory.com/86