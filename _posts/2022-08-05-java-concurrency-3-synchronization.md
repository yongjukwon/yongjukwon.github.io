---
title: Java's Concurrency - 3 (Synchronization)
author: Yongju Kwon
date: 2022-08-05 23:07:00 -0700
categories: [Programming, Java]
tags: [Java, Thread]
---

## ⒈ What is Synchronization?

여러 쓰레드들은 오브젝트나 필드같은 resource들을 서로 공유한다. 이는 두 가지 문제를 일으킬 수 있는데, ¹ 쓰레드들끼리의 충돌과 ² 메모리의 inconsistency이다. `Synchronization`은 이 문제들을 해결할 수 있는 방법이다. Synchronization은 `thread contention` 을 제공함으로써 이루어지는데 이는 두 개 이상의 쓰레드가 같은 resource에 동시에 접근하려고 시도할 때 하나 혹은 그 이상의 쓰레드를 상대적으로 느리게 실행시키거나 혹은 실행을 늦추는 방법이다.

이 포스트에서는 두 가지 문제점이 어떻게 일어날 수 있는지 먼저 알아보고, 해결 방법까지 알아보도록 하자.

## ⒉ Thead interference

아래와 같이 간단한 Counter 클래스가 있다. Variable c의 값을 변경하는 두 메소드(increment, decrement)와 accessor(value)가 존재한다. increment가 불려지면 c에 1이 더해지고, decrement가 불려지면 c에서 1이 빼진다. 하지만 여러 쓰레드가 동시에 이 Counter 오브젝트를 reference 한다면 이런 당연한 계산들이 원하는대로 실행되지 않을 수 있다.

```java
class Counter {
    private int c = 0;

    public void increment() {
        c++;
    }

    public void decrement() {
        c--;
    }

    public int value() {
        return c;
    }
}
```

`Thread interference`는 두 operation이 같은 data를 가지고 각각 다른 쓰레드에서 실행될 때 일어난다. 아래 예에서 c++; 혹은 c--;가 간단한 하나의 expression으로 보일 수 있지만 JVM에서는 이 간단한 expression도 여러 과정으로 translate 될 수 있다. c++를 예로 들면 다음과 같은 과정으로 나뉘어질 수 있다.

1. 현재 c 의 값을 가져온다
2. 불러온 값에 1을 더한다
3. 더해진 값을 c에 다시 저장한다.

c--도 두번째 과정에서 값을 더하는 것 대신 빼는 과정을 넣는다면 똑같이 표현할 수 있다. 자, 이제 thread interference가 일어나는 과정을 한 번 적어보자. Thread A는 increment를 부르고, Thread B는 decrement를 부른다고 가정하자.

1. Thread A: 현재 c 의 값을 가져온다
2. Thread B: 현재 c 의 값을 가져온다
3. Thread A: 불러온 c의 값에 1을 더한다; c = 1
4. Thread B: 불러온 c의 값에서 1을 뺀다; c = -1
5. Thread A: 더해진 값을 c에 저장한다; c = 1
6. Thread B: 뺀 값을 c에 저장한다; c = -1

Thread A의 값은 B에 의해 overwrite 된다. 이 시나리오는 여러 가지 일어날 수 있는 시나리오들 중 하나일 뿐이다. 상황에 따라서 B의 값이 A의 값으로 overwrite 될 수도 있고, 혹은 아무 에러가 없이 실행 될 수도 있다. 이처럼 결과를 예상할 수 없기 때문에 thread interference에 의해 일어나는 버그는 찾기도 힘들고 고치기도 힘들다.

## ⒊ Memory consistency errors

`Memory consistency errors`는 한 쓰레드에서 실행된 코드가 다른 모든 쓰레드들에게 보여지지 않을 때 일어나는 에러이다. 예를 들면, 아래 코드와 같이 counter variable을 thread A가 와 thread B가 ++ operator로 값을 1씩 증가시켰을 때 결과값이 00, 01, 10, 혹은 11이 나올 수 있다는 뜻이다. (예시일뿐!)

```java
  int counter = 0;

  // thread A
  counter++;

  // thread B
  counter++;

  // thread A
  System.out.print(counter);

  // thread B
  System.out.print(counter);

  // Result could be 00, 01, 10, or 11
```

Memory consistency error는 엄밀히 말하면 Java가 아니고 CPU에 의해 일어나는 에러이다. 여기서 자세히 설명하는 것은 무리가 있고, 간단하게 설명하고 넘어가도록 하겠다. 

CPU가 어떤 값을 쓰는(write) 작업을 할 때에는 다른 작업들은 실행될 수 없다. 그래서 어떤 값이 쓰여질 때에는 다른 쓰레드들이 그 값을 가져올 수 없기 떄문에 쓰레드들은 수시로 그 값을 가져갈 수 있는지 확인을 해야 한다. 값을 가져갈 수 있는 상황이 되면 값을 읽게 되는데 이 순서가 값이 쓰여진대로(write) 혹은 프로그래머가 실행한 순서대로 가져오는 게 아니기 때문에 이 경우에 memory consistency error가 일어날 수 있다.

개발자에게 중요한 것은 memory consistency error를 어떻게 방지할 수 있는가이다. Java documentation 에서는 `happens-before` relationship을 만들라고 하는데, 말 그대로 어떤 작업 전에 모든 과정이 일어나게끔해서 그 결과가 다른 쓰레드들에게 visible 하게끔 코드를 작성하라는 것이다. 마치 수학시간에 배운 필요조건, 충분조건처럼 어떤 메소드나 클래스를 happens-before relationship을 가지게끔 만들면 memory consistency error를 방지 할 수 있다는 말이다. 

우리가 지난 두 포스팅 중에 이미 이 관계를 의도하지 않고 이 관계를 만들어 사용했는데, `Thread.start` 와 `Thread.join`이 그것들이다. 간단하게 설명할텐데 그 전에 왜 이것들이 그런 관계를 만들 수 있는지 한 번 생각해보고 읽도록 하자.
- 어떤 쓰레드에서 Thread.start를 사용해 쓰레드를 실행했다면 그 전에 있는 코드는 실행된 쓰레드에게 모두 visible하다
- 성공적으로 Thread.join을 실행되었다면 join된 쓰레드 안에서 실행된 모든 코드는 Thread.join을 실행한 쓰레드에서 모두 visible하다

이 외에도 더 여러가지 happens-before relationship 을 만들어 주는 여러 시나리오가 있는데, 궁금하다면 [Memory Consistency Properties](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html#MemoryVisibility) 혹은 [17.4.5. Happens-before Order](https://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html#jls-17.4.5) 를 참고해보자.

## ⒋ Synchronized methods

자바는 두 가지 기본 synchronization 방법을 제공하는데, ¹`synchronized methods` 와 ²`synchronized statements`이다. 이번 섹션에서는 synchronized methods에 대해 알아보고 다음 섹션([5.Implicit Locks and Synchronization](#-intrinsic-locks-and-synchronization))에서 synchronized statements에 대해 알아볼 것이다.

메소드를 synchronized 로 만들기 위해서는 `synchronized`라는 키워드를 access modifier 뒤에 붙여주면 된다.

e.g)
```java
public class SynchronizedCounter {
    private int c = 0;

    public synchronized void increment() {
        c++;
    }

    public synchronized void decrement() {
        c--;
    }

    public synchronized int value() {
        return c;
    }
}
```

메소드를 synchronized로 만들어 주면 그렇지 않을 때와 비교해 두 가지가 달라진다.
1. 다른 쓰레드들이 메소드를 동시에 부를 수 없게 된다. 한 쓰레드가 synchronized method를 부르면, 같은 메소드를 실행한 다른 쓰레드들은 실행되고 있는 쓰레드가 끝나기 전까지 실행이 미뤄진다.
2. Synchronized method가 존재한다면 자동적으로 happens-before relationship 이 형성된다. 결론적으로 variable c에 대한 memory consistency error를 방지할 수 있다.

결과적으로 Synchronized methods는 앞서 살펴 본 두 가지 문제점 모두 해결할 수 있지만, `Liveness`와 관련된 문제점을 야기할 수 있는데, 다음 포스팅에서 다룰 예정이다.

## ⒌ Intrinsic Locks and Synchronization

 1) `Intrinsic Locks`

`Synchronization`은 `intrinsic lock` 혹은 `monitor lock`이라고 불리는 entity에 의해 이루어진다. 즉, 이 lock이 오브젝트의 상태한 access를 매니징하고 happens-before relationship을 만드는 데에 사용된다.

모든 오브젝트는 intrinsic lock을 가지고 있다. 어떤 오브젝트의 필드에 대한 권한(access)을 얻기 위해, 쓰레드는 그 오브젝트의 intrinsic lock을 먼저 얻어야 한다. 그리고 모든 작업이 끝나면 그 lock을 release한다. 그래서 우리는 쓰레드가 intrinsic lock을 얻은 시점부터 release할 때까지 그 lock을 소유한다(own)라고 표현한다. 한 쓰레드가 intrinsic lock을 소유하고 있는 한 다른 쓰레드들은 같은 lock을 얻을 수 없다. 즉, 다른 쓰레드들이 그 lock을 얻으려고 하면 모두 block된다.

그렇다면 static synchronized method가 실행되면 어떨까? Static method는 object가 아닌 class와 associated 되어 있기 때문에, class의 lock을 이용하게 된다. 즉, static synchronized method가 실행되고 있다면 해당 class에는 다른 쓰레드가 접근할 수 없다.

 2) `Synchronized Statements`

Synchronized 코드를 만드는 다른 방법은 `Synchronized statements`를 사용하는 것이다. Synchronized methods와의 가장 큰 차이점은 Synchronized statements은 오브젝트를 반드시 명시해줘야 하는데 이것이 lock의 scope이다.

아래의 예를 보자. MsLunch 클래스는 c1과 c2라는 long variables를 가지고 있고 이 두 variable은 함께 사용 되지 않는다고 가정해보자. 모든 필드는 synchronized 되어야 하지만 c1을 update할 때 c2의 값이 변경되는 것을 막을 필요는 전혀 없다. 만약 우리가 synchronized method를 사용한다면 쓰레드들은 한 번에 하나(c1 혹은 c2)의 값만 업데이트 할 수 있지만 아래처럼 각각 다른 object를 제공함으로써 scope를 줄여 서로의 업데이트가 방해되지 않도록 만들 수 있다.

```java
public class MsLunch {
    private long c1 = 0;
    private long c2 = 0;
    private Object lock1 = new Object();
    private Object lock2 = new Object();

    public void inc1() {
        synchronized(lock1) {
            c1++;
        }
    }

    public void inc2() {
        synchronized(lock2) {
            c2++;
        }
    }
}
```

3) `Reentrant Synchronization`

`Reentrant Synchronization`은 어떤 한 lock을 얻은 쓰레드는 반복해서 그 lock과 associated된 오브젝트나 메소드에 접근할 수 있는 것을 말한다. 예를 들어, recursive 메소드가 synchronized 되었다고 가정했을 때, 해당 lock을 얻은 쓰레드는 recursive 메소드가 끝이날 때까지 lock을 소유하게 된다.

## ⒍ What's next?

다음 포스팅에서는 volatile keyword와 Liveness, 멀티쓰레딩이 실행되는 어플리케이션이 순서대로 실행시킬수 있는 방법,에 대해 알아볼 것이다.


## ⒎ References

[Synchronization - Oracle](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html)\
[Java Synchronized Blocks - Jenkov.com](https://jenkov.com/tutorials/java-concurrency/synchronized.html#synchronized-blocks-instance-methods)\
[Synchronized Block in Java - Javatpoint.com](https://www.javatpoint.com/synchronized-block-example)