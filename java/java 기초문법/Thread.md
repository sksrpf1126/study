# **_Thread_**

프로그램을 실행하면 프로세스 즉, 실행중인 프로그램을 프로세스라고 하고 OS로부터 필요한 메모리 공간, 자원을 할당받는다.

프로세스는 메모리, 자원 그리고 쓰레드로 구성이 되어 있으며, **_쓰레드_** 란 프로세스의 자원을 활용하여 작업을 수행하는 역할이다.(일꾼의 개념)

하나의 쓰레드는 싱글쓰레드, 두 개 이상의 쓰레드를 사용하는 프로세스를 멀티쓰레드 프로세스라고 한다.

</br>

---

## **_멀티쓰레딩의 장단점_**

- 장점  
  CPU의 사용을을 향상시킨다.  
  자원을 효율적으로 사용할 수 있다.  
  사용자에 대한 응답성이 향상된다.  
  작업이 분리되어 코드가 간결해진다.

- 단점  
  동기화 문제를 고려해야 한다.  
  교착상태 문제를 고려해야 한다.

  </br>

**_교착상태란?_**

두 쓰레드가 각각 자원을 점유한 상태에서 상대편의 자원을 사용하려고 기다리느라 작업이 멈춘 상태로 대기하는 상태를 말한다.

</br>

---

## **_쓰레드 구현방법_**

자바에서 쓰레드를 구현하는 방법은 2가지가 있다.

첫 번째. Thread 클래스를 상속받는다.  
두 번째. Runnable 인터페이스를 구현한다.

일반적으로는 Thread 클래스를 상속받는 경우에 다른 클래스를 상속받을 수 없기 때문에 Runnable 인터페이스를 구현하는 형태로 쓰레드를 구현한다.

```java
class MyThread extends Thread{
  public void run(){}
}


class MyThread2 implements Runnable{
  public void run(){}
}
```

Runnable 인터페이스에는 오로지 run() 추상메서드만 정의되어 있기에 구현시킬때 해당 run만 작성하면 된다.

이를 통해 java에서 Thread를 사용하는 방법은 위 2가지 중 하나의 방식을 택해서 run 메서드에 작업 내용만 작성하면 된다.

</br>

```java
public class ThreadEx1 {
    public static void main(String[] args) {
        MyThread1 thread1 = new MyThread1();
        Runnable r = new MyThread2();

        //Runnable을 통해 Thread를 구현할 경우에는 위에서 Runnable에 대한 참조변수를 만들고
        // new Thread(Runnable r);과 같이 Thread 클래스에 생성자로 넘겨줘야 한다.
        Thread thread2 = new Thread(r);

        thread1.start();
        thread2.start();

//        thread1.start(); IllegalThreadStateException 예외 발생
    }
}

class MyThread1 extends Thread {
    @Override
    public void run() {
        for(int i = 0; i < 5; i++){
            System.out.println(super.getName());
        }
    }
}

class MyThread2 implements Runnable {

    @Override
    public void run() {
        for(int i = 0; i < 5; i++){
            System.out.println(Thread.currentThread().getName());
        }
    }
}

실행결과 :
Thread-0
Thread-0
Thread-0
Thread-0
Thread-0
Thread-1
Thread-1
Thread-1
Thread-1
Thread-1
```

쓰레드 사용 예시이며, Runnable을 구현한 클래스로 쓰레드를 생성하기 위해서는 우선 Runnable 타입의 참조변수를 만들고, new Thread의 생성자로 넘겨주어 Thread 인스턴스를 만들어야 한다.

start()가 Thread의 실행을 시키는 메서드이며, 호출되었다고 바로 실행이 되는 것이 아니라 일단 실행대기 상태에 있다가 자신의 차례가 되어야 실행이 된다. (실행대기중인 쓰레드가 앞에 없다면 바로 실행)

**_쓰레드의 실행순서는 OS의 스케줄러가 작성한 스케줄에 의해 결정이 된다._**

또한, start 메서드는 단 한번만 실행이 가능하며, 또 호출할 경우에는 예외가 발생한다. 그래서 같은 Thread로 또 호출하고 싶다면 새롭게 쓰레드를 생성후에 호출해야 한다.

</br>

---

## **_start()와 run()차이_**

Thread를 실행시킬 때 start 메서드를 호출하였는데, run이라는 메서드를 호출하면 되지 않는가? 라고 생각할 수도 있다.  
하지만 run이라는 메서드를 호출하게 되면 생성한 Thread가 작업을 하기위한 작업 공간을 따로 만들지 않고, 그저 run메서드만을 실행시킨다.

```java
public class ThreaadEx2 {
    public static void main(String[] args) {
        Runnable r = new MyThreadEx2();
        Thread thread = new Thread(r);

        thread.run();

        System.out.println("-----------");

        thread.start();
    }
}

class MyThreadEx2 implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}

실행결과 :
main
-----------
Thread-0
```

모든 쓰레드는 독립적인 작업을 수행하기 위해 자신만의 호출스택을 필요로 한다. (이 조건이 충족이 되어야 쓰레드를 사용한다고 할 수 있음)

하지만 위처럼 run 메서드를 실행시켰을 때의 쓰레드는 메인쓰레드에서 동작이 된다. 쓰레드에 대한 독립적인 공간을 만들지 않았다는 것이다.

반대로 start 메서드를 호출할 때에는 쓰레드명이 변경되었다. 자신만의 호출스택이 만들어져서 거기서 run 메서드가 실행이 된 것이다.

정리하면, run 메서드는 그저 클래스에 정의한 메서드만을 실행시킬 뿐이고, start 메서드는 위에 예시로 main 메서드에서 실행이 되고, 그 후 새로운 작업공간(Thread-0 이란 이름)을 만들어서 거기서 run메서드를 수행한 것이다.

</br>

---

## **_main Thread_**

main 메서드가 실행될 때에도 하나의 쓰레드 위에서 동작하며, 해당 쓰레드의 이름을 main Thread라 한다.

지금까지(싱글 쓰레드)는 main 메서드 수행이 끝나면 프로그램이 종료가 되었지만, 멀티 쓰레드 환경에서는 main 메서드 수행이 끝나도 다른 쓰레드가 실행 중이라면 프로그램은 종료가 되지 않는다.

```java
public class ThreadEx3 {
    public static void main(String[] args) {
        Runnable r = new MyThreadEx3();
        Thread thread = new Thread(r);

        thread.start();

        for(int i = 0; i < 5; i++){
            long total = 0;
            long t = System.currentTimeMillis();

            for(int j = 0; j <10000000; j++){
                total += j;
            }

            System.out.println(Thread.currentThread().getName() + " : " + (System.currentTimeMillis() - t) + "ms");
        }

        System.out.println(Thread.currentThread().getName() + " end!!!!");
    }
}

class MyThreadEx3 implements Runnable{

    @Override
    public void run() {
        for(Integer i = new Integer(0); i < 10; i++){
            long total = 0;
            long t = System.currentTimeMillis();

            for(int j = 0; j <10000000; j++){
                total += j;
            }

            System.out.println(Thread.currentThread().getName() + " : " + (System.currentTimeMillis() - t) + "ms");
        }
        System.out.println(Thread.currentThread().getName() + " end!!!!");
    }
}

실행결과 :
main : 6ms
Thread-0 : 7ms
main : 11ms
main : 3ms
Thread-0 : 16ms
main : 3ms
Thread-0 : 3ms
main : 4ms
Thread-0 : 4ms
Thread-0 : 3ms
main end!!!!
Thread-0 : 4ms
Thread-0 : 3ms
Thread-0 : 3ms
Thread-0 : 4ms
Thread-0 : 3ms
Thread-0 end!!!!

Process finished with exit code 0 //프로그램 종료

```

두개의 쓰레드가 실행중일 때 작동방식을 나타내며, 위처럼 main 메서드가 전부 수행이 마쳤지만 프로그램은 종료가 되지 않고 다른 쓰레드의 작업을 진행한다. 이후 전부 완료가 된 후에 프로그램이 종료가 된 것을 확인할 수 있다.

</br>

---

## **_싱글쓰레드와 멀티쓰레드_**

```java
public class ThreadEx4 {
    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();

        for(int i = 0; i < 300; i++){
            System.out.printf("%s", new String("-"));
        }

        System.out.printf("소요시간 1번 : " + (System.currentTimeMillis() - startTime));

        for(int i = 0; i < 300; i++){
            System.out.printf("%s", new String("|"));
        }

        System.out.print("소요시간 2번 : " + (System.currentTimeMillis() - startTime));
    }
}

실행결과 :
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------소요시간 1번 : 33||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||소요시간 2번 : 55
```

</br>
 
 위는 싱글쓰레드로 연산을 진행한 것

</br>

```java
public class ThreadEx4_2 {
    static long startTime = 0;
    public static void main(String[] args) {

        Thread th = new MyThreadEx4_2();
        th.start();
        startTime = System.currentTimeMillis();

        for(int i = 0; i < 300; i++){
            System.out.printf("%s", new String("-"));
        }
        System.out.print(" 소요시간 1번 : " + (System.currentTimeMillis() - ThreadEx4_2.startTime));
    }
}

class MyThreadEx4_2 extends Thread {
    @Override
    public void run() {
        for(int i = 0; i < 300; i++){
            System.out.printf("%s", new String("|"));
        }

        System.out.print(" 소요시간 2번 : " + (System.currentTimeMillis() - ThreadEx4_2.startTime));
    }
}

실행결과 :
-------------------------------------------------------------------------|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||--------||||||||||||||||||||--------------------||||||||||||||||||-----|||-||---------------------------|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||-------------|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||--------------------------------------------------------------------------------------------------------------------------------------------------------- 소요시간 2번 : 37 소요시간 1번 : 41
```

위는 멀티쓰레드로 진행한 것이다.

간단한 연산은 컨텍스트 스위칭(쓰레드 간의 작업 전환)에 의해 멀티쓰레드보다 싱글쓰레드가 더 빠르다고는 하지만 이것도 싱글코어 또는 성능이 좋지 않은 CPU만 해당되는 사항인 것 같다.  
싱글코어에서의 멀티쓰레드의 작업 진행 방식은 A와 B 2개의 작업이 있을 경우에 A 10%, 작업 전환 후 B 10% 이런 식으로 동시에 작업을 수행하지 않고 쓰레드를 전환해 가면서 작업을 하다보니 싱글 쓰레드보다 성능이 좋지 않다고 하는 것이다.

요즘은, 나오는 하드웨어들이 전부 다 좋은 성능을 가지고 있기 때문에(멀티 코어) 쓰레드들의 작업을 동시에 진행이 가능하기 때문에 간단한 작업에서도 싱글 쓰레드보다 성능이 좋은 것 같다.

위의 멀티쓰레드 실행결과를 보면 출력하는 값들이 왔다갔다 하는 경우가 많은데 그 이유는 멀티 코어에서는 쓰레드들을 동시에 작업하다 보니 화면에 출력하는 명령어를 앞다투어 사용할려고 하다보니 위처럼 결과가 나오는 것이다.

</br>

### **_싱글 쓰레드와 멀티쓰레드 장단점_**

참고 : https://goodbyeanma.tistory.com/15

**_싱글쓰레드_**

장점

- 컨텍스트 스위칭을 하지 않는다.
- 동기화 및 교착상태를 고려하지 않아도 된다.
- 간단한 연산은 멀티쓰레드보다 더 빠를 수 있다.
- 프로그래밍 난이도가 낮고, 메모리를 적게 사용한다.

단점

- CPU를 효율적으로 사용하지 못한다.
- 연산량이 많을 경우 다른 작업을 진행하지 못하고, 해당 작업이 완료될 때까지 기다려야 한다.
- 싱글쓰레드는 에러처리를 하지 못하는 경우 프로그램이 종료가 된다.

</br>

**_멀티쓰레드_**

장점

- CPU의 사용을을 향상시킨다.
- 자원을 효율적으로 사용할 수 있다.
- 사용자에 대한 응답성이 향상된다.
- 작업이 분리되어 코드가 간결해진다.

단점

- 동기화 문제를 고려해야 한다.
- 교착상태 문제를 고려해야 한다.
- 컨텍스트 스위칭에 의해 더 느려질 수 도 있다.

</br>

---

## **_쓰레드의 I/O 블락킹_**

싱글쓰레드인 경우에는 A 작업은 사용자한테 10개의 숫자를 입력받고, B는 A와는 관계없는 내용을 출력하는 작업일 때 A가 끝날때까지 B는 대기해야 한다.

멀티쓰레드인 경우에는 A와 B의 작업을 따로 구분할 수 있다.
input과 output을 구분짓는 것이다.

</br>

---

## **_쓰레드의 우선순위_**

쓰레드는 우선순위(priority)라는 속성(멤버변수)를 가지고 있고 이를 조작하여, 쓰레드간의 우선순위를 지정할 수 있다. 1~10 까지의 값을 가지며, main 메서드는 디폴트로 5의 값을 가진다.

쓰레드의 초기 우선순위는 쓰레드를 생선한 쓰레드로부터 상속받는다. (main 메서드에서 만든 쓰레드들은 디폴트로 5의 값)

```java
public class ThreadEx5 {
    public static void main(String[] args) {
        ThreadEx5_1 th1 = new ThreadEx5_1();
        ThreadEx5_2 th2 = new ThreadEx5_2();

        th2.setPriority(7);

        System.out.println("th1 우선순위 : " + th1.getPriority());
        System.out.println("th2 우선순위 : " + th2.getPriority());

        th1.start();
        th2.start();
    }
}

class ThreadEx5_1 extends Thread{
    @Override
    public void run() {
        for(int i = 0; i < 300; i++){
            System.out.print("-");
            for(int j = 0; j < 10000000; j++){}
        }
    }
}

class ThreadEx5_2 extends Thread{
    @Override
    public void run() {
        for(int i = 0; i < 300; i++){
            System.out.print("|");
            for(int j = 0; j < 10000000; j++){}
        }
    }
}

실행결과 :
th1 우선순위 : 5
th2 우선순위 : 7
-||-||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||--------------||||||||||||||||||||||||||||||||||||||||||||||||||-----|||||-||||||---------|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||------|||||||||||||||||||||||||-------------------------------------------------------------------------------------------|||||||||||||||||||----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

가중치에 따라 동작하는 방식을 나타낸 것이다.

자바의 정석 책에서는 멀티코어일 때 우선순위를 둬도 영향 없이 -|-|-|-|.... 이런식으로 반복되어 나온다.  
즉, 멀티코어라 해도 OS마다 다른 방식으로 스케쥴링 하기도 하고, JVM 구현을 직접 확인해 봐야할 수 도 있다. 물론, 최종적으로 결졍 하는건 OS이기 때문에 JVM 확인만으로는 정확한 확인이 어렵다.

```java
여러번 실행했을 때 나온 결과 중 하나이다.
-|-|----------------------------------------------------------------------------------------------------------------------------|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||--------------------------|--|||||||||||----|||----------------------------------------------------------------------||||||||----------------------------------|||||||||-----||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||--------||||-------------------------|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||
```

위처럼 우선순위가 절대적인 것이 아니다.

</br>

---

## **_쓰레드 그룹_**

파일을 그룹별로 묶어서 관리하듯이, 쓰레드 또한 관련된 쓰레드끼리 묶어서 관리할 수 있다.

자신이 속한 쓰레드 그룹이나 하위 쓰레드 그룹은 변경할 수 있지만, 전혀 다른 그룹은 변경할 수 없다.

자바는 자동으로 어플리케이션이 실행이 될 때 main 쓰레드 그룹, system쓰레드 그룹을 만든다. 보통 JVM내에서 실행시킬려고 만든 쓰레드들은 system쓰레드 그룹에 만들어지고, 우리가 만든 쓰레드들은 보통 main쓰레드 그룹에 속한다.

여러 메서드들은 pg523~524에 제공

</br>

---

## **_데몬 쓰레드_**

데몬쓰레드는 일반쓰레드(데몬쓰레드를 제외한)의 작업을 돕는 쓰레드이다.  
일반쓰레드가 모두 종료가 되면 데몬쓰레드 모두가 강제종료 된다.  
데몬쓰레드가 쓰이는 경우는 가비지컬렉터, 워드나 한글 같은 프로그램에서의 자동저장, 화면자동갱신 등이 있다.  
데몬쓰레드에서 생성한 쓰레드들은 전부 데몬쓰레드가 된다.

```java
public class DaemonThreadEx {
    static boolean autoSave = false;
    public static void main(String[] args) {
        Thread th1 = new DaemonThread();
        th1.setDaemon(true);
        th1.start();

        for(int i = 0; i < 10; i++){
            try {
                Thread.sleep( 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(i);
            if(i == 4) autoSave = true;
        }
    }


}

class DaemonThread extends Thread {
    @Override
    public void run() {
        while(true){
            try {
                Thread.sleep(2 * 1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            if(DaemonThreadEx.autoSave) this.autoSave();
        }
    }

    public void autoSave(){
        System.out.println("작업파일이 자동저장되었습니다.");
    }
}

실행결과 :
0
1
2
3
4
작업파일이 자동저장되었습니다.
5
6
작업파일이 자동저장되었습니다.
7
8
작업파일이 자동저장되었습니다.
9

```

위와 같이 데몬쓰레드를 사용하기 위해서는 start()를 호출하기 전에 호출하는 쓰레드가 데몬쓰레드라는 것을 지정해줘야 하기 때문에 setDaemon()을 호출해줘야한다. true면 데몬쓰레드 false면 일반쓰레드로 지정한다.

위 예시는 데몬쓰레드가 무한반복하면서 메인쓰레드에서 autosave가 true가 될 때 2초마다 한번씩 쓰레드가 작동이된다.

</br>

---

## **_sleep_**

sleep()란 static으로 선언되어 있으며, 넘겨준 시간동안 쓰레드를 일시정지상태로 만든다.  
시간이 지나거나, interrupt 메서드가 호출되면 실행대기상태로 들어가 순서를 기다린다.

interrupt 메서드는 해당 쓰레드에서의 InterrptedException 예외를 발생시켜서 쓰레드를 깨운다. 그래서 sleep 메서드를 호출할 때 try-catch문으로 감싸는 것이다.

```java
public class SleepEx {
    public static void main(String[] args) {
        SleepThread1 th1 = new SleepThread1();
        SleepThread2 th2 = new SleepThread2();

        th1.start(); th2.start();

        try {
            th1.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("<<main 종료>>");
    }
}

class SleepThread1 extends Thread {
    @Override
    public void run() {
        for(int i = 0; i < 300; i++){
            System.out.print("-");
        }
        System.out.println("<<th1 종료>>");
    }
}

class SleepThread2 extends Thread {
    @Override
    public void run() {
        for(int i = 0; i < 300; i++){
            System.out.print("|");
        }
        System.out.println("<<th2 종료>>");
    }
}

실행결과:
-----------------------------------------------------------------------------------------------------------------------------||||||||||||||||||||||||||||||---------------------------||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||-------------------------------------------------------------------||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||-|--------------------------------------------------------------------------------|||||||||<<th1 종료>>
<<th2 종료>>
<<main 종료>>
```

main 쓰레드에서 th1.sleep(2000)을 통해 2초동안 일시정지 상태로 만든 것 같지만, 실행결과를 보면 th1이 제일 빨리 종료가 되고 이후 th2와 main 쓰레드가 종료가 되었다. 왜냐하면 sleep은 호출한 쓰레드에서만 동작이 되기 때문이다.  
그렇기에 Thread.sleep()로 일시정지상태로 만들려고하는 쓰레드 내에서 동작을 시켜야 한다.

</br>

---

## **_interrupt_**

interrput 메서드는 sleep()나 join()에 의해 일시정지 상태인 쓰레드를 InterruptedException 예외를 발생시킴으로써, 일시정지상태를 벗어나게 하는 것이다. (그렇다고 일시정지된 쓰레드를 interrupt 하여도 재실행은 되지 않는다)

보통 stop()가 교착상태를 만들기 쉽기 때문에 deprecated가 되어서, 안전한 종료를 위해 interrupt를 사용한다.

```java
public class InterruptEx {
    public static void main(String[] args) {
        InterThread th1 = new InterThread();
        th1.start();


        try {
            for(int i = 0; i <100000000; i++){} //지연
            th1.interrupt();
            System.out.println("th1 isInterrupted1  " + th1.isInterrupted());
            Thread.sleep(5000);
            System.out.println("th1 isInterrupted2  " + th1.isInterrupted());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("<<main>> 종료");
    }
}

class InterThread extends Thread{
    @Override
    public void run() {
        try {
            System.out.println("th1 start! " + isInterrupted());
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            System.out.println("th1 InterruptedException 발생! ");
        }
        System.out.println("<<th1> 종료");
    }
}

실행결과:
th1 start! false
th1 isInterrupted1  true
th1 InterruptedException 발생!
<<th1> 종료
th1 isInterrupted2  false
<<main>> 종료
```

interrupt()가 호출이 되면 해당 쓰레드의 isInterrupted()값이 true가 된다. 이후 InterruptedException이 발생하는데 이러면 th1의 동작은 멈추고 catch문이 실행되고나서 하단의 코드들을 실행시킨뒤 종료가 된다.

**_interrupt()를 호출한다고 해서 쓰레드를 종료시키는 것이 아니다. 해당 메서드는 그저, interrupt 인스턴스(상태)값을 바꿀 뿐이다._**

</br>

---

## **_join() yield_**

join 메서드는 조인메서드의 대상이되는 쓰레드의 실행이 완료될때까지 해당 쓰레드는 대기 상태를 가진다. 파라미터로 시간을 설정할 수 있다.

```java

public class JoinEx {
    public static void main(String[] args) {
        Worker pizzaWorker = new Worker();
        pizzaWorker.start(); //내부에서 데몬쓰레드로 생성


        for(int i = 0; i < 30; i++){
            if(pizzaWorker.getLeftoverPizza() > 0){
                pizzaWorker.eatingPizza(); //피자를 한조각 먹는다.
                System.out.println("현재 남은 피자는 " + pizzaWorker.getLeftoverPizza() + "개 입니다.");
            }else{
                System.out.println("남은 피자가 존재하지 않습니다.");
            }

            //피자가 절반도 안남았을 경우 피자리필
            if(pizzaWorker.getLeftoverPizza() < 4){
                pizzaWorker.interrupt();
            }
        }
    }
}

class Worker implements Runnable {
    private final static int PIZZA = 8;
    private int leftoverPizza = 8;
    private Thread th;

    public Worker(){
        this.th = new Thread(this);
    }

    public void start(){
        th.setDaemon(true);
        th.start();
    }

    public void interrupt(){
        th.interrupt();
    }

    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(10 * 1000);
            } catch (InterruptedException e) {
                System.out.println("PIZZA refill go!");
            }
            this.refillPizza();
            System.out.println("피자를 리필했습니다.");
        }
    }

    public int getLeftoverPizza() {
        return this.leftoverPizza;
    }

    public void refillPizza(){
        this.leftoverPizza = PIZZA;
    }

    public void eatingPizza(){
        this.leftoverPizza -= 1;
    }
}

실행결과 :
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
현재 남은 피자는 2개 입니다.
현재 남은 피자는 1개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 0개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
현재 남은 피자는 2개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 1개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
현재 남은 피자는 2개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 1개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
PIZZA refill go!
피자를 리필했습니다.
```

위는 join 메서드를 적용하기 이전으로, 피자가 4개미만이 되면 다시 8개로 채운다. 하지만 위처럼 피자를 리필하는 쓰레드가 동작되는 동시에 main 쓰레드도 같이 동작이 되기 때문에 실행결과가 저렇게 나타난다.  
참고로 worker 쓰레드의 run이 무한반복이기 때문에 그냥 interrupt()를 호출해봤자, 상태값만 변경될 뿐이지 프로그램은 종료가 되지 않고 계속 실행중이다.  
stop 메서드는 추천하지 않는 방식이니 interrupt의 상태값을 통해 제어를 하는게 바람직한 방법이다.

</br>

```java
public class JoinEx {
    public static void main(String[] args) {
        Worker pizzaWorker = new Worker();
        pizzaWorker.start(); //내부에서 데몬쓰레드로 생성


        for(int i = 0; i < 30; i++){
            if(pizzaWorker.getLeftoverPizza() > 0){
                pizzaWorker.eatingPizza(); //피자를 한조각 먹는다.
                System.out.println("현재 남은 피자는 " + pizzaWorker.getLeftoverPizza() + "개 입니다.");
            }else{
                System.out.println("남은 피자가 존재하지 않습니다.");
            }

            //피자가 절반도 안남았을 경우 피자리필
            if(pizzaWorker.getLeftoverPizza() < 4){
                pizzaWorker.interrupt();
                pizzaWorker.join(1000); //피자리필시간을 제공
            }
        }
    }
}

class Worker implements Runnable {
    private final static int PIZZA = 8;
    private int leftoverPizza = 8;
    private Thread th;

    public Worker(){
        this.th = new Thread(this);
    }

    public void start(){
        th.setDaemon(true);
        th.start();
    }

    public void interrupt(){
        th.interrupt();
    }

    public void join(int ms){
        try {
            th.join(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(10 * 1000);
            } catch (InterruptedException e) {
                System.out.println("PIZZA refill go!");
            }
            this.refillPizza();
            System.out.println("피자를 리필했습니다.");
        }
    }

    public int getLeftoverPizza() {
        return this.leftoverPizza;
    }

    public void refillPizza(){
        this.leftoverPizza = PIZZA;
    }

    public void eatingPizza(){
        this.leftoverPizza -= 1;
    }
}


실행결과:
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
PIZZA refill go!
피자를 리필했습니다.
현재 남은 피자는 7개 입니다.
현재 남은 피자는 6개 입니다.
현재 남은 피자는 5개 입니다.
현재 남은 피자는 4개 입니다.
현재 남은 피자는 3개 입니다.
PIZZA refill go!
피자를 리필했습니다.

```

위와같이 join메서드를 통해 인터럽터를 호출함과 동시에 해당 쓰레드 작업시간을 제공해주면, 의도한대로 작동이 이루어지는 것을 볼 수 있다.

### **_yield()_**

yield 메서드는 동작중인 쓰레드 본인이 자신에게 주어진 실행시간을 반납하고 다음 쓰레드에게 양보하는 메서드이다.

</br>

---

## **_쓰레드 동기화(synchronization)_**

</br>

```java

public class synchronizationEx1 {
    public static void main(String[] args) {
        Runnable r = new synchroThread1();
        new Thread(r).start();
        new Thread(r).start();
    }
}

class Accout {
    private int balance = 1000;

    public int getBalance() {
        return this.balance;
    }

    public void withdraw(int money){
        if(this.balance >= money){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.balance -= money;
        }
    }
}

class synchroThread1 implements Runnable{
    Accout acc = new Accout();

    @Override
    public void run() {
        while(acc.getBalance() > 0){
            int money = (int)(Math.random() * 3 + 1) * 100;
            acc.withdraw(money);
            System.out.println("balance : " + acc.getBalance());
        }
    }
}

실행결과:
balance : 400
balance : 700
balance : 100
balance : -100
```

멀티쓰레드에서 싱글코어는 동시성에 의해서 A쓰레드, B쓰레드를 컨텍스트 스위칭을 진행하면서 작업을 한다. 이 때 A쓰레드가 작업하던 공유 자원이 잠시 B쓰레드로 제어권이 넘어가면서 B쓰레드에서 A쓰레드와 같은 공유 자원을 건드리게 되면, 의도와는 다른 값이 나오게 되는 것이다.

위 코드의 경우에는 멀티코어로 병렬성으로 작동되다보니, 쓰레드 2개가 동시에 같은 자원을 가지고 작업을 하다보니까 원래 로직은 0원 이하일때는 동작이 안되어서 마이너스 금액이 찍히지 않는게 정상이지만 실행결과를 보면 마이너스 금액이 나오게 된다.

그래서 이러한 문제를 방지하기 위하여 **_쓰레드 동기화_** 를 해주어야 하는데 쓰레드 동기화란 해당 쓰레드가 진행중인 작업에 대해서는 다른 쓰레드가 간섭을 못하게 막는 것이다.

synchronized 블럭을 이용하는 방식이 있고, JDK1.5부터는 locks와 atomic을 통해 다양한 방식으로 동기화가 가능해졌다.

위 방식들은 임계 영역과 잠금을 활용하는 방식인데, 객체마다 lock을 하나씩 가지고 있으며, 해당 lock을 가진 쓰레드만이 임계 구역으로 지정한 코드들에 접근하여 작업을 할 수 있다. 이후에는 반납을 해야한다.

</br>

```java
    public synchronized void withdraw(int money){
        if(this.balance >= money){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.balance -= money;
        }
    }

실행결과 :
balance : 700
balance : 600
balance : 300
balance : 200
balance : 200
balance : 100
balance : 100
balance : 0
balance : 0
```

위와 같이 withdraw 메서드에 synchronized 키워드를 붙여줌으로써, 임계 구역을 생성하였고 그리하여 하나의 쓰레드만이 위의 작업을 진행할 수 있게 되는 것이다.

</br>

---

```java
package chapter13;

public class synchronizationEx1 {
    public static void main(String[] args) {
        Runnable r = new synchroThread1();
        new Thread(r).start();
        new Thread(r).start();
    }
}

class Accout {
    private int balance = 1000;

    public int getBalance() {
        return this.balance;
    }

    public synchronized void withdraw(int money){
        System.out.println("임계구역 안 : " + Thread.currentThread().getName());
        if(this.balance >= money){
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.balance -= money;
        }
    }

    public void test(){
        System.out.println("임계구역 전 : " + Thread.currentThread().getName());
    }
}

class synchroThread1 implements Runnable{
    Accout acc = new Accout();

    @Override
    public void run() {
        while(acc.getBalance() > 0){
            int money = (int)(Math.random() * 3 + 1) * 100;
            acc.test();
            acc.withdraw(money);
            System.out.println("balance : " + acc.getBalance() + "  " + Thread.currentThread().getName());
        }
    }
}

실행결과:
임계구역 전 : Thread-1
임계구역 전 : Thread-0
임계구역 안 : Thread-1
임계구역 안 : Thread-0
balance : 900  Thread-1
임계구역 전 : Thread-1
balance : 700  Thread-0
임계구역 전 : Thread-0
임계구역 안 : Thread-1
balance : 600  Thread-1
임계구역 전 : Thread-1
임계구역 안 : Thread-0
balance : 500  Thread-0
임계구역 안 : Thread-1
임계구역 전 : Thread-0
balance : 200  Thread-1
임계구역 안 : Thread-0
임계구역 전 : Thread-1
임계구역 안 : Thread-1
balance : 200  Thread-0
임계구역 전 : Thread-0
임계구역 안 : Thread-0
balance : 100  Thread-0
balance : 100  Thread-1
임계구역 전 : Thread-1
임계구역 안 : Thread-1
임계구역 전 : Thread-0
balance : 0  Thread-1
임계구역 안 : Thread-0
balance : 0  Thread-0
```

위의 코드를 분석해 보면, 처음에 임계구역 전에는 두 쓰레드가 동시에 로그를 찍었고, 이후에 임계구역 안이 2개가 찍혀서 동시에 접근한 것 처럼 보이지만, 실행해보면 1초차이로 하나만 먼저 찍히고, 이후에 다른쓰레드가 찍힌다. 이를통해 임계구역 코드를 만나면 하나는 실행중이고, 하나는 실행이 완료가 될 때까지 대기를 하는것을 볼 수 있다.

</br>

---

## **_wait()와 notify()_**

계좌라는 객체에서 출금이라는 기능을 사용할려는 쓰레드가 하나 있다. 하지만 계좌에는 출금을 하기 위한 돈이 없고, 해당 쓰레드는 출금을 하기 위한 돈이 계좌에 들어올때까지 계속 반복문을 돌며 임계구역에서 lock을 가지고 실행중이다.

위와 같은 상황에서 같은 계좌 객체의 다른 기능을 사용할려고 하는데, lock을 이미 위 쓰레드가 점유하고 있어서 해당 기능들도 사용할 수 없는 것이다.  
그래서 이러한 문제를 해결하기 위해 등장한 것이 wait()와 notify()이다.

wait()는 호출이 되면 실행 중이던 쓰레드는 해당 호출된 시점에서 작업을 멈추고, 해당 객체의 대기실(waiting pool이라 하며, 모든 객체가 하나씩 가지고 있다)에서 통지를 기다린다. 이후 통지가 와서 자신이 lock을 점유하면 멈춘 시점으로 가서 다시 이어서 작업을 한다.

notify()는 호출이 되면 해당 객체의 대기실에 있던 모든 쓰레드 **_중에서_** 임의의 쓰레드에게만 통지를 한다.  
notifyAll()는 모든 쓰레드에게 통지를 한다. 하지만, lock은 하나 밖에 없으므로 lock을 가지고 서로 경쟁을 한다. 그 후 단 하나의 쓰레드만이 실행.

```java
public class WaitAndNotifyEx {
    public static void main(String[] args) throws Exception {
        Table table = new Table();

        new Thread(new Cook(table), "COOK").start();
        new Thread(new Customer(table, "donut"), "CUST1").start();
        new Thread(new Customer(table, "burger"), "CUST2").start();

        Thread.sleep(5000);
        System.exit(0);
    }
}

class Customer implements Runnable{
    private Table table;
    private String food;

    public Customer(Table table, String food){
        this.table = table;
        this.food = food;
    }

    boolean eatFood(){return table.romove(this.food);}

    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            String name = Thread.currentThread().getName();

            if(this.eatFood()){
                System.out.println(name + " ate a " + food);
            }else{
                System.out.println(name + " failed to eat.");
            }
        }
    }
}

class Cook implements Runnable{
    private Table table;

    public Cook(Table table){
        this.table = table;
    }

    @Override
    public void run() {
        while (true){
            int idx = (int) (Math.random() * table.dishNum());
            table.add(table.dishNames[idx]);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Table{
    String[] dishNames = {"donut", "donut", "burger"};
    final int MAX_FOOD = 6;
    private ArrayList<String> foodList = new ArrayList<>();

    public synchronized void add(String dish){
        if(foodList.size() >= MAX_FOOD){
            return;
        }
        foodList.add(dish);
        System.out.println("Dishes: " + foodList.toString());
    }

    public boolean romove(String dishName){
        synchronized (this){
            while(foodList.size() == 0){
                String name = Thread.currentThread().getName();
                System.out.println(name + " is waiting.");
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            for(int i = 0; i < foodList.size(); i++){
                if(dishName.equals(foodList.get(i))){
                    foodList.remove(i);
                    return true;
                }
            }
        }
        return false;
    }

    public int dishNum(){return dishNames.length;}
}

실행결과:
Dishes: [donut]
CUST2 is waiting.
CUST1 ate a donut
CUST2 is waiting.
CUST2 is waiting.
CUST2 is waiting.
CUST2 is waiting.
CUST2 is waiting.
CUST2 is waiting.
CUST2 is waiting.
CUST2 is waiting.
CUST2 is waiting.
```

위 코드는 빵을 만드는 요리사와 그 빵을 먹는 고객 그리고 빵이 나와서 진열되는 테이블이라는 클래스로 이루어져 있다.  
메인쓰레드에서는 table 객체를 하나만 만들어서 요리사 쓰레드와 고객 쓰레드 2개가 같은 Table 객체를 사용하도록 되어 있다. 이후 메인쓰레드는 5초간 멈춰있다가 프로그램을 종료한다.

결국 위 실행결과는 5초간의 실행결과로써, 요리사는 처음에 donut이라는 빵을 만들었고, 위는 출력결과가 좀 꼬였지만 동작은 CUST1이 동작이 먼저 이루어져서 donut이라는 빵을 먹었고, 이후 lock을 반환한다. 그 후 CUST2가 해당 lock을 차지하여 테이블에 있는 빵을 먹으려고 하는데 단 하나도 존재하지 않기에 무한정 waiting만 진행중이며, lock을 반환하지 않는다.

이러한 문제에 의해 요리사도, 고객1도 table 객체의 lock을 얻을 수 없어서 어떠한 작업도 할 수 없는 상황이 발생하는 것이다.

</br>

---

wait()와 notify()를 추가하여 작성한 로직

```java
package chapter13;

import java.util.ArrayList;

public class WaitAndNotifyEx2 {
    public static void main(String[] args) throws Exception {
        Table2 table2 = new Table2();

        new Thread(new Cook2(table2), "COOK").start();
        new Thread(new Customer2(table2, "donut"), "CUST1").start();
        new Thread(new Customer2(table2, "burger"), "CUST2").start();

        Thread.sleep(2000);
        System.exit(0);
    }
}

class Customer2 implements Runnable{
    private Table2 table;
    private String food;

    public Customer2(Table2 table, String food){
        this.table = table;
        this.food = food;
    }

    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            String name = Thread.currentThread().getName();

            table.romove(this.food);
            System.out.println(name + " ate a " + food);
        }
    }
}

class Cook2 implements Runnable{
    private Table2 table;

    public Cook2(Table2 table){
        this.table = table;
    }

    @Override
    public void run() {
        while (true){
            int idx = (int) (Math.random() * table.dishNum());
            table.add(table.dishNames[idx]);
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Table2{
    String[] dishNames = {"donut", "donut", "burger"};
    final int MAX_FOOD = 6;
    private ArrayList<String> foodList = new ArrayList<>();

    public synchronized void add(String dish){
        if(foodList.size() >= MAX_FOOD){
            String name = Thread.currentThread().getName();
            System.out.println(name + " is waiting");
            try {
                wait(); //table에 음식이 꽉차 있을 경우에 lock을 반환하고 대기
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return;
        }
        foodList.add(dish);
        notify(); //음식을 만들었을 경우 다른 쓰레드에게 통보
        System.out.println("Dishes: " + foodList.toString());
    }

    public void romove(String dishName){
        synchronized (this){
            String name = Thread.currentThread().getName();

            while(foodList.size() == 0){
                System.out.println(name + " is waiting.(테이블에 음식이 X)");
                try {
                    wait(); //테이블에 빵이 없을 경우 lock을 반환하고 대기
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            while (true){

                for(int i = 0; i < foodList.size(); i++){
                    if(dishName.equals(foodList.get(i))){
                        foodList.remove(i);
                        notify(); //음식을 먹을때마다 waiting pool에 있는 다른 쓰레드들에게 통보(누가 lock을 가질지 모름)
                        return;
                    }
                }

                try{
                    System.out.println(name + " is waiting.(원하는 음식 X)");
                    wait(); //lock을 반환하고 대기
                    Thread.sleep(500);
                }catch (InterruptedException e){
                }
            }
        }
    }

    public int dishNum(){return dishNames.length;}
}

실행결과 :
Dishes: [donut]
Dishes: [donut, burger]
Dishes: [donut, burger, burger]
Dishes: [donut, burger, burger, donut]
Dishes: [donut, burger, burger, donut, donut]
Dishes: [donut, burger, burger, donut, donut, donut]
COOK is waiting
CUST1 ate a donut
CUST2 ate a burger
CUST2 ate a burger
CUST1 ate a donut
Dishes: [donut, donut, donut]
Dishes: [donut, donut, donut, burger]
Dishes: [donut, donut, donut, burger, burger]
Dishes: [donut, donut, donut, burger, burger, donut]
COOK is waiting
CUST2 ate a burger
CUST2 ate a burger
CUST1 ate a donut
Dishes: [donut, donut, donut, donut]
Dishes: [donut, donut, donut, donut, donut]
Dishes: [donut, donut, donut, donut, donut, burger]
COOK is waiting
CUST2 ate a burger
CUST1 ate a donut
CUST2 is waiting.(원하는 음식 X)
CUST1 ate a donut
```

테이블이 꽉차있을 때에 요리사는 wait 즉, lock을 반환하고 멈춰있으며, 요리를 만들었을 때에는 notify를 통해 wait를 호출하여 멈춰있는 다른 쓰레드들 중 하나에게 통지를 한다.  
고객들은 테이블에 음식이 없거나, 원하는 음식이 없을 경우에는 wait를 하고 음식을 먹을때에 notify를 호출하여 wait를 호출하여 멈춰있는 다른 쓰레드들 중 하나에게 통지를 한다.

이를 통해 적절한 때에 lock을 반환하여 구현함에 따라 문제없이 동작이 되는것처럼 보인다.

하지만, 여기에서도 하나의 문제가 있다. notify가 바로 누구에게 통지를 할지 모른다는 것인데, 만약 접부다 도넛만 있는 상태에서 버거를 먹는 고객은 원하는 음식이 없어서 wait를 하고 있고, 도넛을 먹는 고객은 하나를 먹고 notify를 호출하여 cook이나 다른 고객에게 통지를 하는데 이때 버거 먹는 고객에게 통지가 가서 lock을 차지하는 경우 해당 고객은 또 원하는 음식이 없어서 lock을 반환한다. 그럼 도넛 먹는 고객은 wait상태가 아니기에 해당 lock을 통해 다시 도넛을 먹는다.  
이게 계속 반복이 되어 요리사가 빵을 못만드는 상태가 오고, 원하는 음식이 없는 고객도 wait, 테이블에 빵이 없어 도넛 먹는 고객도 wait, 한번도 통지가 안와 빵을 못만든 요리사도 wait 즉 모두 통지가 올때까지 대기하고 있는 상태가 발생한다.

</br>

```java
Dishes: [donut]
COOK is waiting
CUST2 is waiting.(원하는 음식 X)
CUST1 ate a donut
CUST1 is waiting.(테이블에 음식이 X)
Dishes: [donut]
CUST2 is waiting.(원하는 음식 X)
COOK is waiting
```

위는 테이블에 빵이 하나만 들어갈 수 있도록 제한을 걸고 실행했을 때이다.

처음에 요리사는 빵을 만들고나서, sleep 시간이 적기 때문에 다른 고객들의 쓰레드가 실행되기전에 또 빵을 만들려고 하는거고 이미 테이블은 꽉차 있기 때문에 wait상태를 가진다.  
다음으로, CUST2 고객이 테이블에 있는 빵을 먹으려다가 버거가 없기에 wait상태에 빠진다.  
그다음 CUST1이 도넛을 먹었고, 통지를 했지만 lock을 또 자신이 차지하여서 다시 한번 빵을 먹으려다가 테이블에 빵이 없어 wait상테에 빠진다.  
그다음 lock은 통지를 받았지만 lock이 없어서 실행이 안되었던 COOK이 한번 더 빵을 만들었고, 이후 notify로 통지를 한다.  
두명의 고객 중 버거를 먹는 고객이 통지를 받아 lock을 가지고 빵을 먹으려 했지만 도넛이어서 결국 wait()빠진다.  
유일하게 wait상태가 아닌 COOK이 요리를 만들려 했지만, 테이블은 이미 꽉차있기에 자신 또한 wait상태에 빠진다.

결국 3개의 쓰레드 전부다 waiting pool에 있기에 이후에는 아무런 동작도 하지 않는다.

이렇게 통지를 오랫동안 못받는 현상을 **_기아현상_** 이라고 한다.

이를 해결하기 위해서 간단한 방법 중 하나가 notifyAll을 통해 모든 쓰레드들에게 통지를 날리는 것이다. 하지만 그러면 모든 쓰레드들이 하나의 lock을 두고 경쟁하는 **_경쟁상태_** 가 발생한다.

경쟁 상태를 개선하기 위해서는 요리사 쓰레드를 구별해서 통지하는 것이 필요하다. 이러한 Lock과 Condition을 이용하면, wait() & notify()로는 불가능한 선별적인 통지가 가능하게 된다.

</br>

---

## **_추가로 공부해야 할 것_**

Lock과 Condition에 대해  
기아현상과 경쟁상태에 대한 이론
