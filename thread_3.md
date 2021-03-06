# thread_3

- 쓰레드의 동기화

  - 동기화(synchronized) : 하나의 작업이 완전히 완료된 후 다른 작업을 수행하는 것
  - 비동기(asynchronous) : 하나의 작업 명령 이후 완료 여부와 상관없이 바로 다음 작업 명령을 수행하는 것

- 필요성 : 여러 쓰레드에서 공유하고 싶은 객체가 필요한 경우 동기화를 사용

- 동기화를 사용하지 않았을 때 발생하는 문제

  ```
  // 공유 객체
  class MyData {
      int data = 3;
  
      public void plusData() {
          int mydata = data;
  
          try {
              Thread.sleep(2000);
          } catch (InterruptedException e) {}
  
          data = mydata + 1;
      }
  }
  
  // 공유 객체를 사용하는 쓰레드
  class PlusThread extends Thread {
      MyData myData;
  
      public PlusThread(MyData myData) {
          this.myData = myData;
      }
  
      @Override
      public void run() {
          myData.plusData();
          System.out.println(getName() + "실행 결과 : " + myData.data);
      }
  }
  
  public class default_class {
      public static void main(String[] args) {
          // 공유 객체 생성
          MyData myData = new MyData();
  
          // plusThread 1
          Thread plusThread1 = new PlusThread(myData);
          plusThread1.setName("plusThread1");
          plusThread1.start();
  
          try {
              Thread.sleep(1000);
          } catch (InterruptedException e) {}
  
          //plusThread 2
          Thread plusThread2 = new PlusThread(myData);
          plusThread2.setName("plusThread2");
          plusThread2.start();
      }
  }
  ```

  ```
  plusThread1실행 결과 : 4
  plusThread2실행 결과 : 4
  ```

  객체를 공유하지 못하고 있음을 알 수 있다.

- 동기화 방법

  - 메서드 동기화

    - 동기화하고자 하는 메서드의 리턴 앞에 synchronized 키워드 추가

    - ```
      // 공유 객체 저장, 더하기 연산 클래스
      class MyData {
          int data = 3;
          public synchronized void plusData() {
              int mydata = data;
              try {
                  Thread.sleep(2000);
              } catch (InterruptedException e) {}
              data = mydata + 1;
          }
      }
      
      // 공유 객체를 사용할 쓰레드 클래스
      class PlusThread extends Thread {
          MyData myData;
          public PlusThread(MyData myData) {
              this.myData = myData;
          }
          @Override
          public void run() {
              myData.plusData();
              System.out.println(getName() + " 실행 결과 : " + myData.data);
          }
      }
      
      public class default_class {
          public static void main(String[] args) {
              // 공유 객체 생성
              MyData myData = new MyData();
      
              // plusThread 1
              Thread plusThread1 = new PlusThread(myData);
              plusThread1.setName("plusThread1");
              plusThread1.start();
      
              try {
                  Thread.sleep(1000);
              } catch (InterruptedException e) {}
      
              // plusThread 2
              Thread plusThread2 = new PlusThread(myData);
              plusThread2.setName("plusThread2");
              plusThread2.start();
          }
      }
      ```

      ```
      plusThread1 실행 결과 : 4
      plusThread2 실행 결과 : 5
      ```

  - 블록 동기화

    - 멀티 쓰레드를 사용하는 프로그램이라 하더라도 동기화 영역에서는 하나의 쓰레드만 실행할 수 있기 때문에 성능면에서 손실 발생

      따라서, 동기화 영역은 꼭 필요한 부분에 한정하여 적용하는 것이 좋음

    - 블록 동기화 : 메서드 전체를 동기화할 필요가 없는 부분을 제외하고 해당 부분만 동기화하는 것

    - ```
      // 공유 객체 생성, 더하기 연산 클래스
      class MyData {
          int data = 3;
          public void plusData() {
              // 블록 동기화
              synchronized (this) {
                  int myData = data;
                  try {
                      Thread.sleep(2000);
                  } catch (InterruptedException e) {}
                  data = myData + 1;
              }
          }
      }
      
      // 공유 객체를 사용하는 쓰레드 클래스
      class PlusThread extends Thread {
          MyData myData;
          public PlusThread(MyData myData) {
              this.myData = myData;
          }
          @Override
          public void run() {
              myData.plusData();
              System.out.println(getName() + " 실행 결과 : " + myData.data);
          }
      }
      
      public class default_class {
          public static void main(String[] args) {
              // 공유 객체 생성
              MyData myData = new MyData();
      
              // plusThread 1
              Thread plusThread1 = new PlusThread(myData);
              plusThread1.setName("plusThread1");
              plusThread1.start();
      
              try {
                  Thread.sleep(1000);
              } catch (InterruptedException e) {}
      
              // plusThread 2
              Thread plusThread2 = new PlusThread(myData);
              plusThread2.setName("plusThread2");
              plusThread2.start();
          }
      }
      ```

      ```
      plusThread1 실행 결과 : 4
      plusThread2 실행 결과 : 5
      ```

- 동기화 원리

  <img src="/Users/seongwon/workspace/study/Java/Java_study/images/thread_3-1.png" alt="thread_3-1" style="zoom:50%;" />

  - 사각형 박스는 같은 열쇠를 사용하는 기준으로 나누어져있다.

  - 동기화의 경우 같은 열쇠를 사용하면 동시에 사용할 수 없다.

    -----

  - 3개의 동기화 영역이 동일한 열쇠로 동기화 됐을 때

    ```
    // 공유 객체
    class MyData {
        synchronized void abc() {
            for(int i = 0; i <3; i++) {
                System.out.println(i + "sec");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {}
            }
        }
    
        synchronized void bcd() {
            for(int i = 0; i < 3; i++) {
                System.out.println(i + "초");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {}
            }
        }
    
        void cde() {
            synchronized(this) {
                for(int i = 0; i < 3; i++) {
                    System.out.println(i + "번째");
                    try {
                        Thread.sleep(1000);
                    } catch(InterruptedException e) {}
                }
            }
        }
    }
    
    public class default_class {
        public static void main(String[] args) {
            // 공유객체
            MyData myData = new MyData();
    
            // 3개의 쓰레드가 각각 메서드 호출
            new Thread() {
                public void run() {
                    myData.abc();
                };
            }.start();
    
            new Thread() {
                public void run() {
                    myData.bcd();
                };
            }.start();
    
            new Thread() {
                public void run() {
                    myData.cde();
                };
            }.start();
        }
    }
    ```

    ```
    0sec
    1sec
    2sec
    0번째
    1번째
    2번째
    0초
    1초
    2초
    ```

    3개의 메서드 모두가 같은 동기화 열쇠를 가지고 있기 때문에 동시에 실행이 되지 않고 있다는 것을 알 수 있다.

    * synchronized void 메서드() 의 경우 보이진 않지만 this 열쇠를 가지고 있다.

  - 동기화 메서드와 동기화 블록이 다른 열쇠를 사용할 때

    ```
    class MyData {
        synchronized void abc() {
            for(int i = 0; i < 3; i++) {
                System.out.println(i + "sec");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {}
            }
        }
    
        synchronized void bcd() {
            for(int i = 0; i < 3; i++) {
                System.out.println(i + "번째");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {}
            }
        }
    
        void cde() {
            synchronized(new Object()) {
                for(int i = 0; i < 3; i++) {
                    System.out.println(i + "초");
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {}
                }
            }
        }
    }
    
    public class default_class {
        public static void main(String[] args) {
            MyData myData = new MyData();
    
            new Thread() {
                public void run() {
                    myData.abc();
                };
            }.start();
    
            new Thread() {
                public void run() {
                    myData.bcd();
                };
            }.start();
    
            new Thread() {
                public void run() {
                    myData.cde();
                };
            }.start();
        }
    }
    ```

    ```
    0sec
    0초
    1sec
    1초
    2sec
    2초
    0번째
    1번째
    2번째
    ```

    sec, 초 -> 서로 다른 동기화 열쇠 -> 동시 실행 가능

    sec, 번째 -> 같은 동기화 열쇠 -> 동시 실행 불가

