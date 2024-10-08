# 5장 설계 원칙 SOLID
1. 단일 책임 원칙 (Single responsibility principle: SRP)
2. 개방-폐쇄 원칙 (Open-closed principle: OCP)
3. 리스코프 치환 원칙 (Liskov substitution principle: LSP)
4. 인터페이스 분리 원칙 (Interface segregation principle: ISP)
5. 의존 역전 원칙 (Dependency inversion principle: DIP)

## 1. 단일 책임 원칙
- 클래스는 단 하나의 책임을 가져야 한다.
- 클래스를 변경하는 이유는 단 한 개여야 한다.

### 1.1 단일 책임 원칙 위반이 불러오는 문제점
1. 책임이 많아지면 연쇄적인 변화가 발생한다.
2. 책임이 분리되지 않으면, 불필요한 부분까지 재사용 될 수 있다.
```java
public class DataViewer {
    
    public void display() {
        String data = loadHtml();
        updateGui(data);
    }
    
    // HTML 프로토콜 이용하여 데이터를 읽어들인다.
    public String loadHtml() {
        HtpClient client = new HttpClient();
        client.connect(url);
        return client.getResponse();
    }
    // 읽어온 데이터를 화면에 출력한다.
    private void updateGui(String data) {
        GuiData guiModel = parseDataToGuiData(data);
        tableUi.changeData(guiModel);
    }
    
    private GuiData parseDataToGuiData(String data) {
        ...// 파싱 처리 코드
    }
    ...// 기타 필드 등 다른 코드
}
```
문제점: 
- DataViewer 클래스는 데이터를 읽고, 파싱하고, UI를 업데이트하는 여러 책임을 가지고 있다. <br>
데이터 형식이 바뀌거나 UI가 변경되면 수정할 부분이 많아진다.
```java
// 데이터 읽기 책임을 가진 클래스
public class HtmlLoader {
    public String loadHtml(String url) {
        HttpClient client = new HttpClient();
        client.connect(url);
        return client.getResponse();
    }
}

// 화면 업데이트 책임을 가진 클래스
public class GuiUpdater {
    public void updateGui(String data) {
        GuiData guiModel = parseDataToGuiData(data);
        tableUi.changeData(guiModel);
    }
    
    private GuiData parseDataToGuiData(String data) {
        // 파싱 처리 코드
    }
}

// DataViewer는 이제 단순히 두 클래스를 조합해 사용하는 역할만 한다.
public class DataViewer {
    private HtmlLoader htmlLoader = new HtmlLoader();
    private GuiUpdater guiUpdater = new GuiUpdater();

    public void display() {
        String data = htmlLoader.loadHtml("http://example.com");
        guiUpdater.updateGui(data);
    }
}

```
개선점:
- HtmlLoader는 데이터 로딩만 담당하고, GuiUpdater는 화면 업데이트만 담당한다.
- DataViewer는 단순히 두 클래스를 사용하는 조합자 역할만 하므로, 변화의 이유가 하나로 축소된다.
### 1.2 책임이란 변화에 대한 것 
- 각각의 책임은 서로 다른 이유로 변경되고, 서로 다른 비율로 변경되는 특징이 있다.
- 책임의 단위는 변화되는 부분과 관련된다.
- 단일 책임 원칙을 잘 지키려면 메서드를 실행하는 것이 누구인지 확인해보자
- 클래스의 사용자들이 서로 다른 메서드들을 사용한다면 그들 메서드는 각각 다른 책임에 속할 가능성이 높다.

## 2. 개방 폐쇄 원칙 
- 확장에 열려 있어야 하고, 변경에는 닫혀 있어야 한다.
  - 기능을 변경하거나 확장할 수 있으면서 그 기능을 사용하는 코드는 수정하지 않는다.
1. 추상화를 이용하는 방법
2. 상속을 이용하는 방법

### 2.1 개방폐쇄 원칙이 깨질때의 주요 증상
1. 추상화와 다형성이 제대로 지켜지지 않은 코드는 개방 폐쇄 원칙을 어기게 된다.
```java
public void drawCharacter(Character character) {
    if (character instanceof Missile) { // 타입확인
        Missile missile = (Missile) character; // 타입 다운 캐스팅
        missile.drawSpecific();
    } else {
        character.draw();
    }
}
```
- 만약 Missile 외에 또 다른 Character의 하위 클래스를 추가하면, <br>
  drawCharacter() 메서드 내부에 새로운 instanceof 검사와 해당 타입에 맞는 로직을 추가해야 한다. <br>
  이는 기존 코드를 수정해야 한다는 뜻이며, 개방-폐쇄 원칙을 위반한다. <br>

2. 비슷한 if-else 블럭이 존재한다.
```java
public class Enemy extends Character {
    private int pathPattern;
    
    public Enemy(int pathPattern) {
        this.pathPattern = pathPattern;
    }
    
    public void draw() {
        if (pathPattern == 1) {
            x += 4;
        } else if (pathPattern == 2) {
            y += 10;
        } else if (pathPattern == 4) {
            x += 4;
            y += 10;
        }
        ...; // 그려 주는 코드
    }
}
```
- 문제점 : 새로운 움직임 패턴을 추가할 때마다 draw() 메서드에 if 문이 추가된다.
```java
// 움직임 전략 인터페이스 정의
public interface MovementStrategy {
    void move(Enemy enemy);
}

// 오른쪽으로 이동하는 패턴 구현
public class RightMoveStrategy implements MovementStrategy {
    @Override
    public void move(Enemy enemy) {
        enemy.setX(enemy.getX() + 4);
    }
}

// 아래로 이동하는 패턴 구현
public class DownMoveStrategy implements MovementStrategy {
    @Override
    public void move(Enemy enemy) {
        enemy.setY(enemy.getY() + 10);
    }
}

// 대각선으로 이동하는 패턴 구현
public class DiagonalMoveStrategy implements MovementStrategy {
    @Override
    public void move(Enemy enemy) {
        enemy.setX(enemy.getX() + 4);
        enemy.setY(enemy.getY() + 10);
    }
}
```
```java
public class Enemy extends Character {
    private MovementStrategy movementStrategy; // 움직임 전략

    // 생성자에서 움직임 전략을 주입받는다.
    public Enemy(MovementStrategy movementStrategy) {
        this.movementStrategy = movementStrategy;
    }

    @Override
    public void draw() {
        // 움직임 전략에 따라 이동
        movementStrategy.move(this);
        // 그리는 코드
        // ...
    }
}
```
- 개선점 : OCP를 지키기 위해, 움직임 패턴을 별도의 클래스 또는 객체로 분리하고 다형성을 사용하여 행동을 정의한다. <br>
이를 통해 새로운 움직임 패턴을 추가할 때 기존 코드를 수정할 필요 없이 확장할 수 있다. <br>

### 2.2 개방 폐쇄 원칙은 유연함에 대한 것
- 개방 폐쇄 원칙은 변경의 유연함과 관련된 원칙이다.
- 개방 폐쇄 원칙은 변화되는 부분을 추상화함으로써 사용자 입장에서 변화를 고정시킨다.
- 상속을 이용한 개방 폐쇄 원칙 구현도 가능하다.
- 변화 요구가 발생하면, 변화와 관련된 구현을 추상화해서 개방폐쇄 원칙에 맞게 수정할 수 있는지 확인하자

## 3. 리스코프 치환 원칙
- 상위 타입의 객체를 하위 타입의 객체로 치환해도 상위 타입을 사용하는 프로그램은 정상적으로 동작해야 한다.
```java
public void someMethod(SuperClass sc) {
    sc.someMethod();
}

//SuperClass의 자식 클래스를 전달해도 정상작동 해야한다.
someMethod(new Subclass());
```

### 3.1 리스코프 치환 원칙을 지키지 않을 때의 문제
#### 1. 상속관계가 아닌 것을 상속관계로 표현할때의 문제 (직사각형 - 정사각형 문제)

```java
public class Rectangle {
  private int width;
  private int height;

  public void setWidth(int width) {
    this.width = width;
  }

  public void setHeight(int height) {
    this.height = height;
  }

  public int getWidth() {
    return width;
  }

  public int getHeight() {
      return height;
  }
}
```
```java
public class Square extends Rectangle {
  @Override
  public void setWidth(int width) {
    super.setWidth(width);
    super.setHeight(width);
  }
  @Override
  public void setHeight(int height) {
    super.setWidth(height);
    super.setHeight(height);
  }
}
```
```java
// 정사각형이 파라미터에 전달 될 경우 문제가 발생한다.
public void increaseHeight(Rectangle rec) {
    if (rec.getHeight() <= rec.getWidth()) {
        rec.setHeight(rec.getWidth() + 10);
    }    
}
```
- 문제점 : 개념상 상속관계처럼 보이는 직사각형 - 정사각형을 상속으로 구현 한 경우 사용시 문제가 생길 수 있다.
- increaseHeight()같은 기능이 필요할대는 Rectangle, Square를 별도의 타입으로 구현해야 한다.

#### 2. 상위 타입에서 지정한 리턴 값의 범위에 해당되지 않는 값을 리턴하는 경우
```java
public class CopyUtil {
    public static void copy(InputStream is, OutputStream out) {
      byte[] data = new byte[512];
      int len = -1;
      
      // InputStream.read() 메서드는 스트림의 끝에 도달하면 -1 리턴
      while ((len = is.read(data)) != -1) {
        out.write(data, 0, len);
      }
    }
}
```
```java
public class SatanInputStream implements InputStream {
    public int read (byte[] data) {
      ...
      return 0; // 데이터가 없을때 0을 리턴
    }
}
```
- 문제점: InputStram을 상속받은 SatanInputStream 이 데이터가 없을때 0을 리턴한다. <br>
  SatanInputStream이 CopyUtil 의 copy() 파라미터로 전달되면 무한루프가 실행된다. <br>
- 하위 타입의 객체는 상위타입의 리턴값을 올바르게 전달하도록 해야한다.

### 3.2 리스코프 치환 원칙은 꼐약과 확장에 대한 것
- <span style="font-weight: bold;">명시된 명세에서 벗어난 값, 익셉션, 기능을 리턴하지 않도록 주의하자!</span>
  - 직사각형 문제에서 Rectangle 클래스의 setHeight() 메서드는 두가지 계약을 제공한다.
    - 1.높이값을 파라미터로 전달받은 값으로 변경한다.
    - 2.폭 값은 변경되지 않는다.
  - 반면 Square 는 setHeight() 메서드를 통해 높이와 폭 모두를 변경한다.
- <span style="font-weight: bold;">리스코프 치환 원칙은 확장에 관한 것이기도 하다</span>
```java
public class Item {
  // 변화되는 기능을 상위 타입에 추가
  public boolean isDiscountAvailable() {
      return true;
  }
}

public class SpecialItem extends Item {
  // 특별 아이템은 할인되지 않는다면
  @Override
  public boolean isDiscountAvailable() {
      return false;
  }
}
```
- 리스코프 치환 원칙이 지켜지지 않으면, 개방 폐쇄 원칙도 지켜지지 않는다.
- 위 코드 처럼 변화하는 코드를 상위 타입에 추가하고 하위 타입이 오버라이딩하면 <br>
  사용하는 코드에서 instanceof 같은 불필요한 코드를 사용할 필요가 없다.