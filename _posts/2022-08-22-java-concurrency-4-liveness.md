---
title: Java's Concurrency - 4 (Liveness)
author: Yongju Kwon
date: 2022-08-22 21:03:00 -0700
categories: [Programming, Java]
tags: [Java, Thread]
---

이번 포스팅에서는 저번에 글이 길어져 포함하지 않은 Liveness에 대해서 알아볼 것이다.

## ⒈ Liveness?

동시에 실행되는 어플리케이션을 시간적으로 적절하게 실행시키는 것을 `Liveness`라고 한다. 한국말로 해석한 것을 찾아보니 활동성이라고 한다. 아래에서 가장 흔한 liveness 문제인 `deadlock`에 대해 알아보고 다른 두 가지 문제인 `starvation`과 `livelock`도 간단히 알아보도록 
하자.

## ⒉ Deadlock

`Deadlock`은 두 개 혹은 그 이상의 쓰레드가 서로를 기다리며 영원히 blocked 된 상태다. 예를 들어, 친구인 A와 B가 있고 이 둘 사이에는 꼭 따라야만 하는 한 가지 규칙이 있다고 가정해보자. 이 규칙은 한 사람이 다른 사람에게 고개를 숙여 인사하면 인사를 받은 사람이 인사할 때까지 처음 인사를 한 사람은 계속 고개를 숙이고 있어야 한다. 이 규칙의 맹점은 무엇일까?  
바로 두 사람이 동시에 인사를 하는 경우를 생각하지 않았다. 그렇다면 두 사람이 동시에 인사를 하면 어떻게 될까? 아래 코드를 통해 알아보자.

```java
public class Deadlock {

  static class Friend {
    private final String name;

    public Friend(String name) {
      this.name = name;
    }

    public String getName() {
      return this.name;
    }

    public synchronized void bow(Friend bower) {
      System.out.printf("%s: %s has bowed to me!\n", this.name, bower.getName());
      bower.bowBack(this);
    }

    public synchronized void bowBack(Friend bower) {
      System.out.printf("%s: %s has bowed back to me!\n", this.name, bower.getName());
    }
  }

  public static void main(String[] args) {
    final Friend a = new Friend("A");
    final Friend b = new Friend("B");

    // b가 a에게 인사한다, 인사 후 a는 b에게 다시 인사 해야 한다. 하지만 밑 쓰레드가 시작되면서 b의 lock을 own한 후 release하지 않으면 b.bowBack(a)가 실행될 수 없다.
    new Thread(() -> a.bow(b)).start();
    // a가 b에게 인사한다, 인사 후 b는 a에게 다시 인사 해야 한다. 하지만 위 쓰레드가 끝나지 않았기 때문에(a의 lock을 release하지 않았기 떄문에) a.bowBack(b)가 실행될 수 없다.
    new Thread(() -> b.bow(a)).start();
  }
}
```

IntelliJ의 debugger를 통해 이 두 쓰레드의 상태를 보면 아래와 같다. Thread-0@780은 Thread-1@783이 lock을 release하기를 기다리고 있고, 똑같이 반대로 Thread-1@783은 Thread-0@780이 lock을 release하기를 기다리고 있다.  
이 상태를 우리는 deadlock이라고 부른다.

![Deadlock](/assets/img/20220822/%08deadlock.png)
_Deadlock_

## ⒊ Starvation and Livelock

Starvation과 livelock은 deadlock보다는 덜 흔한 문제이지만 여전히 개발자라면 마주칠 수 있는 문제들이다.

⑴ `Starvation`
 Starvation은 쓰레드가 다른 쓰레드들과 공유하는 자원들에 접근할 수 없어서 더 이상 진행할 수 없는 상태이다. 보통 'greedy' threads에 의해 일어나는 상황이다.  
하나의 예로, 어떤 오브젝트가 synchronized method를 제공하는데 내부의 코드가 시간이 오래 걸리는 작업이라고 가정해보자. 그런데 어떤 한 쓰레드가 이 메소드를 자주 불러서 다른 쓰레드들이 접근할 때마다 접근이 불가능할 수 있는데 이 상황을 starvation이라고 부른다.

⑵ `Livelock`
 Livelock은 두 쓰레드가 서로에게 의존할 떄 생기는 문제인데, deadlock과는 달리 쓰레드들이 blocked 상태가 되지는 않는다. 아래 
코드 예시를 통해서 알아보자.
경찰과 범인 그리고 납치 된 사람이 있다. 범인은 경찰이 돈을 주면 납치된 사람을 풀어주기로 했고, 경찰은 돈을 주려고 한다. 아래 코드를 실행시키면 어떤 상황이 벌어질까?

```java
/**
 * @author www.codejava.net (All code below)
 */
public class Criminal { // 납치범
    private boolean hostageReleased = false;
 
    public void releaseHostage(Police police) {
        while (!police.isMoneySent()) {
 
            System.out.println("Criminal: waiting police to give ransom"); 
 
            try {
                Thread.sleep(1000); // 경찰로부터 돈을 기다리는중
            } catch (InterruptedException ex) {
                ex.printStackTrace();
            }
        }
 
        System.out.println("Criminal: released hostage");
 
        this.hostageReleased = true;
    }
 
    public boolean isHostageReleased() {
        return this.hostageReleased;
    }
}

public class Police {
    private boolean moneySent = false;
 
    public void giveRansom(Criminal criminal) {
 
        while (!criminal.isHostageReleased()) {
 
            System.out.println("Police: waiting criminal to release hostage");
 
            try {
                Thread.sleep(1000);
            } catch (InterruptedException ex) {
                ex.printStackTrace();
            }
        }
 
        System.out.println("Police: sent money");
 
        this.moneySent = true;
    }
 
    public boolean isMoneySent() {
        return this.moneySent;
    }
}

public class HostageRescueLivelock {
    static final Police police = new Police();
    static final Criminal criminal = new Criminal();
 
    public static void main(String[] args) {
        Thread t1 = new Thread(() ->police.giveRansom(criminal)).start();        
        Thread t2 = new Thread(() -> criminal.releaseHostage(police)).start();
    }
}
```

서로의 상태를 hold하고 있고 업데이트를 하지 못 하기 때문에 결국에는 아래와 같이 두 쓰레드 모두 계속 같은 statement만 프린트 할 것이다. 실생활의 예로 들어보자면 좁은 복도에서 서로 다 방향에서 오고 있던 두 사람이 길을 비켜주기 위해 방향을 바꾸는데 서로 계속 같이 바꾸는 바람에 둘 다 지나갈 수 없게 되는 상태에 비유할 수 있다.

![Livelock](/assets/img/20220822/livelock.png)
_Livelock_

## ⒋ Retrospective

Oracle의 공식 documentation을 읽으면서 Concurrency에 대해서 정리해봐야지 하면서 시작한 시리즈 였는데 4개까지 쓸 줄은 몰랐다. 이 뒤로도 같은 카테고리의 내용이 있는데 지금까지의 내용보다 High-level의 Concurrency를 다루는 내용이다. 내용이 많지는 않아서 두 번 정도 포스팅을 더 하면 마무리할 수 있을 것 같다. 블로그 스터디는 마지막 주 이지만 계속 써 나가서 더 깊이 알고 있는 자바 개발자가 되야지.!

## ⒌ Reference

[Oracle Java Documentation](https://docs.oracle.com/javase/tutorial/essential/concurrency/liveness.html)\
[Understanding Deadlock, Livelock and Starvation with Code Examples in Java](https://www.codejava.net/java-core/concurrency/understanding-deadlock-livelock-and-starvation-with-code-examples-in-java)
