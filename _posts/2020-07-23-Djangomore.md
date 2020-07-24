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
React를 공부하며 배운 바는 아래와 같다.

주로 이 링크를 많이 참조했다 : https://reactjs.org/tutorial/tutorial.html

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
```python
<div className="board-row">
  {this.renderSquare(0)}
  {this.renderSquare(1)}
  {this.renderSquare(2)}
</div>
```

board-row가 원래 장고 코드라면 따로 style.css에 저장이 되어있어야한다. 하지만 여긴 그런게 없어도 비슷한 형식으로 만들어주나보다
this.renderSquare()에서, renderSquare는 아래처럼 만들어져있다. 인풋을 하나 받는다. 

```python
class Board extends React.Component {
  renderSquare(i) {
    return <Square value={i} />;
  }
```  
  
renderSquare함수는 또 Square를 받는다. 이건 또 뭐냐.

```
class Square extends React.Component {
  render() {
    return (
      <button className="square">
        {this.props.value}
      </button>
    );
  }
}
```
이런식이다. render()로 되면 저렇게 <Square ~~~ >입력을 받을 수 있는 것 같다. 
그리고 그 안에 html소스코드를 넣어주면 된다. 그러면 반복적인 html이 필요 없어진다.
이렇게 component들로 여러개를 만들어두면, 나중에 디자인만 여기저기 재조합하기 편하다. 
하나하나 html 바꾸고, <div> 몇십개를 레이어로 해놓고 해메는것보다 훨씬 편리하다.
  
여기에, js에서 할 수 있는 반응을 넣을 수 있다.

```
class Square extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: null,
    };
  }

  render() {
    return (
      <button
        className="square"
        onClick={() => this.setState({value: 'X'})}
      >
        {this.state.value}
      </button>
    );
  }
}
```
이런 코드를 보면, 현재 상태를 null로 지정하고 클릭 시 상태를 X로 변환할 수 있다.
그렇다면, 특정 조건에 어떤 반응을 넣고싶으면 어떻게 할까?
원래 장고라면 반응에 대한 계산을 views.py에서 하고, 그 결과 노출을 css에서 하지 않았을까 싶다.

React에선 아래와 같다.

```
handleClick(i) {
    const squares = this.state.squares.slice();
    squares[i] = 'X';
    this.setState({squares: squares});
  }
```
이러면 명령대로 null 이 X로 바뀐다.

이러한 작업을 거치면 최종 작업은 아래와 같다.

Board라는 클래스만 가져왔고, 또 다른 다양한 클래스들도 있다.
```
class Board extends React.Component {
  handleClick(i) {
    const squares = this.state.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
      squares: squares,
      xIsNext: !this.state.xIsNext,
    });
  }

  renderSquare(i) {
    return (
      <Square
        value={this.props.squares[i]}
        onClick={() => this.props.onClick(i)}
      />
    );
  }

  render() {
    const winner = calculateWinner(this.state.squares);
    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}
```
보면 클래스 내에 쓸 수 있는 툴이 있다. 결국 사용되는건 render()함수고, 여기서 계산을 하고 결과를 html형식으로 출력한다.
보면 출력되는 html은 네모 9개가 다이고, 이는 return 값에 그대로 있다.
그에 대한 연산을 renderSquare에서 하고, 이게 <Square>을 부르는데, 그 Square 안에 필요한 숫자를 인풋으로 넣어준다.
  이 부분 (component)에서 인풋 (interaction)도 받고, 그래서 onClick도 한다. 여기선 결과물 시각화와 출력만 한다.
그리고 다른 class들이 결과물을 뽑고, 우승자 출력/ 계산 등을 아마 메인에서 할 것이다. (이건 하나의 클래스일 뿐이다)
이렇게 여러 클래스를 가진 후, 각각의 앱?부분을 모아서 어떠한 상황엔 class를 출력하고, 우승자가 생기면 보이지 않을 수 있다.
일반적인 함수처럼 코드를 짤 수 있다.
그리고 여기의 class는 장고의 Model과 비슷하게 작용한다.

궁금한점은, 여기서 className="board-row" 등장하지 않는 클래스들도 사용된다. 이들이 아마 기본 디자인? js/ css? 같다
근데 프론트엔트라고 하기에, 결국 css는 누가 하는건지 모르겠다.
결론은, js가 interactive한건 알겠는데 views-html 의 render get 사용과 뭐가 다른지 잘 모르겠고 (뭐 팝업 뛰우는 등은 알겠다. 근데 여기서 그걸 쓰진 않는다)
css는 또다른 영역이란것이다.

## 장고 + 리액트

이제 핵심은, 이미 장고로 쓴 부분을 어떻게 리액트로 변환하는지이다.
기존에 쓴 html은 아마 100% 버릴 것이지만, 그래도 어느정도 얼추 사용은 가능하다.
문제는 views의 내용들과 url내용이다. 이를 다 버릴것인가? 어떻게 사용이 되는것인가?
그리고 models를 어느정도 사용을 해야한다. 그래야 DB에 저장이 가능하다. 그럼 models는 어떻게 사용할 것인가?
이에 대해 우선 해결책을 후가려고 아래 링크를 사용한다.
https://www.valentinog.com/blog/drf/

Django REST + React를 사용하는 형태를 알려주는데, 이를 참고해보자.

해당 링크는 세가지를 제시하는데, 두번째가 내가 생각한 방식이다.
model, view, url 등을 모두 장고에서 짜고, 해당 view를 import 해서 쓰는 형식이다.
JWT를 쓴다고 하는데 이게 뭔지는 모르지만, 이 떄문에 어렵고 복잡하다고 한다.

두번째 방식은, model, view, url모두 쓰긴 하는데 조금 다르다.
model은 그대로 가져가고, url -> view를 하는데 한줄만 있다. 여기서 있는 view함수 하나가 
```
def index(request):
    return render(request, 'frontend/index.html')
```
이런다. 그러면 해당 페이지의 모든 frontend 관련 함수들과 html들이 index.html에 저장되어있다.
그러면, index.html 내에 여러 함수가 있고, 위에 보인것처럼 prop()을 쓰고 그 함수를 쓰면 된다. class도 그 안에서 만들어버린다.

아래 링크는 그 어렵다는 방식을 쓴다.
http://milooy.github.io/TIL/Django/react-with-django-rest-framework.html#react-redux-react-router

보면 모델에
```
class Moim(models.Model):
    author = models.ForeignKey('auth.User')
    title = models.CharField(max_length=200)
    text = models.TextField()
    date = models.DateTimeField()
    ...
```
을 만들고 

url을 아래처럼 만들고
```
urlpatterns = [
    url(r'^moim/$', MoimListView.as_view(), name='moim'),
]
```

views.py에 아래처럼 만든다.
```
class MoimListSerializer(serializers.ModelSerializer):

    class Meta:
        model = Moim
        fields = ('id', 'author', 'title', 'text', 'created_date')


# api/moim 으로 get하면 이 listview로 연결
class MoimListView(generics.ListAPIView):
    queryset = Moim.objects.all()
    serializer_class = MoimListSerializer

    def list(self, request):
        queryset = self.get_queryset()
        serializer_class = self.get_serializer_class()
        serializer = serializer_class(queryset, many=True)

        page = self.paginate_queryset(queryset)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)

        return Response(serializer.data)
```
Serializer가 무엇을 하는지 모르고, API가 헷갈리지만... 특히, generic을 써서 폼이 더 엉망이지만 view가 비어있진 않다. 
  비교용으로, 위에서는 

```def index(request):
    return render(request, 'frontend/index.html') 
```
이 한줄로 모든게 index.html로 넘어갔다.

그리고 view에 사용된 함수를 사용하는데, 

```
import axios from 'axios'
export const FETCH_MOIM = 'FETCH_MOIM';

export function fetchMoim() {
  const request = axios.get('/api/moim/');
  return {
    type: FETCH_MOIM,
    payload: request
  }
}
```
라는 actions/index.js를 가져온 후
containers/MoimList.js에서
```
import { fetchMoim } from '../actions/index';

class MoimList extends Component {
  componentDidMount() {
    this.props.fetchMoim();
  }

  renderMoim () {
    return this.props.moimList.map((moim) => {
      return <li key={moim.id}><Moim moimData={moim}/></li>;
    });
  }
  ....
```
라는 형식으로 사용한다. 
import fetchMoim from actions/index를 가져오는데, 여기있는 fetchMoim이란 함수가 axios.get('/api/moim')을 통해 views의 함수를 불러낸다.

실제로 매우 복잡한 구조다. 애초에 containers/MoimList.js - actions/index.js - views.py를 왔다갔다 하는건데,
이를 html에 모두 때려박는것으로 대체가 가능하다.

그러면 form은 또 어디에 들어가는가?
Django-REST-API는 뭐냐...

## 결론.

새로 만든다. front를 전부 react로 바꿔본다.
우선 오늘 탬플릿을 만들고, 첫페이지만 구상해보자 - profile list 페이지 정도만... 그러고나면 문제가 보이겠지.
