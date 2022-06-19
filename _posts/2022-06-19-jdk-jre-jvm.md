---
title: JDK, JRE and JVM
author: Yongju Kwon
date: 2022-06-19 16:23:00 -0700
categories: [Programming, Java]
tags: [Java, JDK, JRE, JVM]
---

## ⒈ Background

저번 블로그 포스팅 스터디 중 compiler와 interpreter 이야기가 나왔다. 자바가 어떻게 class 파일로 컴파일이 되는지, 그리고 실행 되는지 이야기 해보다가 예전에 같은 궁금증을 가지고 찾아본 내용을 바탕으로 글을 작성해본다.

## ⒉ What is JVM?

Java Virtual Machine(JVM)은 **Java byte code를 machine code로 translate하여 실행 될 수 있는 환경을 제공**한다. Machine code는 프로세서마다 다르기 때문에, 운영체제마다 혹은 같은 운영체제지만 다른 아키텍쳐에서는 각기 다른 JVM을 설치해줘야 한다.

JVM은 자바의 슬로건인 Write once, run anywhere(WORE)의 핵심 요소인데 machine interface를 제공함으로써 자바 어플리케이션을(구체적으로는 .class file) 어떤 환경에서도 실행할 수 있도록 한다. 그러한 이유로 우리는 Java를 platform-independent하다고 한다. 

![how-java-compiles](/assets/img/20220618/how-java-compiles.png)_tutorial.eyehunts.com_

결국 우리는 코드를 쓰고 있는 운영체제에서 실행 되는 코드를 쓰고 있는 게 아니라, JVM에서 변환 될 코드를 쓰고 있다고 생각하면 더 직관적으로 이해할 수 있다.

JVM만을 따로 설치할 수는 없으며 JRE를 설치하면 함께 설치된다.\
JVM에 대해 더 자세한 내용을 알고 싶다면👉🏻[How does JVM work(In progress)](/posts/how-does-jvm-work)


## ⒊ What is JRE?

Java Runtime Environment(JRE)는 **JVM을 포함해서 자바 어플리케이션을 실행시키기 위해 필요한 각종 라이브러리나 파일들을 포함**하고 있다. 

개발을 하기 위해서가 아니고 자바 어플리케이션을 실행만 하기 위해서라면 JRE만 설치하면 된다.

## ⒋ What is JDK?

Java Development Kit(JDK)는 **자바 어플리케이션을 개발하기 위해 필요한 패키지**이다. JDK는 JRE와 함께 개발에 필요한 툴(컴파일러, JavaDoc, 디버거 등)을 포함하고 있다.

JRE와 JDK에 포함되는 라이브러리나 파일들은 이 그림을 참고하자👉🏻[JDK, JRE & JVM (Complex version)](#-conclusion)

## ⒌ Conclusion

따라서 JVM, JRE 그리고 JDK의 관계는 다음과 같다.

![JDK_JRE_JVM_simple](/assets/img/20220618/jdk-jre-jvm-simple.png)
_JDK, JRE & JVM(geeksforgeeks.org)_

그리고 더욱 깊게 알고 싶은 분들을 위해..🫠
![JDK_JRE_JVM_complex](/assets/img/20220618/jdk-jre-jvm-complex.png)
_JDK, JRE & JVM(oracle.com)_

## ⒍ References

[Oracle](https://oracle.com)\
[Difference between JDK, JRE and JVM](https://www.geeksforgeeks.org/differences-jdk-jre-jvm/)