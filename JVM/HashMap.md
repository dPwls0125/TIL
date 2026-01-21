# HashMap 코드 분석 

**해시 테이블 정의** 
- 검색하고자 하는 키 값(Key Value)를 입력받아
- 해시 함수를 통해 해시 코드(Hash Code)를 반환받고, 이 해시 코드를 
- 배열의 인덱스로 환산하여 데이터에 접근하는 방식이다. 


**충돌(Collision)의 발생 원인과 해결의 중요성**
1. 검색 시간의 변화 
a. 해시 테이블의 최대 장점은 검색 시간이 O(1)이라는 것이다. 
b. 그러나 충돌(Collision)이 많은 경우, 최대 검색 시간은 O(n)까지 걸릴 수 있다. 
ex) 하나의 인덱스에 Node가 몰리면, 원하는 값을 찾기 위해 n만큼 뒤져야할 수 있음. 

2. 좋은 해시 알고리즘의 기준
좋은 알고리즘은 입력받은 키를 가지고 얼마나 골고루 잘 분배하는지에 의해 결정된다. 

3. 충돌의 근본적 원인  
a. 키 값의 가짓수와 해시 코드의 한계 : 키 값은 문자열로 가짓수가 무한한 반면, 해시 코드는 정수 양만큼만 제공될 수 있어 알고리즘이 아무리 좋아도 어떤 키들은 중복되는 해시 코드를 가질 수밖에 없다. 
b. 배열 방의 한정 : 서로 다른 키가 다른 해시 코드를 만들어냈더라도, 배열 방이 한정되어 있어 같은 방에 할당받는 경우도 발생한다.  

4. 콜리전(Collision)의 정의 : 서로 다른 키를 넣었을 때 동일한 해시 코드가 만들어져 한 방에 들어와 충돌하게 되는 경우를 모두 "콜리전"이라고 정의한다.

5. 해시 테이블 구현의 핵심 이슈: 충돌을 최소화하기 위해 좋은 해시 알고리즘을 만드는 것이 해시 테이블 구현에서 매우 중요한 이슈이다.







## 1. putVal

```
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;

            return o instanceof Map.Entry<?, ?> e
                    && Objects.equals(key, e.getKey())
                    && Objects.equals(value, e.getValue());
        }
    }
```