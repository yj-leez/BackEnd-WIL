# 3장. 모든 객체의 공통 메서드

## 아이템 10. equals는 일반 규약을 지켜 재정의하라

- Object.equals() 호출 시 주소 값을 비교하여 자기 자신의 인스턴스와만 같은지 판단
- equals를 재정의함으로 불필요한 문제가 생길 수 있으므로 재정의 하지말자
    
    재정의 할 필요가 없는 경우
    
    1. 각 인스턴스가 본질적으로 고유한 경우
    2. 설계 상 논리적 동치성을 검사할 일이 없는 경우
    3. 상위 클래스 (주로 abstract) 의 equals()가 하위 클래스에도 딱 들어맞는 경우
    
    ```java
    @Override public boolean equals(Object o) {
    		throw new AssertionError(); // 호출 금지
    }
    ```
    
- equals를 재정의 하는 경우
    
    : 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때
    
    → 주로 String, Integer과 같은 값 클래스가 여기에 해당
    
- equals가 만족해야하는 규칙들
    1. 반사성: 객체는 자기 자신과 같아야함 → 앵간하면 만족
    2. 대칭성: 두 객체는 서로에 대한 동치 여부에 대해 똑같이 답해야함
        
        대소문자를 구별하지 않는 문자열 CaseInsesitiveString 객체의 equals를 String과 비교하도록 구현 시 한 방향으로만 작동 → 다음과 같이 해결
        
        ```java
        // 수정한 equals 메서드 (56쪽)
        @Override public boolean equals(Object o) {
            return o instanceof CaseInsensitiveString &&
                   ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
        }
        ```
        
    3. 추이성: 첫 번째 객체와 두 번째 객체가 같고 두 번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 함
        
        → 상속 관계에서 하위 클래스가 필드를 추가한다면 정상적으로 equals 규약을 만족시킬 방법이 없음
        
        → 상속 대신 컴포지션을 사용하라
        
        ```java
        public class ColorPoint {
            private final Point point; // 상속하는 대신 private 필드로
            private final Color color;
        
            public ColorPoint(int x, int y, Color color) {
                point = new Point(x, y);
                this.color = Objects.requireNonNull(color);
            }
        
            /**
             * Colorpoint와 같은 위치의 일반 Point를 반환하는 뷰 메서드 추가
             */
            public Point asPoint() {
                return point;
            }
        
            @Override public boolean equals(Object o) {
                if (!(o instanceof ColorPoint))
                    return false;
                ColorPoint cp = (ColorPoint) o;
                return cp.point.equals(point) && cp.color.equals(color);
            }
        }
        ```
        
    4. 일관성: 두 객체가 같다면 수정되지 않는 한 앞으로도 영원히 같아야 함
        - 신뢰할 수 없는 자원이 끼어들게 해서는 안 됨: url.equals() 수행 시 ip로 비교하는데 결과가 항상 같음을 보장할 수 없음 → 잘못 설계된 예시
    5. null-아님: 모든 객체가 null과 같지 않아야 함 → 명시적 null 검사보다는 매개변수가 올바른 타입인지 검사 하는 묵시적 null 검사 활용하자
        
        ```java
        @Override public boolean equals(Object o) {
            /*	명시적 Null 검사 */
        //		if (o == null) 
        //			return false;
        
        		/* 묵시적 Null 검사 */
        		if(!(o instanceof MyType))
        			return false;
        		Mytype mt = (MyType) o;
        		...
        }
        ```
        

- 양질의 equals 메서드 구현 방법
    1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인함
    2. `instanceof 연산자`로 입력이 올바른 타입인지 확인함
    3. 입력을 올바른 타입으로 형변환
    4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사
    
    *floate와 double을 제외한 기본 타입 필드는 `== 연산자`로 비교하고, 참조 타입 필드는 각각의 `equals 메서드`로, float와 double 필드는 각각 정적 메서드인 `Float.compare(float, float)`와 `Double.compare(double, double)`로 비교
    
    1. 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교
    2. equals를 재정의할 땐 hashCode도 반드시 재정의하라
    
    → 구글이 제공하는 AutoValue 프레임워크 사용 시 메서드들을 알아서 작성해줌