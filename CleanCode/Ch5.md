# Ch5

### 적절한 행 길이를 유지하라

- 개념은 빈 행으로 분리하라: 빈 행은 새로운 개념을 시작한다는 시각적 단서이다.
- 수직 거리
    - 서로 밀접한 개념은 세로로 가까이 둬야 한다. 타당한 근거가 없다면 서로 밀접한 개념은 한 파일에 속해야 마땅하다. 이게 바로 protected 변수를 피해야 하는 이유 중 하나다.
    - 변수는 사용하는 위치에 최대한 가까이 선언한다.
    - 한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다.
    - 개념적 친화도가 높은 코드는 세로로 가까이 배치한다.
- 세로 순서
    - 호출하는 함수보다 호출되는 함수를 나중에 배치한다. 그러면 소스 코드 모듈이 고차원에서 저차원으로 자연스럽게 내려간다. 신문 기사와 마찬가지로 가장 중요한 개념을 가장 먼저 표현한다. 가장 중요한 개념을 표현할 때는 세세한 사항을 최대한 배제한다.
- 들여쓰기로 범위를 제대로 표현하자
    
    ```java
    public Class CommentWidget extends TextWidget{
    	public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n|\n\r)?";
    	
    	public CommentWidget(ParentWidget parent, String text){super(parent,text);}
    	public String render() throws Exception {return "";}
    }
    ```
    
    ↓ 훨씬 더 가독성이 좋아진다.
    
    ```java
    public Class CommentWidget extends TextWidget{
    	public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n|\n\r)?";
    	
    	public CommentWidget(ParentWidget parent, String text){
    		super(parent,text);
    	}
    
    	public String render() throws Exception {
    		return "";
    	}
    }
    ```
    
- 팀 규칙
    - 어디에 괄호를 넣을지, 들여쓰기는 몇 자로 할지, 클래스와 변수와 메서드 이름은 어떻게 지을지 결정하는 걸 권장한다. 좋은 소프트웨어 시스템은 읽기 쉬운 문서로 이뤄지며 일관적이고 매끄러운 스타일로 되어있다.