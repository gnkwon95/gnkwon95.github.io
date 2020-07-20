---
title: "DjangoUpgrade"
date: 2020-07-18
categories: Python Django
---

## 1. 네비게이션 만들기

네비게이션은 대부분 웹사이트 상단에 있는 긴 창으로, 자유롭게 웹사이트를 이동하게 해준다.

### 1.1 base.html에 네이게이션 바 추가하기

기본적으로, 홈 (멘토들 리스트)으로 언제든 갈 수 있게 해준다. 이는 모든 페이지에 공통으로 들어간다. 
즉, base.html에 들어가면 좋다. 그러니 base.html의 <body> 바로 다음에 아래와 같이 넣는다.

```
<nav class="navbar navbar-expand-lg navbar-light bg-light border-bottom">
    <a class="navbar-brand" href="{% url 'mentor:index' %}">ADE</a>
    <button class="navbar-toggler ml-auto" type="button" data-toggle="collapse" data-target="#navbarNav"
            aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
    </button>
    <div class="collapse navbar-collapse flex-grow-0" id="navbarNav">
        <ul class="navbar-nav">
            <li class="nav-item ">
                <a class="nav-link" href="#">로그인</a>
            </li>
        </ul>
    </div>
</nav>
```

코드를 보면 navbar-brand, 즉 네이게이션 바에서 브랜드가 들어갈 위치에 글 ADE가 들어가게 되어있다.
navbar의 생긴 형태는 <button> 관련 문구로 정의가 되는 것 같다.
navbar-nav에서 nav-item, nav-link를 통해 로그인 관련 링크도 만들었다. (물론 구현되진 않았다.)

### 1.2 js 사용하기

bootstrap에 의해 로그인창은 화면의 크기에 따라 원래 자리에 있거나 상세목록으로 숨겨진다.
지금 상태로는 상세 목록을 볼 수 없는데, css를 전혀 손보지 않았기 떄문이다. 

웹사이트 지시하는대로 bootstrap.min.js 와 jquery-3.5.1.min.js를 static폴더 안에 넣어준다.
그리고 base.html의 endblock 뒤에 아래와 같이 추가한다
```
<script src="{% static 'jquery-3.5.1.min.js' %}"</script>
<script src="{% static 'bootstrap.min.js' %}"</script>
```

이제 상세목록을 누르면 로그인을 확인할 수 있다.


### 1.3 네비게이션 관련 html 재정리하기

이래선 base.html이 지저분하다. 관련 html을 따로 만들기 위해 같은 폴더에 navbar.html을 만든다.

navbar.html에 base.html에 있던 navbar관련 코드를 옮긴 후 base.html에는
{% include "navbar.html" %}로 대신한다.

## 2. Pagination

현재 메인페이지는 무수히 많은 관련 프로필을 한번에 출력한다. 이를 페이지로 나눠서 보려고 한다.
다행히, 이를 도와주는 툴이 있다.

### 2.1 views.py 수정하기

views.py를 아래와 같이 바꿔준다. 

```
from django.core.paginator import Paginator 

def index(request):
    page = request.GET.get('page', '1')
    profile_list = Profile.objects.order_by('-create_date')
    
    paginator = Paginator(profile_list, 10)
    page_obj = paginator.get_page(page)
    
    context = {'profile_list': page_obj}
    
    return render(request, 'mentor/profile_list.html', context)
```
Paginator에 의해 10개만 끊어서 profile_list를 뽑아오고, 이를 정리해서 출력한다.


### 2.2 template 수정하기

view를 수정했다면 당연히 template도 수정하게된다.

```
    <!--paging -->
    <ul class="pagination justify-content-center">
        <!-- previous page -->
        {% if question_list.has_previous %}
        <li class="page-item">
            <a class="page-link" href="?page={{profile_list.previous_page_number}}">이전</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">이전</a>
        </li>
        {% endif %}
        <!-- page lists -->
        {% for page_number in profile_list.paginator.page_range %}
            {if page_number==profile_list.number %}
            <li class="page-item active" aria-current="page">
                <a class="page-link" href="?page={{page_number}}">{{page_number}}</a>
            </li>
            {% else %}
            <li class="page-item">
                <a class="page-link" href="?page={{page_number}}"> {{page_number}}</a>
            </li>
            {% endif %}
        {% endfor %}
        <!-- next page -->
        {% if profile_list.has_next %}
        <li class="page-item">
            <a class="page-link" href="?page={{profile_list.next_page_number}}">다음</a>
        </li>
        {% else %}
        <li class="page-item disabled">
            <a class="page-link" tabindex="-1" aria-disabled="true" href="#">다음</a>
        </li>
        {% endif %}
    </ul>
    <!-- end pagination related html -->
```

멘토 등록하기 링크 위에 위와 같이 추가했다.
이전 페이지가 있을 경우, 다음 페이지가 있을 경우를 앞뒤에 구현하고, 사이에 페이지에 해당되는 내용을 출력하고 페이지 숫자를 출력한다.
views.py에서 {'profile_list': page_obj}를 적은게 해당 페이지에 들어갈 10개의 컨텐츠를 정리해준다.

bootstrap과 html의 형식을 복합적으로 사용했다. 한줄한줄 이해가 당장 필요해보이지는 않으니 자세한 설명은 하지 않겠다.

### 2.3 pagelist 만들기

구글이나 다양한 검색엔진등을 보면 한번에 10개 페이지만 페이지리스트에 보인다. 지금대로면 1~1000개가 한페이지에 도배된다.
이를 정리하자.

page list출력하는 부분에서, 단순 출력 대신

```
 {% if page_number >= profile_list.number|add:-5 and page_number <= profile_list.number|add:5 %}
 {% if page_number == profile_list.number %}
 ```
 
을 사용해주자 (당연히 endif가 하나 추가된다). 하나의 if문이 바뀌고, 하나가 추가된다.
기본적인 if문의 사용이다.
신기한점은, 직접 | (백스페이스 밑)을 사용하면 에러가 뜨는데, 예시에서 주는것을 복붙하면 에러가 없다.
눈으로 보기엔 차이가 안보이는데, 무슨 문제인지 모르겠다. 

## 3. index 처리

목록을 보면 제일 최근 등록부터 1번이다.
두번쨰 페이지가 생기면 그 페이지는 다시 또 1번~10번이다. 
다양한 게시글을 보면 가장 오래된것이 1번이 붙고, 제일 최근것은 n이란 숫자가 붙는다.
페이지마자 index도 다르다. 이를 정리해본다.

### 3.1 탬플릿 필터

index를 알아서 계산해주는 툴은 없는 것 같다.
그렇다면 이를 수기로 계산해야하는데, 이를 매번 하기는 번거로우니 이를 정리하는 필터를 따로 만든다.
마치 C에서의 macro와 같은 역할이 될 것 같다.

우선 hr/hr/templatetags를 만들고, mentor_filter.py를 만든다. 그리고 아래와 같이 추가한다

```
from django import template

register = template.Library()

@register.filter
def sub(value, arg):
    return value - arg
```

그리고 profile_list.html에 아래와 같이 수정한다

기존에 {{ for ~~~}} 였던 부분을

```
{{profile_list.paginator.count|sub:profile_list.start_index|sub:forloop.counter0|add:1}}
```
로 대체한다. 
번호 = 전체건수 - 시작인덱스 - 현재인덱스 + 1 라는 뜻이다.

## 4. 댓글 수 함께 입력하기

각 프로필 옆에 후기가 몇개 있는지 입력해보자.
조회가 몇개인지, 평점이 몇점인지도 이런식으로 입력이 가능할 것 같다.
<a href= "{% ... <> {{profile.title}}</a} 아래에 이를 추가한다
```
{% if profile.comment_set.count > 0 %}
<span class="text-danger small ml-2"> {{profile.comment_set.count }}</span>
{% endif %}
```
직관적인 사용방식이다.


