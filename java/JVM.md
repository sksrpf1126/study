# **_JVM(Java Virtual Machine)_**

- ## **_JVM이란?_**

  - JVM은 **_자바 가상 머신_** 이라 하며, 자바를 실행하기 위한 가상의 기계를 의미한다.  
    JAVA 언어가 OS에 영향을 받지 않고 독립적으로 실행이 되는것이 하나의 장점이라고 하는데, 그 이유가 JVM이 OS 위에서 독립적으로 동작이 되어 자바를 실행하기 때문이다.

  - JDK(Java Development Kit)를 설치할 때 JVM 또한 같이 설치가 된다.(JDK안에 존재)

  </br>

  ***

- ## **_JVM이 JAVA를 실행하는 과정_**

  - JVM은 우선 OS로부터 메모리를 할당받으며, 해당 메모리를 영역에 맞게 나누어서 관리한다.

  - 자바의 문법으로 작성하여 만들어진 .java파일은 사람이 이해하는 원시코드일 뿐 컴퓨터는 이해할 수 없다.  
    그렇기에 프로그램이 실행이 될 때에 자바컴파일러가 해당 .java파일을 .class파일이라는 바이트 코드로 컴파일을 해준다.

  - 이후 JVM 내에 존재하는 Class Loader가 .class파일을 JVM내의 Runtime Data Area에 적재한다.

  - 적재된 .class파일은 바이트코드로, 아직은 컴퓨터가 이해할 수 있는 수준이 아니므로 JVM 내의 실행엔진이 적재된 .class파일을 컴퓨터가 이해할 수 있는 형태로 변경하고 이를 실행한다.

    ```
    변경하는 방식은 인터프리터 방식과 JIT(Just-In-Time) 컴파일러에 의한 컴파일 방식 두 방식을 사용하는데, 기존에는 인터프리터 방식으로 하나의 명령어마다 해석하고 실행하며 
    JVM내에서 해당 명령어들(메서드)이 얼마나 자주 사용하는지 판단을 하고, 사용 여부가 많을 경우 인터프리터 방식을 중지하고, 나머지 코드들을 JIT 컴파일러에 의해 컴파일을 하여 캐싱해 둔다.
    ```

  </br>

  ***

- ## **_JVM 구조_**

  </br>

  <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/187596670-d8455df2-e6a3-40fb-9717-9e42cf75e460.png" height = 350px></p>

  - ### **_Class Loader_**

    클래스 로더는, 자바 컴파일러에 의해서 java 파일(원시 코드) -> class 파일(바이트 코드)로 변환이 된 class 파일을 JVM 내로 가져와 적재시키는 역할을 한다.

  - ### **_Execution Engine(실행 엔진)_**

    클래스 로더에 의해 적재된 class 파일(바이트 코드)을 기계가 이해하고 실행할 수 있는 형태로 변환시켜주는 역할을 하며, 인터프리터와 JIT 컴파일러 두 가지 방식으로 나뉘어진다.

  - ### **_Interpreter_**

    바이트 코드에서 하나의 명령어단위로 해석하고 실행시킨다.

  - ### **_JIT(Just-In-Time) Compiler_**

    인터프리터 방식으로 해석이 되고 실행이 되다가, 해당 명령어들이 전체적으로 많이 사용이 되는 경우에는 JIT 컴파일러에 의해서 런타임 중에 컴파일을 하고 캐싱해둔다.  
    JVM의 캐시 메모리는 크게 할당하지 않기 때문에 자주 쓰는 부분들만 컴파일을 해둔다.

    ```
    인터프리터와 컴파일러

    인터프리터의 경우 실행(런타임)하고 나서 하나의 명령어마다 해석 후 실행을 함

    컴파일러의 경우 모든 코드를 기계어로 해석(컴파일)한 후에 컴파일에 의해 만들어진 실행 파일들을 통해서 실행을 함

    그렇기에 실행 시간까지는 인터프리터가 빠르지만 이후에 전체적인 프로그램의 동작시간은 컴파일러가 더 빠르다.
    하지만 컴파일러는 OS가 바뀔때마다 컴파일을 다시해야하며, 코드를 수정한 뒤에도 다시 컴파일 과정을 거치고 실행이 되는 단점이 있다.

    하지만 JAVA는 위 두 장점을 사용할 수 있다.
    런타임 이전에 자바컴파일러로 자바파일을 클래스파일 즉 바이트 코드로 변환을 하는데, 해당 바이트 코드는 OS에 종속적이지 않기 때문에 OS가 바뀌어도 자바컴파일러에 의한 컴파일 과정을 거치지 않아도 된다.
    또한, 인터프리터 방식에 의해 바로 실행이 되어 실행시간이 빠르며, 이후 자주 사용하는 명령어들은 JIT 컴파일러에 의해 변환되어 캐싱해두기 때문에 컴파일의 장점 또한 가질 수 있는 것이다.
    ```

  - ### **_Garbage Collector(GC)_**

    사용되지 않는 인스턴스(객체)를 찾아서 메모리에서 제거한다. (자세한 내용은 Heap 영역에서)

  - ### **_Runtime Data Area_**

      </br>
        <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/187596678-ddd42564-6162-4a57-82b0-2e7892acbbb6.png" height = 350px></p>

    - 프로그램을 수행하기 위해서 OS로부터 할당을받은 메모리 공간

    </br>

  - ### **_PC Register_**

    쓰레드마다 하나씩 존재하며, 쓰레드 내에서 어떤 부분을 어떠한 명령으로 실행해야 할지에 대한 정보가 담겨있는 부분이다.

  - ### **_JVM Stack_**

    쓰레드마다 하나씩 존재하며, 프로그램 실행과정에서 임시로 할당되었다가 소멸이 되는 데이터들이 담기는 공간으로, 일반적으로 메서드의 매개변수, 지역변수와 같은 데이터가 저장이 된다.

  - ### **_Native Method Stack_**

    쓰레드마다 하나씩 존재하며, 자바의 바이트 코드가 아닌 다른 실행할 수 있는 코드 즉, 다른 언어로 작성되어진 코드를 실행하기 위한 공간이다.

  - ### **_Method Area(= Class Area, = Static Area)_**

    모든 쓰레드가 공유하는 영역으로, 클래스에 대한 정보(클래스안에 선언된 메서드나 멤버필드 등 객체를 만들기 위해서 필요한 정보)와 Static으로 선언되어 모든 객체가 공유하여 사용할 수 있는 데이터들 그리고 constant pool(상수 풀)이라는 메서드 영역 내부에 또다른 공간에서는 리터럴, 상수 등을 저장할 수 있는 공간 또한 존재한다.

  - ### **_Heap Area_**

      </br>

    <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/187596689-f1a863a1-ee11-46fc-9e58-4583e85fc7d8.png" height = 150px></p>

    위 사진은 JAVA 7의 Heap영역 구조를 나타내며, 아래 이미지는 Java 7과 Java8과의 메모리 구조 변화에 대해 나타낸다.

  <p align = "center"><img src="https://user-images.githubusercontent.com/62879192/187611022-1dbcee42-1425-474e-b5d1-d3595adece7f.jpg" height = 250px></p>
       출처 : https://www.programmersought.com/article/4905216600/

    </br>
    자바 7에서 자바 8로 넘어가면서 Heap영역에 존재했던 PermGen(Permanent) 영역이 Heap영역에서 빠지고 Native Memory라는 공간(JVM이 아닌 OS에서 관리하는 메모리 공간)으로 MetaSpace라는 이름으로 변경되어 들어갔다.  
    MetaSpace는 JVM에서 관리하고 실행하는 모든 Class들에 대한 MetaData 즉 해당 클래스에 대한 모든 정보를 저장하는 곳으로써, 클래스의 구조, 메서드에 대한 MetaData, 어노테이션 등의 정보를 담고 있다.

    </br>

  Heap영역은 모든 쓰레드가 공유하는 영역으로, Java8 이후부터 Eden, Survivor1,2와 Old 3개의 영역으로 나눠지게 되었으며, 처음에 객체가 만들어질 때에는 Eden영역에 생성이 되고, 이후 Eden영역이 꽉찰 경우에는 Survivor1이나 2중에 빈 공간이 있는 곳으로 옮겨진다.

  Eden영역과 Survivor1,2에서는 Minor GC가 실행되면서 사용되지 않는 객체는 제거하고, 살아남는 객체의 경우 Age Bit라는 값을 카운팅한다.

  Age Bit가 일정 카운트 넘어갈 경우에는 Old영역으로 넘어가지며, 또한 객체의 크기가 클 경우에도 Old영역으로 넘어가진다.

  Old영역에서는 Major GC가 동작이 되며, Minor GC보다는 빈도수가 적다.

  GC가 실행이 될 때 실행이 되는 쓰레드를 제외한 모든 쓰레드는 동작이 멈추며 이를 Stop The World라고 표현하는데 GC를 커스텀(성능 개선)하는 이유 중 하나가 바로 이 Stop The World의 시간을 줄이기 위해서라고 한다.

    </br>

---

# **_참고_**

- https://doozi0316.tistory.com/entry/1%EC%A3%BC%EC%B0%A8-JVM%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%B4%EB%A9%B0-%EC%9E%90%EB%B0%94-%EC%BD%94%EB%93%9C%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%8B%A4%ED%96%89%ED%95%98%EB%8A%94-%EA%B2%83%EC%9D%B8%EA%B0%80 [JVM 구조]
- https://asfirstalways.tistory.com/158 [JVM 구조]
- https://jaemunbro.medium.com/java-metaspace%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90-ac363816d35e [metaspace에 대해]
