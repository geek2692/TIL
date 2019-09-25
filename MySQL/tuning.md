## 개발자를 위한 MYSQL  성능 최적화

------

## Index

- 인덱스 지정할 컬럼들을 기준으로 메모리 영역에 목차 생성
- insert, update, delete 성능저하 -> delete, update를 하기위해 조회하는것은 인덱스를 사용하고 있다면 빠름
- select(조회) 성능향상

##### MySQL에서는 기본적으로 B-Tree 형태의 인덱스 구조 

- 리프노드는 더블 링크드 리스트로 연결되어 탐색에 용이
- 인덱스 탐색은 기본적으로 Root -> Branch -> Leaf -> 디스크저장소 순으로 진행
- 메모리에서 읽는것보다 디스크에서 읽는게 성능이 안좋음
  - 결국 인덱스 성능을 향상시킨다는건 디스크 접근을 줄이고 Root에서 Leaf까지 탐색을 얼마나 덜하느냐에 달림 
- 인덱스 개수는 3개정도가 적당
  - 너무 많은 인덱스 지정시 새로운 row를 추가시마다 인덱스를 추가해야하고 수정/삭제 마다 인덱스 수정이 필요함 (성능저하)
  - 옵티마이저가 인덱스를 잘못 선택할 확율이 있음

#### InnoDB

- B-Tree구조를 기본으로 클러스터링 인덱스 구조를 가짐

  클러스터링 인덱스: 프라이머리 키값이 비슷한 레코드끼리 묶어서 저장
  (참조: https://12bme.tistory.com/149)

#### 인덱스 컬럼기준

- 단일인덱스로 선정하는경우 카디날리티(중복된수치, 예를들면 주민등록번호, 사업자번호)가 높은 것을 인덱스컬럼 사용
- 만약 성별을 인덱스로 선정한 경우 50% 밖에 걸러내지 못함, 유니크한 값에 가까울수록 적합함
- **여러 컬럼으로 인덱스를 선정시 기준**
  - 카디널리티가 높은것에서 낮은것순으로 사용하는것이 성능이 더 뛰어남
  - 첫번째 인덱스는 조회조건에 꼭 포함되야됨, 그래야 인덱스를 사용하여 조회함
  - 인덱스 조회시 주의 사항 (참조: [https://jojoldu.tistory.com/243?category=761883#4-%EC%9D%B8%EB%8D%B1%EC%8A%A4-%EC%A1%B0%ED%9A%8C%EC%8B%9C-%EC%A3%BC%EC%9D%98-%EC%82%AC%ED%95%AD](https://jojoldu.tistory.com/243?category=761883#4-인덱스-조회시-주의-사항))

------

##### Inner Join / Outer Join

##### Inner Join

- 서로 매칭되는것만 엮어 조회

##### outer join

- 1:N outer join: N이 count 1이상일때

##### semi join

- 1:N 비효율 해결하기위해
- 사용상황: 1 select 하고있는데 groupby, distinct가 나오면서 집계함수가 없다
- loose scan
- first match

#### join 방식

**nested loop**

#### 복합인덱스

- equal 조건은 좌측으로 구성

#### 형변환

- 우선순위: 문자 < 숫자

------

desc index : mysql 5.7 이하 버젼 지원안함

plan:

- row: rows가 많다 -> 튜닝포인트
- type:
  - index: index full scan
  - all: all scan
- extra:
  - using filesort
  - using join buffer

위의 경우 튜닝이 필요