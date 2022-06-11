---
title: Lazy Loading
author: Yongju Kwon
date: 2022-06-11 01:28:00 -0700
categories: [Programming, Java]
tags: [Java, Database, Web Development, REST API, Spring]
---

## 1. Background

회사에서 DB에 Blob(Binary Large Object)을 업로드하는 REST API를 만들게 되었다. 현재는 이 오브젝트만 ORM에서 빠져 있는 상태이다. 그 이유는 만약 이 오브젝트가 업로드 될 column을 다른 column들과 같은 방법으로 mapping을 하면 테이블이 읽혀질 때마다 함께 읽혀지기 때문에 Performance가 감소할 수 있기 때문이다. 현재 돌아가는 서비스가 주기적으로 이 오브젝트가 속한 테이블을 읽고 있고, 서비스가 이 오브젝트를 성공적으로 한 번만 읽으면 되기 때문에 Lazy Loading을 possible option으로 생각하고 디자인중이다. 회사에서 사용하게 될 환경은 Java Spring이지만 검색 중 Lazy Loading이 프론트엔드에서도 쓰인다는 걸 알게 됐고 프론트엔드, 백엔드 개발자 모두 이해할 수 있도록 글을 써보려고 한다.

## 2. What is Lazy Loading & Eager Loading?

`Lazy Loading`은 다른 말로 `Asynchronous Loading`이라고도 불리며, 컴퓨터 프로그래밍에서 사용되는 디자인 패턴중의 한가지이다. 주로 웹 서비스 개발 중 오브젝트의 initialization을 필요할 때까지 늦추는 것을 의미한다. 간단한 예로는, 어떤 한 웹 페이지의 밑(스크롤을 내려야만 볼 수 있는 곳)에 존재하는 한 이미지가 우리가 스크롤을 내리기 전까지는 해당 이미지 오브젝트 대신 placeholder가 존재하다가 스크롤을 내렸을 때에서야 그 이미지를 로딩하는 것이 있다.

Lazy Loading의 반대로는 `Eager Loading`이 있다. Eager Loading은 `Pre-loading`이라고도 불리며 프로그램이 실행되면(코드가 execute되면) 모든 resources를 모두 불러온다. 우리가 이 블로그를 열었을 때 이 블로그 안의 모든 글, 이미지, 동영상 등이 한 번에 불러오는 것을 예로 들 수 있다.

## 3. Why do we use Lazy Loading?

Lazy Loading을 쓰는 이유로는 단연 performance enhancement가 꼽히겠다. 조금 더 자세히 설명하면,

  1. Reduced initial load time/memory\
    모든 resource가 아닌 일부분만 처음에 불러오기 때문에 load time이 줄어들고 불러오지 않은 resources 덕분에 사용할 메모리도 적어진다

  2. Faster connection to users; better user experience\
    줄어드는 load time은 유저들에게 더 빠른 contents 제공을 보장한다

  3. Avoiding unnecessary code execution\
    같은 맥락으로 Lazy loading으로 불려올 resources에 대한 코드들은 불필요한 실행을 피할 수 있다

  4. Cost-effective\
    time/memory 최적화로 불필요한 resources들을 불러오지 않음으로써 운영비용을 아낄 수 있다

단점도 존재하는데,

  1. Extra codes\
    Lazy Loading을 위해서 작성해야 하는 코드가 당연히 필요하고, 때로는 생각 외로 복잡해질 수 있다.(편하게 사용할 수 있도록 점점 많은 방법들이 개발되고 있다)

  2. Low web rankings\
    Initial load 되지 않는 페이지들이 존재한다면 Search Engine Optimization(SEO) 결과에 영향을 미칠 수 있다.

## 4. Where is Lazy Loading used?

이 글을 작성하려고 검색 해 보기 전까지는 프론트엔드에서 Lazy Loading이 사용 된다는 건 생각을 해보지 않았는데 너무나 당연한 사실이었다. 편협한 시야를 언제쯤 넓힐 수 있을까😢\
프론트엔드와 백엔드 한 가지씩 예를 들어서 Lazy Loading이 어떻게 사용 될 수 있는 지 보려고 한다.

### Front-end

```react
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react -router-dom';

// lazy-loading syntax in ReactJS
const About = lazy(() => import('./About'));

const App = () => (
  // ... 
    <Route
      path="/"
      exact
      render={() => (
        <div>
          <h1>This is the main page</h1>
          <a href="/about">Click here</a>
        </div>
      )}
    />
  // ...
);

export default App;
```
{: file='App.jsx'}
```react
import React from 'react';

const About = () => {
   return (
      <div>
         <h1>This is the about section</h1>
      </div>
   );
};
export default About;
```
{: file='About.js'}

- Output

![Output](/assets/img/20220610/Lazy.jpg)
_About 페이지는 Click here 링크를 클릭해야만 render 된다_

### Back-end

```java
public class Item {
  ...

  @JsonIgnore
  @OneToMany(cascade = CascadeType.ALL, mappedBy = "item")
  private List<PurchaseItem> purchaseItems = new ArrayList<>();

  ...
}
```
{: file='Item.java'}
```java
public class PurchaseItem {
  ...

  @JoinColumn(name = "item_id")
  // lazy-loading syntax in Java Spring
  @ManyToOne(fetch = LAZY, cascade = CascadeType.PERSIST)
  private Item item;

  ...
}
```
{: file='PurchaseItem.java'}

작년에 취업 준비하면서 만들었던 토이 프로젝트에서 마침 lazy loading을 썼던 예가 있어서 가지고 왔다.

Inventory Management System을 만들었는데 Item은 많은 PurchaseItem을 List로 가질 수 있고, 이 PurchaseItem들은 영수증에 보여지게 된다. 유저가 구매할 때는 당연히 필요가 없고 구매 후에만 보여지면 되기 때문에 lazy loading을 사용했다. 만약 lazy loading을 하지 않는다면 시스템을 켜는 순간 Item들과 함께 모든 고객들이 구매했던 Item들이 필요 하지도 않은데 모두 load 될 것 이고 당연히 time/memory waste가 발생한다.

작은 프로젝트 였지만, 공부를 목적으로 만들었기 때문에 굉장히 큰 시스템이란 가정 하에 performance를 높이기 위해 사용해 보았다.

## 5. Conclusion

Lazy Loading의 단점을 생각해보지는 않았는데, 이번 REST API를 개발 하는 데에 있어서는 큰 걸림돌이 될 것 같지는 않다. 꼭 필요한 코드이기 때문에 첫번째 단점(extra code)은 해당이 안되고, 두번째 단점(SEO results)도 해당이 되지 않는다. 다음주에 앱 디자인 리뷰 미팅이 있는데 이 글을 바탕으로 설명하며 또 다른 pitfalls이 있는지 피드백도 받아보고 새로운 점이 있으면 업데이트 해야지.

## 6. References
[Lazy Loading - Wikipedia](https://en.wikipedia.org/wiki/Lazy_loading)\
[What is Lazy Loading - GeeksforGeeks](https://www.geeksforgeeks.org/what-is-lazy-loading/)\
[Lazy Loading in ReactJS - TutorialsPoint](https://www.tutorialspoint.com/lazy-loading-in-reactjs)\
[How to Set Up Lazy Loading for Optimal SEO Results](https://www.webfx.com/seo/tech/lazy-loading-seo/)
