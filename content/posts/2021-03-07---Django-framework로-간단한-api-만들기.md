---
title: "Django framework로 간단한 api 만들기"
date: "2021-03-07 22:31:00"
template: "post"
draft: false
category: "Backend"
tags:
  - "Backend"
  - "Server"
  - "Api"
  - "Framework"
description: "Django framework를 활용해 간단한 api를 만들어보고 Django의 기본 개념을 익힙니다."
---

본 포스트는 [인프런의 플러터와 장고로 1시간만에 퀴즈 앱/서버 만들기 [무작정 풀스택] 강의](https://www.inflearn.com/course/%ED%94%8C%EB%9F%AC%ED%84%B0-%EC%9E%A5%EA%B3%A0-%ED%80%B4%EC%A6%88%EC%95%B1-%EC%84%9C%EB%B2%84-%ED%92%80%EC%8A%A4%ED%83%9D)를 듣고 내용을 정리한 것입니다. 퀴즈 모델에 관한 api를 만들어봅니다.

<a id="venv-header"></a>

## 먼저 [가상환경](#venv-content)을 설정합니다.

가상환경을 설정하는 명령어는 다음과 같습니다.
```python
// 가상환경 설정
python3 -m venv venv(본인의 가상환경 이름)
// 가상환경 활성화
source venv/bin/activate
```
그리고 Django project를 cli로 생성합니다.
```python
// 프로젝트 생성 (.을 꼭 적어야함)
django-admin startproject myapi .
python manage.py startapp quiz
```

## settings.py 파일을 수정합니다.

```python
// 모든 호스트 허용
ALLOWED_HOSTS ['*']
// 타임존 변경
TIME_ZONE = 'Asia/Seoul'
// 스태틱 파일 경로 추가
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
// INSTALLED_APPS에 'quiz', 'rest_framework' 추가
```
<a id="installed-app-header"></a>

[INSTALLED_APP](#installed-app-content)

<a id="migration-header"></a>

## [migration](#migration-content)을 합니다.

```python
// migration 파일 생성
python manage.py makemigrations
// migration 진행
python manage.py migrate
// 서버 구동
python manage.py runserver
```

여기까지 하면 기본적인 서버를 만들었습니다.

<a id="model-header"></a>

## 다음으로 [Model](#model-content)을 생성하겠습니다.
```python
// quiz/models.py
from django.db import models

# Create your models here.
class Quiz(models.Model):
    title = models.CharField(max_length=200)
    body = models.TextField()
    answer = models.IntegerField()
```

<a id="serializer-header"></a>

## [Serializer](#serializer-content) 생성

```python
// serializers.py
from rest_framework import serializers
from .models import Quiz

class QuizSerializer(serializers.ModelSerializer):
    class Meta:
        model = Quiz
        fields = ('title', 'body', 'answer')
```

Model과 Serializer를 만들었으니, 이제는 실제로 api를 만들어봅니다.

## api 생성
<a id="view-header"></a>

[view](#view-content)
```python
// quiz/views.py
from rest_framework.response import Response
from rest_framework.decorators import api_view
from .models import Quiz
from .serializers import QuizSerializer
import random
# Create your views here.
@api_view(['GET'])
def randomQuiz(request, id):
    totalQuizs = Quiz.objects.all()
    // sample의 두번째 인자는 length
    randomQuizs = random.sample(list(totalQuizs), id)
    // serializer에 many=True를 주면, 다수의 데이터도 직렬화해줍니다.
    serializer = QuizSerializer(randomQuizs, many=True)
    return Response(serializer.data)
```

## 이제 api를 url에 연결하면 됩니다.
myapi내의 `urls.py`는 전체 app에 대한 url을 관리하고, `quiz/urls.py`는 quiz app에 대한 url만 관리합니다.
```python
// quiz/urls.py
from django.urls import path, include
from .views import randomQuiz

urlpatterns = [
    path("<int:id>/", randomQuiz)
]
```
```python
// myapi/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('quiz/', include('quiz.urls')),
]
```

<a id="dynamic-url-header"></a>

[django에서 동적 url 작성법](#dynamic-url-content)


## api 개발을 완료했습니다. 이제 model data를 생성하기 위해 admin을 등록해봅니다.
```python
// quiz/admin.py
from django.contrib import admin
from .models import Quiz

# Register your models here.
admin.site.register(Quiz)
```

model이 변경되었기 떄문에 migration을 한번 더 진행합니다.

![image](https://user-images.githubusercontent.com/32301380/110243829-6d95e600-7f9f-11eb-90a3-7aad363ddebe.png)

## 이제 admin에 접속할 수 있는 슈퍼계정을 만듭니다.

```
python manage.py createsuperuser
```

## 서버를 구동해봅니다.
```
python manage.py runserver
```

## 어드민에 접속해서 Model을 만들어봅니다.

![image](https://user-images.githubusercontent.com/32301380/110244000-13e1eb80-7fa0-11eb-8ed6-ab76b5ccdbcb.png)

## Model data를 4개 이상 만들고 "/quiz/3"을 호출해봅니다.
api가 잘 작동하는 것을 확인할 수 있습니다!

![image](https://user-images.githubusercontent.com/32301380/110244192-e5b0db80-7fa0-11eb-8e5b-4acb4ae19639.png)

이렇게 django framework로 model 생성 및 api를 만들어보았습니다. 코드를 따라치다가 원리가 궁금해서 내용을 정리해보았습니다. 다음에는 flutter app부분도 내용을 정리할까 합니다.

## 내용 정리

<a id="venv-content"></a>
[돌아가기](#venv-header)
<details>
<summary>
가상환경이란
</summary>
가상환경은 해당 프로젝트의 Python 개발환경을 구축하기 위해 모듈을 독립적으로 관리하는 환경입니다. 가상환경을 설정하는 이유는 각 모듈이 다른 모듈에 의존성을 가지고 있기 때문에 이것저것 설치하다가 충돌이 날 수 있기 때문입니다.
</details>
<br/>

<a id="installed-app-content"></a>
[돌아가기](#installed-app-header)
<details>
<summary>
INSTALLED_APPS란
</summary>

[참고](https://docs.djangoproject.com/ko/3.1/intro/tutorial02/)<br/>
INSTALLED_APPS는 Django와 함께 딸려오는 다음의 앱들 등을 포함합니다.
- django.contrib.admin - 관리용 사이트
- django.contrib.auth - 인증 시스템
  
그외 앱들
...

Django 인스턴스에서 활성화된 모든 Django application들의 이름을 담고 있습니다.

</details>
<br/>

<a id="migration-content"></a>
[돌아가기](#migration-header)
<details>
<summary>
migration이란
</summary>

[참고](https://docs.djangoproject.com/ko/3.1/intro/tutorial02/)<br/>
Django framework는 기본적으로 SQLite를 사용하도록 구성되어 있습니다. migrate 명령어를 실행하면 INSTALLED_APPS를 참고해 필요한 데이터베이스 테이블을 생성합니다.

</details>
<br/>

<a id="model-content"></a>
[돌아가기](#model-header)
<details>
<summary>
Model field reference
</summary>

[참고](https://docs.djangoproject.com/en/3.1/ref/models/fields)
- CharField
    - `class CharField(max_length=None, **options)`
    - small to large string field
    - 대량의 텍스트는 TextField를 사용할 것
    - max_length는 required argument
- TextField
    - `class TextField(**options)`
    - large textfield
- InterField
    - `class IntegerField(**options)`
    - An integer
  
</details>
<br/>

<a id="serializer-content"></a>
[돌아가기](#serializer-header)
<details>
<summary>
Serializer란
</summary>

[참고](https://www.django-rest-framework.org/api-guide/serializers/)<br/>
쿼리셋, 모델 인스턴스 등의 복잡한 데이터를 JSON, XML 등의 컨텐츠 타입으로 쉽게 변환시켜주는 역할.
여기서는 Django의 모델 데이터를 JSON 타입으로 바꿔주는(직렬화) 코드(클래스)
### ModelSerializer
이 class는 `Serializer` class와 같지만 다음 차이점이 있습니다.
- model을 기반으로 field를 자동생성합니다.
- validator를 자동생성합니다.
- 간단한 `.create()`, `.update()` 메서드를 기본적으로 가지고 있습니다.

따라서, 간단하게 Serializer를 만들 때 사용됩니다.

</details>
<br/>

<a id="view-content"></a>
[돌아가기](#view-header)
<details>
<summary>
view란
</summary>

[참고](https://docs.djangoproject.com/en/3.1/topics/http/views/)<br/>
프론트엔드 개발자로서 view란 view template을 의미하는 줄 알았지만, Django에서의 view function은 **Web request를 받고 Web response를 return**하는 역할을 합니다. express framework의 route와 유사하다는 생각이 들었습니다.

</details>
<br/>

<a id="dynamic-url-content"></a>
[돌아가기](#dynamic-url-header)
<details>
<summary>
django에서 동적 url 작성법
</summary>

- str - '/'를 제외한 비어있지 않은 문자열
- int - 0이상의 정수
- slug - 아스키 문자나 숫자, 하이픈, 언더스코어 등으로 이루어진 slug string
    - ex. building-your-1st-django-site
- uuid - UUID
- path - '/'를 포함한 비어있지 않은 문자열
    - str은 부분 URL path
    - path는 완전한 URL path
  
</details>
<br/>