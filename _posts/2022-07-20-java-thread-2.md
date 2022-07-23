---
title: Java's Thread - 2 (Basic)
author: Yongju Kwon
date: 2022-07-22 22:27:00 -0700
categories: []
tags: []
---

## ⒈ Background
오늘은 지난 글에 이어 현재 동작하고 있는 thread를 control 할 수 있는 세 가지 방법에 대해서 알아보려고 한다.

1. `Sleep`
2. `Interrupts`
3. `Joins`

하나씩 알아본 후에 세 가지 방법이 모두 포함된 example을 통해 어떻게 사용할 수 있는지 알아보자.

## ⒉ Sleep

`Thread.sleep`은 일정 시간동안 현재 실행 되고 있는 작업을 미루는 역할을 한다. 멈춘 작업이 실행되고 있던 프로세서를 다른 쓰레드나 어플리케이션이 이용하게 할 수 있다는 점에서 유용하다.

sleep 메소드는 두 가지가 존재 하는데 ¹ millisecond만 넣거나, ² millisecond와 nanosecond를 함께 넣을 수 있다. 후자의 경우 제공된 millisecond + nanosecond로 쓰레드를 멈출 시간을 지정할 수 있다. 

sleep 메소드를 사용할 때 가장 조심해야 할 점은 우리가 지정한 시간만큼 정확히 쓰레드를 멈춘다는 보장을 하지 못 한다는 점이다. 그 이유는 우리가 지정한 시간이 OS의 timer와 scheduler에 의존하기 때문이다. 간단하게 말하면, 현재 시스템이 바쁜 상태라면 쓰레드는 waiting 상태(아래 Thread life cycle 참고)에서 우리가 지정한 시간보다 더 오래 있을 수 있고, 반대의 경우에는 우리가 지정한 시간만큼 보다 정확하게 있을 것이다. 결론적으로, sleep 메소드는 언제든 우리가 지정한 시간만큼 100% 정확하게 동작하지 않는다는 걸 기억해두자.

![Thread Life Cycle](/assets/img/20220720/thread_life_cycle.jpeg)
_Thread life cycle (https://commons.wikimedia.org/)_

또한, sleep period 는 아래에서 알아 볼 `interrupts`에 의해 종료될 수 있다. 쓰레드가 sleep에 의해 waiting 상태일 때 다른 쓰레드에 의해 방해(interrupt) 받으면 `InterruptedException`을 던진다.

## ⒊ Interrupts

Interrupt는 쓰레드에게 멈추고, 다른 일을 해야한다는 일종의 신호이다. Interrupt에 어떻게 response 할 것 인지는 코드를 작성하는 개발자의 마음이다. 해당 쓰레드를 종료하는 것이 보통이고, 위에서 알아본 Thread.sleep이나 Thread.join() 등 여러가지 메소드들은 InterruptedException을 던진다.

각 쓰레드는 각자의 interrupt status를 알려주는 flag를 가지고 있고, 이 flag는 true 혹은 false로 설정될 수 있다. 기본값은 false이고 해당 쓰레드가 interrupted 되면 true로 설정된다.

Interrupted 될 쓰레드가 있다면 interruption을 핸들링 할 코드를 반드시 작성해야 한다.

Interrupt flag 를 확인할 수 있는 두 가지 메소드가 있는데,  ¹interrupted 과 ²isInterrupted 이다. 두 메소드에는 두 가지 큰 차이점이 있는데,

⑴ interrupted는 Thread 오브젝트의 static method이고 isInterrupted는 instance method이다. 
⑵ interrupted는 실행된 후 interrupt status flag를 clear 하지만 isInterrupted는 flag를 변경하지 않는다.

|   | Interrupted | isInterrupted  |
| :---: | :---: | :---:|
| Scope | Static method | Instance method |
| Clear flag | True  | False |

isInterrupted는 보통 logging이나 debugging 할 때 쓰이고, 대개는 interrupted를 사용하는 것이 권장된다. 이유는 isInterrupted는 당시의 flag snapshot을 주는 것이기 때문에 다시 실행했을 때 이미 outdated된 status를 줄 수 있기 때문이다. 또한 interrupted를 통해 interrupt status flag가 clear 됐다는 것은 개발자가 그에 대응하는 액션을 취했다는 의미이기 때문이다.

## ⒋ Joins

join 메소드는 현재 쓰레드에서 join 메소드가 불려진 쓰레드가 현재 하는 일을 완료할 때까지 기다려줄 수 있도록 한다. join은 sleep과 비슷한 점이 많다. 기다려줄 시간을 지정할 수도 있고 혹은 아무 argument 없이 쓰레드가 완료될 때까지 기다릴 수도 있다. 역시 우리가 지정한 시간을 100% 정확하게 보장 받을 수 없다는 것을 유의하자. 또한 interrupt에 InterruptedException으로 대응하는 점도 같다.

## ⒌ Example

sleep, interrupt과 join을 모두 이용한 example을 통해서 정리해보도록 하자. Comment를 따라서 차근차근 코드를 읽어보고 코드 블럭 밑 정리된 글을 읽어보자.
 
```java
public class SimpleThreads {
    static void threadMessage(String message) { // 쓰레드 정보와 메세지를 표현해 줄 helper method
        String threadName = Thread.currentThread().getName();
        System.out.format("%s: %s%n", threadName, message);
    }

    private static class MessageLoop implements Runnable { // 실제 실행될 thread
        public void run() {
            String importantInfo[] = { "Mares eat oats", "Does eat oats", "Little lambs eat ivy", "A kid will eat ivy too"};

            try { // 4초 텀을 두고 String을 프린트 한다
                for (int i = 0; i < importantInfo.length; i++) {
                    Thread.sleep(4000); // 4초를 기다린다
                    threadMessage(importantInfo[i]); // String을 하나씩 출력 한다
                }
            } catch (InterruptedException e) { // 이 쓰레드가 interrupted 되어 sleep 메소드가 InterruptedException을 던지면
                threadMessage("I wasn't done!"); // 메세지 출력
            }
        }
    }

    public static void main(String args[]) throws InterruptedException {
        long patienceInMS = 1000 * 60 * 60; // MessageLoop 쓰레드를 interrupt하기 전 기다릴 시간, 기본값: 1시간

        if (args.length > 0) { // 커맨드 라인을 통해 시간을 지정해주면, patienceInMS 값을 변경한다
            try {
                patienceInMS = Long.parseLong(args[0]) * 1000;
            } catch (NumberFormatException e) {
                System.err.println("Argument must be an integer.");
                System.exit(1);
            }
        }

        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeInMS(); // 현재 시간
        Thread t = new Thread(new MessageLoop()); // 쓰레드 instantiation
        t.start(); // 쓰레드 실행

        threadMessage("Waiting for MessageLoop thread to finish");
        while (t.isAlive()) { // MessageLoop 쓰레드가 살아있는 동안 
            threadMessage("Still waiting...");
            t.join(1000); // MessageLoop 쓰레드가 끝나기를 1초간 기다린다
            // 만약 patienceInMS 보다 긴 시간동안 실행되었는데도 아직 MessageLoop가 종료되지 않았다면
            if (((System.currentTimeInMS() - startTime) > patienceInMS) && t.isAlive()) { 
                threadMessage("Tired of waiting!");
                t.interrupt();  // MessageLoop를 interrupt 한다
            }
        }
        threadMessage("Finally!");
    }
}
```

Comment에 써 놓은 대로 만약 주어진 patienceInMS보다 긴 시간동안 MessageLoop가 실행되고 있다면, interrupt를 통해 MessageLoop가 실행되고 있는 thread가 InterruptedException을 던져서 `I wasn't done!` 이라는 메세지와 함께 thread를 종료시킨다. thread가 종료되면 t.isAlive()가 false가 되고 `Finally!` 가 출력되며 프로그램이 종료된다.


## ⒍ Reference

[Oracle: Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/index.html)
[Java: Difference in usage between Thread.interrupted() and Thread.isInterrupted()?](https://stackoverflow.com/a/62438836)