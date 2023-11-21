# JVM

  자바 가상 머신은 자바 플랫폼의 초석이다. JAVA는 자바 가상 머신 위에서 실행되며 JAVA의 동작 방식을 이해하기 위해서는 자바 가상 머신에 대한 이해가 선행.

  이번글은 JVM 에 대해서 알아볼려고 한다.


  ## JDK

  - JAVA 개발 환경으로 자바 어플리케이션을 개발하기 위해 필요한 도구를 제공한다.
  - 자바 언어를 바이트 코드로 컴파일 해주는 자바 컴파일러(Javac), 자바 클래스 파일을 해석해주는 역 어셈블리어(Javap) 등이 있다.

  ## JRE
  - JRE는 자바 실행 환경으로 JVM, 자바 클래스 라이브러리, 기타 자바 어플리케이션 실행에 필요한 파일들을 포함한다.


  ## JVM

  JVM은 자바 가상머신으로 간단하게 말하면 컴파일된 코드(바이트코드)를 실행시켜주는 가상의 컴퓨터 라고 생각하면 된다. 참고로 JVM은 H/W와 OS 위에서 실행되기 때문에 JVM 자체는 플랫폼에 종속적이다.


  ## JAVA CODE 실행 과정

  ![image](https://github.com/russell-seo/TIL/assets/79154652/f926ed12-cacb-42ea-98c3-ba94c243ac00)

  1. 작성한 자바 코드를 JAVA Compiler를 통해서 자바 바이트 코드로 컴파일 한다.
  2. 컴파일된 바이트코드를 JVM 클래스로더 에게 전달한다.
  3. 클래스로더는 동적로딩(Dynamic Loading)을 통해 필요한 클래스들을 로딩 및 링크하여 런타임 데이터 영역, 즉 JVM 메모리에 올린다.
  4. 실행 엔진은 JVM 메모리에 올라온 바이트 코드들을 명령어 단위로 하나씩 가져와서 실행한다.


 ### Class Loader

 ![image](https://github.com/russell-seo/TIL/assets/79154652/11d5eabc-f304-489a-90b6-86f0e18b917c)



- Bootstrap Class Loader
  - 최상위 클래스 로더로 유일하게 JAVA가 아니라 네이티브 코드로 구현되어 있다.
  - JVM이 실행될 때 같이 메모리에 올라간다.
  - Object 클래스를 비롯하여 JAVA API들을 로드한다.
 
- Extension Class Loader
  - JAVA API를 제외한 확장 클래스 들을 로드한다.
 
- System Class Loader
  - 부트스트랩과 익스텐션 클래스 로더가 JVM 자체의 구성요소들을 로드한다면, 시스템 클래스 로더는 어플리케이션의 클래스들을 로드한다.
  - 사용자가 지정한 $CLASSPATH 내의 클래스들을 로드한다.
 
- User-Defined Class Loader
  - 어플리케이션 사용자가 직접 코드상에서 생성하여 사용하는 클래스
 

### Runtime Data Area

JVM이 OS위에서 실행되면서 할당받는 메모리 영역이 바로 런타임 데이터 영역이다.

![image](https://github.com/russell-seo/TIL/assets/79154652/bc0b14a7-147f-4c41-bbc2-3bd5688fd2cb)


PC 레지스터, JVM 스택, Native Method Stack은 `쓰레드(Thread)`마다 하나씩 생성되고, `Heap`,`Method Area`는 모든 쓰레드가 공유해서 사용할 수 있다.

- PC Register
  - PC 레지스터는 현재 수행중인 명령의 주소를 가지며 쓰레드가 시작될 때 생성되며 각 쓰레드 마다 하나씩 존재한다.
 
- JVM Stack
  - Stack Frame 이라는 구조를 저장하는 스택이다. 예외 발생 시 printStackTrace() 메서드로 보여주는 Stack Trace의 각 라인 하나가 스택 프레임을 표현한다.
  - JVM Stack 역시 PC 레지스터와 마찬가지로 쓰레드가 시작될 때 생성되며 각 쓰레드 마다 하나씩 생성
 
- Native Method Stack
  - JAVA 외의 언어로 작성된 네이티브 코드를 위한 스택이다.
  - JNI(Java Native Interface)를 통해 호출하는 C/C++ 등의 코드를 수행하기 위한 스택.
 
- `Heap`
  - 인스턴스 또는 객체를 저장하는 공간으로 GC대상이다.
  - JVM 성능 등의 이슈에서 가장 많이 언급되는 공간이다.
 
- Method Area
  - 모든 쓰레드가 공유하는 영역으로 JVM이 시작될 때 생성된다.
  - JVM이 읽어들인 각각의 클래스와 인터페이스에 대한 런타임 상수 풀, 필드와 메서드에 대한 정보, Static 변수, 메서드의 바이트 코드등을 보관한다.
 
- Runtime Constant Pool
  - JVM 동작에서 가장 핵심적인 역할을 수행하는 곳.
  - 각 클래스와 인터페이스의 상수 뿐 만 아니라, 메서드와 필드에 대한 모든 레퍼런스까지 담고 있는 테이블
  - 어떤 메서드나 필드를 참조할 때 JVM은 런타임 상수 풀을 통해 해당 메서드나 필드의 실제 메모리상 주소를 찾아서 참조한다.
 

### 실행엔진(Execution Engine)

![image](https://github.com/russell-seo/TIL/assets/79154652/71d97eab-63c6-44c0-b743-9c2af6356842)

실행 엔진은 클래스 로더를 통해 런타임 데이터 영역에 배치된 바이트 코드를 명령어 단위로 읽어서 실행(CPU가 기게 명렁어를 하나씩 실행 하듯)

바이트 코드의 각 명령어는 1바이트 크기의 Operation Code와 추가 피연산자로 이루어져 있다.

- 인터프리터 : 바이트 코드 명령어를 하나씩 읽어서 해석하고 실행한다. 하나하나의 해석은 빠르지만 전체 실행 속도는 느리다는 단점.
  - JVM 안에서 바이트코드는 기본적으로 인터프리터 방식
 
- JIT Compiler : 인터프리터 단점을 보완하기 위해 도입된 방식
  - 바이트 코드 전체를 컴파일 하여 네이티브 코드로 변경하고 이후에는 해당 메소드를 더이상 인터프리팅 하지 않고 네이티브 코드로 직접 실행
  - 하나씩 인터프리팅 하여 실행하는 것이 아니라 바이트 코드 전체가 컴파일된 네이티브 코드를 실행하는 것
 