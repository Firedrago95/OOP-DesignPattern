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
````java
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
````
문제점: 
- DataViewer 클래스는 데이터를 읽고, 파싱하고, UI를 업데이트하는 여러 책임을 가지고 있다. <br>
데이터 형식이 바뀌거나 UI가 변경되면 수정할 부분이 많아진다.
````java
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

````
개선점:
- HtmlLoader는 데이터 로딩만 담당하고, GuiUpdater는 화면 업데이트만 담당한다.
- DataViewer는 단순히 두 클래스를 사용하는 조합자 역할만 하므로, 변화의 이유가 하나로 축소된다.
### 1.2 책임이란 변화에 대한 것 
- 각각의 책임은 서로 다른 이유로 변경되고, 서로 다른 비율로 변경되는 특징이 있다.
- 책임의 단위는 변화되는 부분과 관련된다.
- 단일 책임 원칙을 잘 지키려면 메서드를 실행하는 것이 누구인지 확인해보자
- 클래스의 사용자들이 서로 다른 메서드들을 사용한다면 그들 메서드는 각각 다른 책임에 속할 가능성이 높다.

## 2. 개방 폐쇄 원칙 