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

## 7장 나머지와 8장, 9장 초반이 또 저장이 안됐다... 9장 중간부터 이어서 작성한다.

### 9.2 수정 버튼 만들기

이제 글을 수정할 권한을 주자. 본인의 글만 수정할 수 있으니 if문을 사용하고, 이때엔 버튼을 추가한다.
아래는 profile_detail.html
```
 {% if request.user == profile.user_id %}
<div class="my-3">
    <a href = "{% url 'mentor:profile_modify' profile.id %}"
       class="btn btn-sm btn-outline-secondary">수정</a>
</div>
 ```
 이전에 말한대로 author는 글쓴이의 이름 (중복가능), user_id는 고유값이므로 author이 아닌 user_id를 쓴다.
 
 ### 9.3 url, views 편집
 
 mentor/urls.py 에 
 ```path('profile/modify/<int:profile_id>/', views.profile_modify, name='profile_modify'),```
 를 추가하고
 
 views.py에는 아래를 추가한다.
 
 ```
 from django.contrib import messages
 
 @login_required(login_url = 'common:login')
def profile_modify(request, profile_id):
    profile = get_object_or_404(Profile, pk=profile_id)
    if request.user != profile.author:
        messages.error(request, '수정권한이 없습니다')
        return redirect('mentor:detail', profile_id=profile.id)

    if request.method == "POST":
        form = ProfileForm(request.POST, instance=profile)
        if form.is_valid():
            profile=form.save(commit=False)
            profile.user_id = request.user
            profile.modify_date=timezone.now()
            profile.save()
            return redirect('mentor:detail', profile_id=profile.id)
    else:
        form = ProfileForm(instance=profile)
    context = {'form': form}
    return render(request, 'mentor/profile_form.html', context)
 ```
 우선, 권한이 없을 시 수정권한이 없습니다를 출력해준다. 
 else(get) 에서는 ProfileForm페이지가 호출되고, 저장하기 버튼을 누르면 (POST) 데이터를 저장하고 redirect한다.
 
 GET에서 instance=profile은 ProfileForm에서 기존에 채워져있는 데이터가 수정 전 형태가 될 수 있게 함을 뜻한다.
 POST에서는 instance=profile로 하되 request.POST부분을 후에 덮어씌운다.
 
 ### 9.4 비슷하게 삭제 만들기
 
 우선 html을 수정한다. 기존 수정 글 밑에 아래 코드를 붙이면 삭제 버튼도 같이 생긴다.
 
 ```
 <a href="#" class="delete btn btn-sm btn-outline-secondary"
data-uri="{% url 'mentor:profile_delete' profile.id %}">삭제</a>
```
 class에 삭제항목을 추가했고
 data-uri를 통해 jQuery를 가져온다. profile_delete 함수를 만들어야한다.
 
 ### 9.5 jQuery 사용하기
 
 javascript는 유저와 interaction이 가능하게 하는 입체적인 코드다.
 삭제 버튼을 누르면 확인을 하고, 여기서 확인이 되면 정말 삭제하는 코드를 만들 수 있다.
 우선 jQuery를 사용하기 위해 기본 html에 이를 허락해주는 코드를 짜야한다.
 
 base.html아래부분에 아래 코드를 추가하자.
 
```
{% block script %}
{% endblock %}
```
 script 관련 코드를 base.html의 연장된 html에서 사용하겠다는 뜻이다.
 참고로, jQuery 3.5.1은 이전에 이미 가져왔다. 다만 이와 관련된 코드를 쓰겠다고 선언을 안했을 뿐이다.
 
 
 아래는 필요한 jQuery 코드이다. 이를 profile_detail.html 끝부분에 추가해주면 된다.
 ```
 {% block script %}
<script type='text/javascript'>
$(document).ready(function(){
    $(".delete").on('click', function() {
        if(confirm("정말로 삭제하시겠습니까?")) {
            location.href = $(this).data('uri');
        }
    });
});
</script>
{% endblock %}
 ```
 javascript형태를 가져오고, 화면 로드 시 바로 사용되는 jQuery (확인 버튼 출력 시 나오는) 경우에 .delete함수가 적용되고, 클릭으로 작동한다.
 정말 삭제하겠습니다 버튼에 confirm을 누르면 기존 ui로 다시 돌아간다.
 
 ### 9.6 노가다 반복
 
 mentor: profile_delete를 사용한다고 했으니, 관련 url을 만들어준다.
 ```
 path('profile/delete/<int:profile_id>/', views.profile_delete, name='profile_delete'),
```

그리고 관련 함수를 views.py에 만들어준다.
```

@login_required(login_url='common:login')
def profile_delete(request, profile_id):
    profile = get_object_or_404(Profile, pk=profile_id)
    if request.user!= profile.user_id:
        messages.error(request, '삭제권한이 없습니다')
        return redirect('mentor:detail', profile_id=profile.id)
    profile.delete()
    return redirect('mentor:index')
```
클래식하게 권한이 없으면 제한하고, 아니면 delete()를 적용한다. 
이번에도 author이 아닌 user_id를 사용한다.

```
    path('comment/modify/<int:comment_id>/', views.comment_modify, name='comment_modify'),
```
를 url에 추가해주고

views.py에 comment_modify 함수도 만들어준다. profile_modify와 거의 같다.


그리고 profile_form과 달리 comment_form은 따로 있지 않다.
프로필 상세페이지에서 바로 질문을 붙였기때문에 따로 form만을 위한 html을 만들지 않았다.
그래서 만든다. comment_form.html

```
{% extends 'base.html' %}

{% block content %}
<div class="container my-3">
    <form method="post" class="post-form">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="content">후기내용</label>
            <textarea class="form-control" name="content" id="content" rows="10">
                {{form.content.value|default_if_none:'' }}
            </textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>

{% endblock %}
```

## 10. 대댓 만들기

대댓도 만들 수 있다. 오랜만에 models함수에 새로운 class를 추가하고, 데이터의 구조를 크게 바꿀 계획이다.


### 10.1 models.py 클래스 생성
```
class Repl(models.Model):
    user_id = models.ForeignKey(User, on_delete=models.CASCADE)
    content=models.TextField()
    create_date=models.DateTimeField()
    modify_date=models.DateTimeField(null=True, blank=True)
    profile = models.ForeignKey(Profile, null=True, blank=True, on_delete=models.CASCADE)
    comment = models.ForeignKey(Comment, null=True, blank=True, on_delete=models.CASCADE)
```
위와 같은 새로운 클래스를 만들고, 이름을 리플로 한다. 프로필, 코멘트를 그대로 가져오고 user_id도 가져온다.
on_delete를 통해 상위 개념이 사라지면 하위 개념들도 모두 사라지게 된다. 글이 사라지거나 댓글이 사라지면 대댓도 사라진다.

migrations를 수행 한다.

### 10.2 노가다 수행

1. html에 버튼 추가
2. urls.py 매핑
3. forms.py 작성 (필요 없으면 생략)
4. views.py에 함수 작성
5. 4번에서 사용하는 탬플릿이 있다면 작성

이 순서대로 하면 된다. 자세한 내용은 코드를 참고한다.

#### profile_detail.html 추가

```
        {% if profile.repl_set.count > 0 %}
        <div class="mt-3">
            {% for repl in profile.repl_set.all %}
            <div class="repl py-2 text-muted">
                <span style="white-space: pre-line;"> {{repl.content}}</span>
                <span>
                    - {{ repl.user_id }}, {{repl.create_date}}
                    {% if repl.modify_date %}
                    (수정:{{repl.modify_date}})
                    {% endif %}
                </span>
                {% if request.user == repl.user_id %}
                <a href="{% url 'mentor:repl_modify_profile' repl.id %}" class="small">수정</a>
                <a href="#" class="small delete" data-uri="{% url 'mentor:repl_delete_profile' repl.id %}">삭제</a>
                {% endif %}
            </div>
            {% endfor %}
        </div>
        {% endif %}
    </div>
    <a href="{% url 'mentor:repl_create_profile' profile.id %}" class="small"><small>댓글 추가 ..</small></a>
</div>
```
마지막줄에 위 클래스를 보면 class="repl..." 부분이 있다. repl 라는 클래스는 bootstrap에 있지 않다. style.css에 만들어야한다.

```
.repl {
    border-top:dotted 1px #ddd;
    font-size:0.7em;
}
```
이를 추가해준다.  위에 점선을 추가하고 폰트 사이즈가 0.7em이 된다. 점선의 색은 #ddd 이다.

#### urls.py

사용된 url은 수정/ 삭제/ 댓글추가 기능들이다. 각자 href를 사용하니, 이들의 url과 함수들을 모두 작업해야한다.

```
    path('repl/create/profile/<int:profile_id>/', views.repl_create_profile, name='repl_create_profile'),
    path('repl/modify/profile/<int:repl_id>/', views.repl_modify_profile, name='repl_modify_profile'),
    path('repl/delete/profile/<int:crepl_id>/', views.repl_delete_profile, name='repl_delete_profile'),
```
댓글 등록에는 어디 등록하는지 (profile_id)가 필요하고 수정 삭제는 해당 대댓(repl_id)이 필요하다.

#### forms.py

CommentForm과 비슷한 아래 클래스를 만든다. (import Repl 잊지말자)
```
class ReplForm(forms.ModelForm):
    class Meta:
        model = Repl
        Fields = ['content']
        labels = {
            'content': '대댓내용',
        }
```

#### views.py

이제 views를 작업해야한다. (import CommentForm) 잊지말자


```
@login_required(login_url='common:login')
def repl_create_profile(request, profile_id):
    profile = get_object_or_404(Profile, pk=profile_id)
    if request.method == "POST":
        form = ReplForm(request.POST)
        if form.is_valid():
            repl = form.save(commit=False)
            repl.create_date = timezone.now()
            repl.profile = profile
            repl.user_id = request.user
            repl.save()
            return redirect('mentor:detail', profile_id=profile.id)
    else:
        form = ReplForm()
    context = {'form': form}
    return render(request, 'mentor/repl_form.html', context)
```

다음으로는 repl_modify_profile와 repl_delete_profile을 작성한다.

comment_modify 와 comment_delete을 복붙해서 조금만 수정했다. 새로운 개념은 없다.

```
@login_required(login_url='common:login')
def repl_modify_profile(request, repl_id):
    repl = get_object_or_404(Repl, pk=repl_id)
    if request.user != repl.user_id:
        messages.error(request, '댓글수정권한이 없습니다')
        return redirect('mentor:detail', profile_id=repl.profile.id)

    if request.method == "POST":
        form = ReplForm(request.POST, instance=repl)
        if form.is_valid():
            repl = form.save(commit=False)
            repl.user_id = request.user
            repl.modify_date = timezone.now()
            repl.save()
            return redirect('mentor:detail', profile_id=repl.profile.id)
    else:
        form = ReplForm(instance=repl)
    context = {'form': form}
    return render(request, 'mentor/repl_form.html', context)

@login_required(login_url='common:login')
def repl_delete_profile(request, repl_id):
    repl = get_object_or_404(Repl, pk=repl_id)
    if request.user!= repl.user_id:
        messages.error(request, '댓글삭제권한이 없습니다')
        return redirect('mentor:detail', profile_id=repl.profile_id)
    repl.delete()
    return redirect('mentor:index', profile_id=repl.profile_id)
```

#### comment_form.html

comment_form으로 대댓 인풋을 받으니 이 문서도 만들어준다.

```
{% extends 'base.html' %}

{% block content %}
<div class="container my-3">
    <h5 class="border-bottom pb-2">댓글달기</h5>
    <form method="post" class="post-form my-3">
        {% csrf_token %}
        {% include "form_errors.html" %}
        <div class="form-group">
            <label for="content">댓글내용</label>
            <textarea class="form-control" name="content" id="content" rows="3">
                {{form.content.value|default_if_none:'' }}
            </textarea>
        </div>
        <button type="submit" class="btn btn-primary">저장하기</button>
    </form>
</div>

{% endblock %}
```

이와 관련해서는 코드를 다 작성했으나 ADE프로젝트의 UI에는 필요하지 않아 전부 주석처리하고 넘어가겠다.
대댓 기능으로 피보팅해서 시도하려고 했으나 불필요하다고 느꼈고, 대댓버튼을 누르면 작은 form칸일 생기는 UI를 제작하기 난해해서 스킵한다.

## 11. views.py 정리

views.py파일에 상당히 많은 내용이 들어가고, 협업 시 작동이 어려울 수 있으니 파일들을 구분하겠다.
또한 예시에서는 두가지 방식을 알려주는데, 협업에 두번째 방식이 더 좋다고 하니 두번째 방식을 채택한다.

### 11.1 폴더 구분

mentor 내에 views 라는 폴더를 만들고 base_views, profile_views, comment_views로 나눈다
주석처리한 repl_views 내용은 base_views에 그대로 둔다.

### 11.2 urls.py 구분

메인 hr/urls.py에서 아래와 같이 바꾼다.
```
from django.contrib import admin
from django.urls import path, include
from ..mentor.views import base_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('mentor/', include('mentor.urls')),
    path('common/', include('common.urls')),
    path('', base_views.index, name='index'),
]
```

그리고 아래와 같이 mentor/views/urls.py의 내용을 바꿔준다. repl관련 함수는 사용하지 않는다.
```
from django.urls import path
#from . import views

from .views import base_views, profile_views, comment_views

app_name = 'mentor'

urlpatterns = [
    # base_views.py
    path('', base_views.index, name = 'index'),
    path('<int:profile_id>/', base_views.detail, name='detail'),
    
    #profile_views.py
    path('profile/create/', profile_views.profile_create, name='profile_create'),
    path('profile/modify/<int:profile_id>/', profile_views.profile_modify, name='profile_modify'),
    path('profile/delete/<int:profile_id>/', profile_views.profile_delete, name='profile_delete'),
    
    #comment_views.py
    path('comment/create/<int:profile_id>/', comment_views.comment_create, name='comment_create'),
    path('comment/modify/<int:comment_id>/', comment_views.comment_modify, name='comment_modify'),
]
```
이제 관련해서 나올 수 있는 각종 버그들을 확인하고 다음으로 넘어가자.

## 12. 추천기능 만들기

슬슬 좋아요 버튼을 만들어보자.

### 12.1 모델 변경

우선 모델에 좋아요를 추가한다. Profile, Comment 다 추가한다. 원래는 Repl에도 추가해야한다. 
User가 profile, comment 양쪽에 사용될 수 있기떄문에 각각의 구분이 필요하다. 그래서 related_name을 추가해준다.

```
voter = models.ManyToManyField(User, related_name='voter_profile')
```
comment의 경우 아래와 같은 수정도 필요하다.
``` user_id = models.ForeignKey(User, on_delete=models.CASCADE, related_name='user_id_profile') ```

### 12.2 버튼 생성

우선 탬플릿에 추천 관련 버튼을 만들어야한다. 다행히 class=recommend가 있으니 활용할 수 있다.
예시와는 조금 다른 방식을 채택했는데, 짧은 질문이 아니고 긴 자기소개가 되기에 아래처럼 추천만 간단히 넣었다

``` ## 해당부분은 후에 살짝 조정함.
<div class="row my-3"> <!-- 상단부분 구분 -->
    <div class="col-1"> <!-- 왼쪽 1의 비중을 추천으로 두고 -->
        <div class="bg-light text-center p-3 border font-weight-bolder mb-1">{{profile.voter.count}}</div>
        <a href="#"
           class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
    </div>

    <div class="col-11"> <!-- 나머지를 12개정도로 쪼개서 오른쪽으로 두고 -->
        {% if request.user == profile.user_id %} <!-- 수정/삭제 -->
        <div class="d-flex justify-content-end">
            <a href = "{% url 'mentor:profile_modify' profile.id %}"
               class="btn btn-sm btn-outline-secondary">수정</a>
            <a href="#" class="delete btn btn-sm btn-outline-secondary"
               data-uri="{% url 'mentor:profile_delete' profile.id %}">삭제</a>
        </div>
        {% endif %}
        <div class="d-flex justify-content-end"> <!-- 만든 날짜. 수정날짜도 추가해야함 -->
            <div class="badge badge-light p-2">{{profile.create_date}}</div>
        </div>
    </div>
</div> <!-- 여기까지 상단 내용 -->

```

### 12.3 정말 추천하시겠습니까?

해당 부분은 왜 넣는지 모르겠다. 좋으면 좋은거지. javascript를 조금 응용하는데, 우린 쓰지 않도록 하자.
자세히 보면 12.2 버튼 생성에도 data-uri를 쓰지 않는다.

### 12.4 노가다

url 매핑 등의 나머지 작업들을 해보자.

mentor/urls.py에 vote_views를 import 하고, 관련 path를 만든다. vote_views.py 파일도 생성한다.

vote_profile 함수 - 본인이 본인 글 추천을 못하게 한다. 그리고 messages로 에러를 출력한다.

```
@login_required(login_url='common:login')
def vote_profile(request, profile_id):
    profile = get_object_or_404(Profile, pk=profile_id)
    if request.user == profile.user_id:
        messages.error(request, "본인이 작성한 글은 추천할 수 없습니다")
    else:
        profile.voter.add(request.user)
    return redirect('mentor:detail', profile_id=profile.id)
```
본인은 직접 추천할 수 없게 설정을 하고 
profile.voter.add를 한다. 여기서 add는 기본으로 vote관련 주어진 함수로 1을 추가하고, 한 request.user당 한번밖에 누르지 못하게 한다.

messages를 어떻게 출력할지도 html에서 지정할 수 있다.

```
{% if messages %}
    <div class="alert alert-danger my-3" role="alert">
        {% for message in messages %}
        <strong>{{message.tags}}</strong>
        <ul><li>{{message.message}}</li></ul>
        {% endfor %}
    </div>
    {% endif %}
```
### 12.5 댓글 좋아요

메인 글과 비슷하게 하면 된다. 메인 글에서는 좋아요 밑에 글이 나왔지만, 후기에는 좋아요 옆에 글이 나오면 (원래 코드와 같으면) 좋을 것 같다.
우선, html부터 손보자.

```
{% for comment in profile.comment_set.all %}
    <div class="row my-3"> <!-- 수정 부분 - 일단 row 3을 먼저 고정한다 -->
        <div class="col-1"> <!-- 왼쪽 부분을 col-1로 나누고, 여기에 vote comment 내용을 넣는다 -->
            <div class="bg-light text-center p-3 border font-weight-bolder mb-1">{{comment.voter.count}}</div>
            <a href="{% url 'mentor:vote_comment' comment.id %}" class="recommend btn btn-sm btn-secondary btn-block my-1">추천</a>
        </div>
        <div class="col-11"> <!-- 이후 오른쪽 영역을 col-11로정의하고, 그 안에 카드/ 카드안에 후기들을 넣는다 -->
            <div class="card">
                <div class="card-body">
                    <div class="card-text" style="white-space: pre-line;">{{comment.content}}</div>
                    <div class="d-flex justify-content-end">
                        <div class="badge badge-light p-2 text-left">
                            <div class="mb-2">{{comment.user_id}}</div>
                            <div>{{comment.create_date}}</div>
                        </div>
                    </div>
                    {% if request.user == comment.user_id %}
                    <a href="{% url 'mentor:comment_modify' comment.id %}" class="btn btn-sm btn-outline-secondary">수정</a>
                    {% endif %}
                </div>
            </div>
        </div>
    </div>
    {% endfor %}
```
이런 모양이 나온다. 

### 노가다 반복

urls.py에 

``` path('vote/comment/<int:comment_id>/', vote_views.vote_comment, name='vote_comment'), ```
를 추가하고

comment_views.py에 아래 코드를 넣는다

```
@login_required(login_url='common:login')
def vote_comment(request, comment_id):
    comment = get_object_or_404(Comment, pk=comment_id)
    if request.user == comment.user_id:
        messages.error(request, '본인이 작성한 글은 추천할 수 없습니다')
    else:
        comment.voter.add(request.user)
    return redirect('mentor:detail', profile_id=comment.profile.id)
```
이러면 의도한 디자인이 나온다!

## 13. 앵커 만들기

스크롤을 돌리고, 다른 곳으로 튀었다가 돌아오면 맨 위가 아닌 이전 위치 혹은 그에 가까운곳으로 호출하고싶다.
이를 앵커라고 한다. 꼭 지금수준에서 필요한지 모르지만 일단 만들자.

### 13.1 html

profile_detail.html에 아래를 추가한다
``` <a name="comment_{{comment.id}}"></a> ```
여기에 앵커 고유의 이름이 저장되는 것이다.

### 13.2 views

comment_views.py에서 redirect를 포맷팅을 이용해 아래와 같이 바꾼다.

```
return redirect('{}#comment_{}'.format(
                resolve_url('mentor:detail', profile_id=profile.id), comment.id))
```
comment_create, comment_modify 둘 다 적용한다. 살짝 다른것을 확인할 수 있지만, 쉽게 이해할만하다.

### 13.3 대댓

예시글은 대댓에도 이를 적용하지만, 우리는 대댓이 없으므로 무시한다. 다음으로 넘어가자.

## 14. 마크다운

마크다운은 글씨에 포맷팅을 해주는 것이다. 지금 깃헙에 쓸때 제목에 ## 를 쓰거나 * 를 쓰면 포맷이 바뀌는 것과 같다.
이 기능을 제공하면 글쓴이가 편할 수 있지만, 이 기능을 제대로 아는 사람이 몇명이 될지 모르겠다 (이게 노션도 아니고...)
그래도 혹시 이를 나중에 수기로 적용할 수 있게 추가하거나..? 할 수 있으니 일단은 만들어보자. 

아니다. 이게 블로그도 아니고.. 나중에 추가하자 그딴건.

## 15. 검색창

이건 필요하다. 검색을 해야하지 않겠나.
나중에 전체/ 제목/ 글쓴이/ 기업/ 자기소개글 등으로 검색 세분화가 가능하겠지만, 우선은 짬뽕된 or 함수에 집중하다.

### 15.1 profile_list.html

오랜만에 프로필 검색 리스트 페이지를 보자. 상단에 아래를 추가해주자. 검색창 input을 받고, kw를 받는다. 검색한 value는 kw로 입력받고 이를 검색한다.

```
<div class="row justify-content-end my-3"> <!--검색창 -->
        <div class="col-4 input-group">
            <input type="text" class="form-control kw" value="{{ kw|default_if_none:'' }}">
            <div class="input-group-append">
                <button class="btn btn-outline-secondary" type="button" id="btn_search">찾기</button>
            </div>
        </div>
    </div>
```

그리고 페이지의 제일 마지막, endblock 전에 아래와같은 form을 넣어준다

```
<form id="searchForm" method="get" action="{% url 'index' %}">
    <input type="hidden" id="kw" name="kw" value="{{kw|default_if_none:'' }}">
    <input type="hidden" id="page" name="page" value="{{page}}">
</form>
```

index라는 함수가 kw, page 값을 저장하고 context로 전달해준다.

또한 기존 페이지처리는 링크를 href=?page ~~~ 식으로 읽어오게 되어있다.
이를 data-page를 써서 자유롭게 바뀌도록 (?) 한다. 직접 페이지값에 이름을 부여하지 않고 page-data 속성을 부여해서 이를 통해 값을 읽는 형식이다.

### 15.2 profile_list javascript

동적 계산이 필요하기 때문에 javascript를 사용한다. block script와 endblock 사이에 작성된다.

```
{% block script %}
<script type='text/javascript'>
    $(document).ready(function(){
        $(".page-link").on('click', function(){
            $("#page").val($(this).data("page"));
            $("#searchForm").submit();
        });
        
        $("#btn_search").on('click',function(){
            $("#kw").val($(".kw").val());
            $("#page").val(1);
            $("#searchForm").submit();
        });
    });
</script>
{% endblock %}
```
이부분은 새로운 분야라 조금 설명이 필요하다.

우선 <a class="page-link"...> 부분을 보면 page-link란 class에 접근시의 인풋이 주어진다.
여기 스크립트에서 $(".page-link") 부분이 해당 함수가 활성화 (ready) 되면 특정 script 관련 함수를 수행하라는 것을 의미한다.

그리고 검색버튼을 클릭하면 $("#btn_search") 이 on (click) 될때 특정 함수를 수행하도록 되어있다.
각각 무언가 함수 수행을 하고 특정 Form을 활성화 (submit) 한다.

val(1)은 무언가 검색하면 늘 1페이지로 노출되도록 설정하는 것이다.

### 15.3 base_views.py

이제 이 함수들을 적용할 기본페이지에 함수들을 추가하면 된다.

index 함수의 첫부분을 아래처럼 수정해준다
```
page = request.GET.get('page', '1') #페이지
    kw = request.GET.get('kw', '') #검색어 추가
    
    profile_list = Profile.objects.order_by('-create_date')
    if kw:
        profile_list=profile_list.filter(
            Q(title__icontains=kw) |
            Q(school__icontains=kw) |
            Q(workExperience__icontains=kw) |
            Q(PR__icontains=kw) |
            Q(comment__user_id__username__icontains=kw) 
        ).distinct()
 <...>
 context={'profile_list': page_obj, 'page': page, 'kw': kw}
```
하위 모델속성을 보기 위해서는 __ (언더바 두개)를 사용하고, html에 page 와 kw도 넘겨주기 때문에, 이도 context에 추가해준다.
왜인지 모르겠지만 comment__content__icontains=kw는 에러가 나온다. 
    
### 15.4 정렬하기

검색이 가능하다면, 비슷하게 page, kw, get, jquery등을 활용해서 정렬도 가능할것이다.

우선 정렬 기준을 볼 수 있게 해보자. 아래를 상단에 넣으면 3개 옵션이 있는 간단한 form을 쓸 수 있다. 셋중 하나를 드롭다운으로 검색할 수 있어진다.


```
<div class="col-2">
    <select class="form-control so">
        <option value="recent" {%if so == 'recent' %}selected{% endif %}>최신순</option>
        <option value="recommend" {%if so == 'recommend' %}selected{% endif %}>추천순</option>
        <option value="popular" {% if so == 'popular' %}selected{% endif %}>인기순</option>
    </select>
</div>
```
그리고 아래에, searchForm에 해당되는 숨겨진 타입중에 so를 통한 searchForm도 추가해주자.

```
<input type="hidden" id="so" name="so" value="{{so}}">
```

아래 자바스크립트부분의 ready function 에도 아래와 같이 so에 해당하는 script를 추가해준다.

```
$(".so").on('change', function() {
    $("#so").val($(this).val());
    $("#page").val(1);
    $("#searchForm").submit();
});
```

### 15.5 base_views.py의 index함수 수정하기

아까와 같이 함수를 수정해주어야한다. 각자 버튼을 누르면 어떤 일을 할지 (정렬, 어떤 기준) 알려줘야한다.
이제 검색 기준이 생겼으니, order_by의 순서에 따른 profile_list가 매번 다르게된다.
코드는 아래와 같다.

```
so = request.GET.get('so', 'recent') #정렬 기준

if so == 'recommend':
    profile_list = Profile.objects.annotate(num_voter=Count('voter')).order_by('-num_voter', '-create_date')
elif so=='popular':
    profile_list=Profile.objects.annotate(num_comment=Count('comment')).order_by('-num_comment', '-create_date')
else: #recent
    profile_list=Profile.objects.order_by('-create_date')
```
정렬기준을 추가하고, 디폴트로 recent가 되어있다. 
하지만 get을 통해 다른게 될 수 있다. 
그리고 투표수가 같으면 최신순으로 정렬되도록 두가지 목적을 두었다.
annotate의 사용방법은 쿼리와 비슷해보여서 어렵지 않고, 결국 order_by가 정렬기준이다.
보면 인기순은 후기순으로 정렬되어있다. 그냥 헷갈리지 않게 후기순으로 해야겠다.
끝에 content에 so를 추가해주는것을 빼먹지 않는다.

사용해보면 정렬 필터를 넣고, 검색 필터도 넣을 수 있다.
작동 원리는 조금 복잡하지만 코드 자체는 매우 간단하다.

## 16. 추가 기능

생각하는 추가 기능은 아래와 같다.

1. 답변 페이징/ 정렬 (지금은 최신순. 메인페이지처럼 한페이지에 10개만 들어가게 변경, 좋아요 순 등으로 정렬)

2. 카테고리 
	2.1 하나의 자유게시판에서 인턴/정규직/이직 등 카테고리 나누기
	2.2 상단바에 다른 내용 추가 (채팅이 되겠지)

3. 비밀번호 분실과 변경/ 아이디...
	3.1 내 페이지 내에서 아이디/ 비번 변경
	3.2 비번 찾기는 이메일로 임시 비번 보내야함. 어려움

4. 프로필 만들기
	4.1 여기에 아디비번 바꾸기 링크 추가
	4.2 자기소개글, 올린 글/댓글 수, 최근접속 등 추가
	4.3 유저가 자기 개인 사진 업로드할 수 있게 기능 추가

5. 최근 답변, 최근 댓글
	5.1 비교적 쉬운데, 게시판이 아니니 필요 없을수도

6. 조회수 표시
	6.1 일단 조회도 트레킹 해야하는데... 어떻게 하지.
	6.2 조회순으로 정렬 기능 추가
	6.3 조회 수 실시간으로 바뀌는 칼럼 메인에 추가

7. 소셜 로그인
	7.1 소셜미디어로 로그인 가능하게 하기

8. 채팅 서비스 구현하기 (상단 바에서)

+ 멘토 정보 입력란 업데이트 (리스트로 해서 jQuery 얹기. 자격증+점수 or 경험+년도)
+ 멘토 프로필 나눠서 보기 - 네이버쇼핑처럼 맨 위에는 자기소개와 여기저기 정보 왔다갔다 할 수 있는 탭 추가
+ 회원가입시 약관동의/ 개인정보 확인
+ 전화번호로 확인문자 보내기
+ 연락 신청하기
+ 멘토 정보 등록 시 승인 대기 기능 (바로 올라가지 않게)
+ 멘토 자기 정보 검증 기능 추가하기 - 당장은 지인 위주로 해서 자격증 확인 제출란은 필요 없을듯
+ 슈퍼계정 만들어서 승인해줄 수 있게 하기
+ 멘토 available 확인 및 기능 추가하기 - on/off 까지는 아니어도 어느날에 되는지.. UI좀 고민해봐서 달력에 회색/초록색 표시를 하거나 등
+ 멘토 멘티 계정 구분하기 - 회원가입 시 두개로 갈라서. 어떻게 UI 다를지도 괸
+ 멘티 계정 멘토계정으로 전환할 수 있는 UI 추가하기

정말 많구나... 화이팅..
