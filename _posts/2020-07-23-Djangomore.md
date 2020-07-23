---
title: "Djangomore"
date: 2020-07-23
categories: Python Django
---

스켈레톤을 만들었으니, 더 많은 준비를 해야한다.

## Backend

우선 깃헙에 해당 파일을 올렸다. 깃헙 사용법은 익숙하니 따로 다루지 않는다.

그리고 기존에 쓰던 aws서버에서 ade파일을 clone 한 후
가상환경을 설치했다.

그 외에 몇가지 설정들을 추가해야했다. 우선 AWS 계정의 보안 서버에서 local host 8000을 허용해줘야하고,
장고의 settings 부분에 내 서버의 ip를 추가해주어야 했다. 

이후 python3 manage.py runserver 0:8000 을 수행하니 로컬서버 8000에서 코드가 정상작동됐다. 
그리고 <서버아이디:8000> 을 수행하니 폰, 컴퓨터 모두에서 정상작동 됐다.

하지만 이미 도메인이 있고, 이를 도메인에 얹어야 프로그램을 돌리는 맛이 날 것 같았다.

그래서 settings 부분에 adedata.com, www.adedata.com도 추가했다.
AWS의 Route53에 adedata.com 서버를 등록하고, 4가지 임시 도메인을 받았다. 그 외에 www.adedata.com도 등록했다.
그리고 구글 도메인 DNS에 4개의 임시 도메인을 추가하고, 현재 48시간을 기다리는 중이다. 

그리고 aws 서버에서 disown-h를 통해 서버를 써도 프로그램이 실행되게 만들었다. 우선 서버에 올리는 가장 기본적인 백엔드는 작업이 끝났다.

## React

언제까지 장고로 js를 만들고, 꾸미고, 작업을 할 수는 없다. UI개선을 위해 React.js를 써야한다 (jQuery한두개를 쓰긴 했지만 리액트가 프론트에는 좋다고 한다)
React를 공부하며 배운 바는 아래와 같다

1. Django와 함께 쓰이고, 앱 단위로 UI를 꾸미는 것 같다.
2. React는 component (함수와 비슷한것) 위주로 구성된다. 인풋이 정해져있고, 아웃풋 형태가 html과 유사하다.
3. Declarative라고 하는데, 작동 원리를 설명하지 않아도 알아서 작동된다는 뜻인 것 같다. 
  이게 어떻가 가능한지 모르겠다. 그러면 model, views, html, url들 없이 코드를 만들 수 있는건가?
  
  아래 HTML 코드가
```HTML
<div className="shopping-list">
  <h1>Shopping List for {props.name}</h1>
  <ul>
    <li>Instagram</li>
    <li>WhatsApp</li>
    <li>Oculus</li>
  </ul>
</div>;
```
아래 형태만으로도 구현이 가능하다.
```python
React.createElement("div", {
  className: "shopping-list"
}, React.createElement("h1", null, "Shopping List for ", props.name),
React.createElement("ul", null, 
React.createElement("li", null, "Instagram"), 
React.createElement("li", null, "WhatsApp"), 
React.createElement("li", null, "Oculus")));
```
사실 이게 더 편리한건지 모르겠다...?


아래 코드를 보면 이해가 조금 더 잘된다

9개의 3x3상자를 그리려고 한다면, react에서는 html을 아래처럼 할 수 있다
```
<div className="board-row">
  {this.renderSquare(0)}
  {this.renderSquare(1)}
  {this.renderSquare(2)}
</div>


board-row가 원래 장고 코드라면 따로 style.css에 저장이 되어있어야한다. 하지만 여긴 그런게 없어도 비슷한 형식으로 만들어주나보다
this.renderSquare()에서, renderSquare는 

class Board extends React.Component {
  renderSquare(i) {
    return <Square value={i} />;
  }
  
  
