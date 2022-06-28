---
title: How does JVM work
author: Yongju Kwon
date: 2022-06-24 23:26:00 -0700
categories: [Programming, Java]
tags: [Java, JVM, Architecture]
---

이 글을 먼저 읽고 오는 걸 추천드립니다: [JVM, JRE and JDK](/posts/jvm-jre-jdk/)

## ⒈ What is Java Virtual Machine(JVM)

Java Virtual Machine(JVM)의 주된 목적은 Java 컴파일러를 통해 변환된 Byte code(.class 파일)를 Machine code로 변환해주는 것이다. 이 글에서는 JVM의 아키텍쳐를 함께 보면서 어떻게 변환 과정이 이루어지는 지 알아볼 것이다.

JVM의 아키텍쳐는 아래 그림과 같고, 세 가지 큰 요소로 이루어져 있다. 
  1. Class Loader Subsystem
  2. Runtime Data Area
  3. Execution Engine

![jvm_architecture_simple](/assets/img/20110624/jvm_architecture.jpeg)
_JVM Architecture simple version(careerbless.com)_

3가지 요소를 하나씩 자세히 알아보며 어떤 sub components들로 구성이 되어 있고 어떻게 동작하는 지 알아보자.

## ⒉ JVM's 3 components

### ２-⒈ Class Loader Subsystem
  
  ![class_loader_subsystem](/assets/img/20110624/class_loader_subsystem.png){:width="50%"}

  `Class Loader Subsystem` 안에서 세 가지 작업이 프로세싱 되는데, Loading, Linking 그리고 Initialization이다.

  1) `Loading`: 작성한 코드 내에 정의된 클래스들이 불려진다

  2) `Linking`: Verify, Prepare와 Resolve 라는 sub-processing이 이루어진다

  - `Verify`: JVM안의 Byte Code Verifier가 클래스 파일들이 제대로 포맷팅 되어 있는지, 구조적으로 정확한지, Valid한 컴파일러에 의해서 컴파일 되었는지 확인하는 역할을 한다. 만약 invalid 하다고 판단이 되면 java.lang.VerifyError를 던진다

  - `Prepare`: static variables을 위한 메모리를 할당하고 타입에 따라 default value로 initialize한다

  - `Resolve`: symbolic 메모리 레퍼런스들이 원래의 레퍼런스로 변환된다.

  3) `Initialization`: linking process(Prepare, specifically)에서 initialize된 static variables에 할당된 값을 제공하고, static block들이 실행된다.


### ２-⒉ Runtime Data Areas

  ![runtime_data_area](/assets/img/20110624/runtime_data_area.png){:width="80%"}

  `Runtime Data Areas`에는 총 5가지 요소가 존재한다.

  1) `Method Area`: Static variables를 포함한 모든 클래스 레벨 데이터가 이 곳에 저장된다. JVM안에 오직 한 Method Area만 존재 하기 때문에 모든 resources는 share된다.

  2) `Heap Area`: 모든 Objects, instance variables와 arrays가 이 곳에 저장된다. 역시 JVM안에 오직 한 Heap Area만 존재 하므로 모든 resources는 share된다.
  > <sup>Method Area와 Heap Area는 여러 threads들에 의해 share되는 메모리기 때문에 thread-safe하지 않다</sup>

  3) `Stack Area`: 한 thread 당 하나의 stack area가 만들어진다. 그 thread 안에서 불려지는 method들 하나당 하나의 stack이 쌓여 memory에 쌓이고 우리는 stack frame이라고 부른다. 모든 local variables가 저장 되고, 당연히 share되지 않는 resource이기 때문에 thread-safe하다.\
  Stack frame은 다시 세 가지 작은 요소들로 나누어지는데,

  - `Local Variable Array`: 말 그대로 local variable을 담는 array이다. Method의 모든 parameter와 local variable들을 담고, array의 한 slot당 4 Bytes가 할당된다. int, float과 reference는 1 slot을 차지하고 Byte, short와 char은 담기기 전에 int type으로 변환되기 떄문에 역시 1 slot을 차지한다. double과 long은 2 slots을 차지하고 Boolean은 JVM마다 다르지만 거의 모든 JVM이 1 slot을 할당한다.\
  (e.g.) 아래 코드 속 parameter들은 그 아래 그림처럼 local variable array에 담기게 된다.
  
    ```java
      class Example
      {
        public void bike(int i, long l, float f, 
                    double d, Object o, byte b)
        {
        } 
      }     

    ```
  
  ![local_variable_array](/assets/img/20110624/local_variable_array.jpeg){:width="50%"}
  _local_variable_array(geeksforgeeks.org)_

  - `Operand stack`: 하나의 workspace 또는 work area 라고 생각할 수 있고, calculation 중간의 결과들을 저장할 수 있는 장소이다\
  - `Frame data`: Constant Pool, 이전 스택 프레임에 대한 정보, 현재 method가 속한 클래스에 대한 reference 등의 정보를 갖는다

  4). `PC Registers`: 하나의 thread 당 하나의 PC register가 생성된다. PC register는 현재 실행되는 instruction의 address를 가지고 있다.
  >만약 현재 실행되는 method가 native라면 PC register의 값은 undefined이다

  5) `Native Method Stack`: Native method의 정보를 저장한다. 하나의 thread 당 하나의 native method stack이 생성된다.


### ２-⒊ Execution Engine

![execution_engine](/assets/img/20110624/execution_engine.png){:width="60%"}
_execution engine(geeksforgeeks.org)_

Execution Engine은 RDA에 변환되어 저장된 byte code를 마치 instruction 단위로 읽어서 마치 CPU처럼 하나씩 실행한다. Byte code command는 1-byte의 Opcode와 Operand로 이루어져 있는데, execution engine은 Opcode를 corresponding operand와 함께 실행시키고 다음 Opcode로 넘어간다. 이 과정은 execution engine의 두 요소에 의해 이루어 진다.

1) `Interpreter`: Interpreter는 byte code instruction을 machine code instruction으로 변환한다. 

2) `Just In Time(JIT) Compiler`: JIT Compiler의 목적은 performance 향상이다. JIT Complier안의 `Profiler`는 어떤 한 method가 계속 반복적으로 불리는지 감지하는 역할을 한다. 반복적으로 불리는 method를 `Hotspot` 이라고 부르며 Hotspot이 정해진 thread hold value(JVM마다 다름)에 도달하면 JIT Compiler가 그 Hotspot에 해당되는 native code를 생성한다. 그 뒤로는 interpreter가 매 번 native code로 변환하는 프로세스 대신 JIT compiler에 의해 생성된 native code가 사용함으로써 performance 향상을 이룬다.

<!-- 하나의 instruction안에는 -->

## ⒊ Retrospective

JVM architecture에 관련된 동영상도 많이 보고, 글도 많이 읽으면서 공부할 때는 너무 재미있고 사람들에게 알려줄 생각에 들떠 있었다. 글을 적기 시작하니 도대체 어디서부터 어디까지 설명을 해야할 지 감이 안잡혔다😭. 자세히까지는 알 필요가 없지만 CPU architecture, Stack, Heap, references 등에 대한 기본적인 지식을 가진 사람을 독자로 잡고 글을 써 보았다. 그렇지 않으면 끝까지 읽는 사람이 손에 꼽을 정도로 길고 지루한 글이 될 것 같았다(그렇게 자세한 글을 쓸 자신도 없다🫠). 그래도 아직 부족한 글이라 이미지나 예를 많이 넣어서 조금 더 쉽게 이해할 수 있는 글로 완성 시켜야겠다.

## ⒋ References

[How JVM Works - JVM Architecture?](https://www.geeksforgeeks.org/jvm-works-jvm-architecture/)\
[waytoeasylearn.com](https://www.waytoeasylearn.com/learn/)\
[The JVM Architecture Explained - Dzone](https://dzone.com/articles/jvm-architecture-explained)\
[JVM Architecture -tutorial, Youtube](https://www.youtube.com/watch?v=ZBJ0u9MaKtM)