# 아이템14- Comparable을 구현할지 고려하라

Comparable 인터페이스를 구현하면 compareTo를 재정의하여 배열을 쉽게 재정렬할 수 있다. 

따라서 알파벳, 숫자, 연대와 같이 순서가 명확한 클래스를 작성한다면 comparable을 구현하자

 

### compareTo 메서드의 일반 규약

순서를 위해 이 객체를 주어진 객체와 비교합니다. 

이 객체가 주어진 객체보다 작으면 음의 정수를 반환해야하고, 

같은 경우에는 0, 큰경우에는 양의 정수를 반환합니다. 

만약 해당 객체와 비교할 수 없는 타입이 주어진다면, `ClassCastException`을 던져야 한다. 

1. 대칭성 

`Comparable을 구현한 클래스는 모든 x, y에 대하여 sgn(x.compareTo(y)) == -sgn(y.compareTo(x))여야 한다. 따라서 x.compareTo(y)는 y.compareTo(x)가 예외를 던질때에 한해 예외를 던져야 한다.`

1. 추이성 

첫번째가 두번째보다 크고 두번째가 세번째보다 크면 첫번째는 세번째보다 커야한다. 

`Comparable을 구현한 클래스는 추이성을 보장해야 한다. 즉, (x.compareTo(y)>0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0`

1. 반사성 

크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같다. 

`Comparable을 구현한 클래스는 모든 z에 대해 x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))이다.`

1. equals

`compareTo` 메서드로 수행한 동치성테스트 결과가 `equals`와 같아야한다.

`(x.compareTo(y) == 0) == (x.equals(y))여야 한다. Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 그 사실을 명시해야 한다. (주의: 이 클래스의 순서는 equals 메서드와 일관되지 않다.)`
