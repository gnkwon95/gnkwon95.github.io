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


## 5. 로그인/ 로그아웃

서비스의 핵심이 될 수 있는 로그인/아웃 방식에 대한 코드다.

### 5.0 기본 설정

settings.py의 installed_apps에 자동으로 추가되는 django.contrib.auth 라를 툴을 사용한다.
우선 hr/hr에 mentor폴더와 나란히 common이랑 app을 새로 만든다. (django-admin startapp common).
(설명을 따라가다보면 settings.py가 들어있는 hr디랙터리의 이름이 config으로 되어있다.)
config의 settings의 installed_apps에 'common.apps.CommonConfig',를 추가해준다.
config의 urls에는 path('common/', include('common.urls'))를 추가한다. (mentor와 비슷)
그리고 commons/urls.py에 mentor/urls.py와 비슷한 탬플릿을 준비해둔다.

### 5.1 로그인 환경 설정

우선 navbar.html의 href 에서 href="{% url 'common:login' %}" 를 수정해준다. 
현재 #로 비움처리되어있는 부분에 실제 direct할 위치를 알려준다. 물론, 지금 common이란 app_name에 login은 없다.
common/urls.py에 아래를 추가해준다.
```
from django.contrib.auth import views as auth_views
path('login/', auth_views.LoginView.as_view(), name='login')
```
이 툴은 직접 login관련 페이지의 views.py함수를 만들 필요를 대신해준다.
views 관련 함수들은 대신해주지만 시각화 탬플릿 (html)은 직접 제작해야한다.
그러니 as_view 안의 내용을 auth_views.LoginView.as_view(template_name ='common/login.html') 로 지정한다. 이제 common/login.html을 쓰기만 하면 된다.

예시에서 시키는 내용과 똑같이 입력한다.
```
{% extends "base.html" %}
{% block content %}

<div class="container my-3">
    <form method="post" class="post-form" action="{% url 'common:login' %}">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="username">사용자ID</label>
            <input type="text" class="form-control" name="username" id="username"
                   value="{{ form.username.value|default_if_none:'' ">
        </div>
        <div class="form-group">
            <label for="password">비밀번호</label>
            <input type="password" class="form-control" name="password" id="password"
                   value="{{ form.password.value|default_if_none:'' }}">
        </div>
        <button type="submit" class="btn btn-primary">로그인</button>
    </form>
</div>
{% endblock %}
```

보면 include "form_errors.html"이 있다. 작성한 기억이 없으니, 당연히 작성해준다.
templages/form_errors.html에 아래와 같다
```
{% if form.errors %}
    {% for field in form %}
        {% for error in field.errors %}
            <div class="alert alert-danger">
                <strong> {{ field.label }}</strong>
                {{ error }}
            </div>
        {% endfor %}
    {% endfor %}
    {% for error in form.non_field_errors %}
        <div class="alert alert-danger">
            <strong> {{error }}</strong>
        </div>
    {% endfor %}
{% endif %}
```
이전에 멘토 등록 페이지와 제법 비슷하지만, 입력 시에도 다양한 이유로 에러 출력이 가능해진다. (예로 비번이 틀린다거나, 아이디가 없거나)
전체적 코드의 흐름은 비슷하다.

### 5.2 로그인 허용

위와 같이 하면 로그인 페이지가 생기고, 입력이 잘못되면 오류를 출력한다.
하지만 실제 로그인을 해주지는 않는다.
성공적인 로그인은 django.contrib.auth 에 따라 /accounts/profile을 검색한다. 당연히, 지금 없다.

우선, settings에 LOGIN_REDIRECT_URL = '/' 를 아무곳에나 추가해준다.
이대로라면, 로그인이 성공하면 '/'로 redirect 해야한다. 하지만 기본 urls.py에 '/'는 없다.
이를 추가해주자. 
```
from mentor import views
path('', views.index, name='index'),
```
를 하면, redirect로 mentor의 views 의 index페이지로 간다.
mentor 밑에 빨간줄이 생기면서 찾을 수 없다고 하는데... 수행은 잘 된다.

로그인 이후에는 로그아웃할 수 있어야하니 navbar.html의 로그인 출력 란을
```
 {% if user.is_authenticated %}
<a class="nav-link" href="{% url 'common:logout' %}"> {{user.username}}(로그아웃)</a>
{% else %}
<a class="nav-link" href="{% url 'common:login' %}">로그인</a>
{% endif %}
```
로 대신한다.

common/urls.py 에 ```path('logout/', auth_views.LogoutView.as_view(), name='logout'),``` 를 추가하고
settings.py에 ```LOGOUT_REDIRECT_URL = '/'``` 를 추가하면 로그아웃도 구현이 된다.

## 6. 계정 만들기

이제 로그인을 할 수 있으니, admin 외의 계정들도 만들 수 있으면 좋겠다.

### 6.1 링크 url 등 만들기
우선, login.html에 계정 생성 글을 추가한다.

```
<div class="row">
    <div class="col-4">
        <h4>로그인</h4>
    </div>
    <div class="col-8 text-right">
        <span>또는 <a href="{% url 'common:signup' %}">계정을 만드세요.</a></span>
    </div>
</div>
```

{% url 'common:signup' %}에 맞는 signup링크를 만들어야한다.
common/urls.py 에 아래를 추가한다
```
path('signup/', views.signup, name='signup'),
```
### 6.2 아이디 등록하기

이전에도, form을 받고 정보를 저장하기 위해서는 forms.py를 만들었다. 
이번에도 해보자. common/forms.py를 만들고 아래를 만든다. (참고로, 위의 views/signup도 잊지 말자.미완성이다.)

```
from django import forms
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User

class UserForm(UserCreationForm):
    email = forms.EmailField(label="이메일")

    class Meta:
        model = User
        fields = ("username", "email")
```
UserCreationForm은 아이디-비번1-비번2 구조를 받고 비번1, 비번2가 같은지 확인해준다. 
상식적으로 맞는 비밀번호가 입력되면 is_valid() 가 True가 된다.

### 6.3 아이디 생성 관련 페이지 만들기

드디어 views/signup의 차례다.
이번에도 1. 페이지를 띄운다. 2. POST시 확인하고 다른 페이지로 돌린다. 두가지를 수행하는 signup view를 만들어야한다.
```
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login
from common.forms import UserForm

def signup(request):
    if request.method == "POST":
        form = UserForm(request.POST)
        if form.is_valid():
            form.save()
            username = form.cleaned_date.get('username')
            raw_password = form.cleaned_data.get('password1')
            user = authenticate(username=username, password=raw_password)
            login(request, user)
            return redirect('index')
    else:
        form = UserForm()
    return render(request, 'common/signup.html', {'form': form})
```

먼저, else를 보면 페이지를 기본으로 띄울때는 common/signup.html로 간다. 물론 아직 없는 페이지다.
그리고 POST를 넣으면 UserForm class를 채워넣고, is_valid()이면 정보를 입력한 후 로그인하고, 메인페이지로 redirect한다.
cleaned_data는 개별항목을 따로 받기 위해 쓴다고 하는데, 왜 쓰는지 사실 잘 모르겠다.

이제 singup.html을 만들 차례다. templages/common/signup.html을 아래와 같이 채운다
```
{% extends "base.html" %}
{% block content %}

<div class="container my-3">
    <div class="row my-3">
        <div class="col-4">
            <h4>계정생성</h4>
        </div>
        <div class="col-8 text-right">
            <span>또는 <a href="{% url 'common:login' %}">로그인 하세요.</a></span>
        </div>
    </div>
    <form method="post" class="post-form">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="username">사용자 이름</label>
            <input type="text" class="form-control" name="username" id="username"
                   value="{{ form.username.value|default_if_none:'' }}">
        </div>
        <div class="form-group">
            <label for="password1">비밀번호</label>
            <input type="password" class="form-control" name="password2" id="password2"
                   value="{{ form.password.value|default_if_none:'' }}">
        </div>
        <div class="form-group">
            <label for="email">이메일</label>
            <input type="text" class="form-control" name="email" id="email"
                   value="{{ form.email.value|default_if_none:'' }}">
        </div>
        <button type="submit" class="btn btn-primary">생성하기</button>
    </form>
</div>

{% endblock %}
```
계정생성 또는 로그인칸이 위에 있고
form_errors를 포함한 form을 받는다. 사용자, 비밀번호, 이메일을 입력받고, 제일밑에 생성하기 버튼이 있다.

이제 아이디를 자유롭게 만들 수 있다.

## 7. 새로운 항목 만들기

예시에서는 프로필을 올린 사람의 계정이 질문 옆에 추가되는 항목을 새로 만드는 것을 한다.
이제 아이디가 생겼으니 당연히 질문자/ 답변자의 아이디를 추가할 수 있다.
반대로 나는 이 외에 새로운 항목 여러가지를 추가하고자 한다. 
당장 떠오르는것은
1. 이전 이력을 단일항목에서 여러개 리스트로 추가하는것과
2. 합격 기업-직군 및 불합격 기업-직군 리스트 작성하기
이다. 물론 우선, 예시를 따라 먼저 아이디 입력을 추가하겠다.

### 7.0 기본 설정 설명

예시에는 author이 없기에 author을 추가한다. 
하지만 나의 모델은 작성자의 id와 본명이 다르다. 그러기에 author은 그대로 두고, 원래 author의 이름을 user_id라고 하겠다.
(원래 id를 쓰려고 했는데 기본설정값이라 사용하지 못하는 것 같다)
하지만 노출되는 이름은 의도대로 id 가 아닌 author로 하겠다.

### 7.1 models.py 수정

오랜만에 mentor/models.py에 돌아와서 항목을 추가하자. (Profile 부분)
```
from django.contrib.auth.models import User
user_id = models.ForeignKey(User, on_delete=models.CASCADE)
```
를 추가한다.

그리고 python manage.py makemigrations 후 python manage.py migrate를 수행한다.
무언가 에러 비스무리한게 뜨는데, 처리법은 앞서 이미 다뤘다.

비슷하게 comment에 추가한다.

### 7.2 views.py 수정

그럼 이제 user_id가 노출될 수 있게 설정을 바꿔보자.
profile_create, comment_create 함수 중 if form.is_valid() 안에 아래와 같이 추가한다
```
profile.user_id = request.user
```

### 7.3 login required

하지만 로그인이 되어있지 않다면, 에러가 출력될 것이다 (당연하다)
comment_create, profile_create 위에 아래와 같이 추가해준다.
```
@login_required(login_url='common:login')
```
물론 from django.contrib.auth.decorators import login_required 로 import가 필요한다.
안에 login_url을 넣어주며 이 에러가 출력될 시 로그인 페이지로 가이드해줄 수 있다.
또한 이 페이지를 통해 redirect 되어 로그인을 하면 처음부터가 아닌 해당 페이지로 다시 갈 수 있다.
url 에서 ?next=/ ~~~ 구문을 통해 다음페이지를 확인할 수 있다.

하지만 url이 해당 글처럼 된다고 바로 실행되지는 않는다. 이를 수정할 수 있다.
login.html에서, csrf_token 다음, include form errors 전에 아래와 같이 추가한다.
```
<input type="hidden" name="next" value="{{ next }}"
```

### 7.4 로그인 없는 후기 방지

생각해보면, 글 작성은 애초에 로그인하지 않으면 '작성하기'를 할 수 없으니 form페이지에 접근도 못한다.
하지만 후기는 로그인하지 않아도 form에 접근할 수 있고, 글을 쓴 후 '제출'을 해야 로그인을 요구한다.
UI가 매우 나쁠 것 같으니, 로그인하지 않으면 후기를 아예 쓰지도 못하게 하고싶다.

profile_detail.html 에서 후기 관련 textarea 부분에 if문을 추가해보자.
```
<textarea {% if not user.is_authenticated %}disabled{% endif %} name="content" id="content" class="form-control" rows="10"></textarea>
```

### 7.5 기존 이력 변경하기

이번엔 기존에 배운 내용을 응용할 것이다. 
현재 mentor/models.py를 보면 workExperience가 models.CharField()로 하나의 입력이 되어있다. 
리를 길이가 제한된 리스트로 바꿀 수 없을까?



