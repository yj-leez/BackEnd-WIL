# Ch6

### 자료 추상화

- 자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 것이 좋다.

```java
// 구체적인 Point 클래스
public class Point {
    // 필드값이 public으로 오픈
    public double x;
    public double y;
}

// 추상적인 Point 클래스

public interface Point {
    // 필드값이 외부로 노출되어 있지 않고 오직 getter / setter로만 접근 가능
    double getX();
    double getY();
    void setCartesian(double x, double y);

    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```

- 위의 예시에서는 getter/ setter를 통해 필드값을 감추었다. 하지만 모든 필드에 대해 getter/setter 처리를 한다고 추상화를 한 것이 아니다.
- 아래 예시를 보면 첫번째는 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려준다. 반면, 두 번째는 갤런 단위로 디테일하게 읽어서 반환하고 있음을 메소드 이름을 통해 알 수 있다.
- 첫번째는 정보가 어디서 오는지 전혀 드러나지 않는다. 하지만 두번째는 두 함수 각각 변수값을 읽어 그대로 반환할 뿐이다. 사실 사용자가 알고 싶은 건그래서 몇프로나 남았는데? 임에도 불구하고 말이다. 이렇듯 자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 것이 좋다. 인터페이스나 getter/setter 만으로 추상화가 이뤄지지 않는다.

```java
public interface Vehicle {
    double getPercentFuelRemaining();
}
```

```java
public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline();
}
```

### 자료/ 객체 비대칭

- 자료구조를 사용하는 절차적인 코드는 기존 자료구조를 변경하지 않으면서 새 함수를 추가하기 쉽다.
    
    ```java
    import java.awt.Point;
    
    public class Square {
        public Point topLeft;
        public double side;
    }
    
    public class Rectangle {
        public Point topLeft;
        public double height;
        public double width;
    }
    
    public class Circle {
        public Point center;
        public double radius;
    }
    
    public class Geometry {
        public final double PI = 3.141592;
    
        public double area(Object shape) throws NoSuchShapeException {
            if (shape instanceof Square) {
                Square s = (Square) shape;
                return s.side * s.side;
            } else if (shape instanceof Rectangle) {
                Rectangle r = (Rectangle) shape;
                return r.height * r.width;
            } else if (shape instanceof Circle) {
                Circle c = (Circle) shape;
                return PI * c.radius * c.radius;
            }
            throw new NoSuchShapeException();
        }
    }
    
    ```
    
- 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다. 하지만 메소드를 추가할 때는 절차 지향보다 훨씬 고쳐야할 게 많다. 해당 행동과 관련된 모든 객체에게 행동을 부여해야 하기 때문이다.
    
    ```java
    import java.awt.Shape;
    
    public class Square implements Shape {
        private Point topLeft;
        private double side;
    
        public double area() {
            return side * side;
        }
    }
    
    public class Rectangle implements Shape {
        private Point topLeft;
        private double height;
        private double width;
    
        public double area() {
            return height * width;
        }
    }
    
    public class Circle {
        private Point center;
        private double radius;
    
        public double area() {
            return PI * radius * radius;
        }
    }
    ```
    
- 예컨대 복잡한 시스템을 짜다 보면 새로운 자료 타입이 필요한 경우가 생긴다. 이때는 클래스 및 객체지향 기법을 사용하는 것이 적절하다. 반면, 새로운 함수가 필요한 경우도 생긴다. 이때는 절차적인 코드와 자료구조가 더 적합하다.즉, 모든 것이 객체라는 생각은 미신임을 알아야 한다. 때로는 단순한 자료구조와 절차적인 코드가 가장 적합한 상황도 있다.

### 디미터 법칙

- 디미터 법칙은,클래스 C의 메서드 f는 아래와 같은 객체의 메소드만 호출헤애 한다고 주장한다.
- `final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();`
    
    우리는 모든 getter를 체인으로 연결함으로써 내부 속사정을 다 들여볼 수 있게 된다. 위와 같은 코드를 기차 충돌이라 한다. 이런 건 조잡하기에 피하는 게 좋다.
    
    ```java
    s = ctxt.getOptions();
    File scratchDir = opts.getScratchDir()
    final String outputDir = scratchDir.getAbsolutePath();
    ```
    
    이렇게 분리한다고 내부 속사정 엿보기가 안 드러나는 것은 아니지만, 만약 ctxt, Options, ScratchDir이 자료구조였다면 드러나는 것이 당연하다.
    

### 자료 전달 객체

- 자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다. 이런 자료 구조체를 자료 전달 객체(Data Transfer Object, DTO)라고 한다. DTO는 굉장히 유용한 구조체다. 특히 DB와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용하다.흔히 DTO는 DB에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체이다.