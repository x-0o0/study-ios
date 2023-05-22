# Copy

## 왜 공부하는가?
인턴 때부터 흔하게 접한거지만 따로 정리해둔적이 없어서 이번 기회에 정리하고자 한다.

## Copy 종류
### Deep copy
- 데이터 자체를 복사
- 서로 다른 메모리를 차지하기 때문에 값 변형에 대해 독립적
- e.g., struct
- 메모리 효율이 좋지 못함
### Shallow copy
- 같은 주소값을 공유하기 때문에 값 변형이 발생하면 원본도 수정됨
- e.g., class, 참조타입
### Copy-On-Write (COW)
1. Shallow copy
2. 데이터 변형 발생시에 Deep copy
- 장점: 메모리 관리 효율적
- 단점: 런타임에 데이터 변경 여부를 체크하는 작업이 필요하기 때문에 시간 손실있음

## 참조 타입의 Deep Copy
참조타입은 기본적으로 shallow copy 이지만 `NSCopying` 프로토콜을 준수하여 Deep 복사가 가능
참조 타입 안에 참조 타입의 프로퍼티가 있는 경우, 해당 프로퍼티에 한해서 Shallow copy 가 되지만, 해당 프로퍼티의 값이 새인스턴스로 할당된 경우 Deep Copy 발생
참조타입 프로퍼티가 `NSCopying` 준수하면 Deep Copy 가능
