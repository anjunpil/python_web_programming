## Django

장점 - 제공하는 기능이 풍부, 쉽고 빠르게 웹 개발이 가능

#### 3.1 일반적인 특징

장고 웹 프레임워크의 주요 기능별 특징

- MVC 패턴 기반 MVT 
  - Model - 데이터베이스에 엑세스하는 컴포넌트
  - View - 데이터를 가져오고 변형하는 컴포넌트
  - Template - 데이터를 사용자에게 보여주는 컴포넌트

- 객체관계 매핑(ORM)
  - ORM 기능을 통해 DB 시스템지원, SQL문장 사용 않고도 테이블 조작 ok
  - 설정을 조금만 변경하면 다른 DB로도 변경 가능!

- 자동으로 구성되는 관리자 화면
- 직관적이고 쉬운 URL 방식
- 자체 템플릿 시스템 - 디자인 로직에 대한 코딩을 분리하여 독립적으로 개발 진행
- 캐시 시스템  - 캐시용 페이지 메모리,DB 내부, 파일 시스템 중 아무 곳에나 저장 가능

#### 3.3 장고에서의 애플리케이션 개발 방식

MVC - model-view-contrioller , 데이터-사용자인터페이스-데이터 처리하는 로직

![MVT 패턴](https://t1.daumcdn.net/cfile/tistory/99B782465BB8D0C61A)

- 클라이언트로부터 요청을 받으면 URLconf를 이용하여 URL을 분석
- URL분석 결과를 통해 해당 URL에 대한 처리를 담당할 뷰를 결정
- 뷰는 자신의 로직을 실행하면서, 만일 DB처리가 필요하면 모델을 통해 처리하고 그 결과를 반환받음
- 뷰는 자신의 로직 처리가 끝나면 템플릿을 사용해 클라이언트에 전송할 HTML 파일 생성
- 뷰는 최종 결과로 HTML 파일을 클라이언트에게 보내 응답 

#### 3.3.2 Model -DB의 정의

모델이란 사용될 데이터의 대한 정의를 담고 있는 클래스

ORM 기법을 사용하여 애플리케이션에서 사용할 데이터베이스를 클래스로 매핑해서 코딩이 가능

즉 하나의 모델 클래스 -> 하나의 테이블에 매핑

모델 클래스 속성 -> 테이블의 컬럼에 매핑

ORM 기법을 사용하여 애플리케이션에서는 DB에 대한 엑세스를 SQL없이도 클래스를 다루는 것처럼 할 수 있어 편리

ORM을 통한 API는 변경할 필요가 없어서 DB엔진을 쉽게 변경할 수 있다

- ORM(object-relatational-mappin) - 기존 웹 프로그래밍에서 db에 접근하려면 직접 SQL 언어를 사용해 데이터를 요청해야 했고, 개발자는 SQL 및 데이터베이스에 접근하기 위한 드라이버 API를 잘 알고 있어야 했음

  하지만 ORM에서는 DB대신에 객체(class)를 사용해 데이터를 처리할 수 있음

  객체 대상으로 자업 실행 -> ORM이 자동으로 적절한 SQL구문 or API를 호출

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length =30)
```

내부적으로 SQL명령을 사용하여 DB 테이블을 생성

- 테이블명은 애플리케이션명과 모델 클래스명을 밑줄(_)로 연결
- PK는 `person` 클래스에서 정의하지 않아도 자동으로 부여, 직접 지정 가능

#### 3.3.3 URLconf - URL 정의

- URLconf - URL/뷰 매핑을 뜻함

```python
from django.urls import path

from. import views

urlpatterns = [
    path('articles/2003/',views.special_case_2003),
    path('articles/<int:year>/',views.year_archive),
    path('articles/<int:year>/<int:month>/',views.month_archive)
    path('articles/<int:year>/<int:month>/<slug:slug>',views.article_detail),
]
```

`articles/2003` URL 부분, `views.special_case_2003` 처리함수(view)

- setting.py 파일의 ROOT_URLCONF 항목을 읽어 최상위 URLconf(urls.py) 위치를 알아냄
- URLconf를 로딩 urlpatterns 변수에 지정되어 있는 URL리스트를 검사
- URL 패턴이 매치되면 검사 종료
- 매치된 URL의 뷰를 호출 , 매칭에 실패하면 에러를 처리하는 뷰를 호출

`<int:year>`  꺾쇠부분은 URL패턴의 일부 문자열을 추출하기 위함 ex)`<type:name>`

- Path Converter - <>(꺾쇠 부분)
  - str :  `/`를 제외한 모든 문자열과 매치
  - int : 0 or 양의 정수와 매치
  - slug : slug 형식의 문자열(ASCII,숫자,하이픈,밑줄)과 매치
  - uuid : UUID 형식의 문자열과 매치
  - path : `/`를 포함한 모든 문자열과 매치, 일부가 아닌 전체를 추출하고자 할 때 사용

path() 함수를 사용을 하다, URL을 정교하게 정의할 때 re_path(),정규표현식을 사용

![정규표현식 문자들](https://t1.daumcdn.net/cfile/tistory/99B44A375A9BD8FD10)

#### 3.3.4 View - 로직 정의

웹 요청에 있는 URL을 분석, 그 결과로 해당 URL에 매핑된 뷰를 호출

해당 애플리케이션 로직에 맞는 처리 후 , 결과 데이터를 HTML로 변환하기 위하여 템플릿 처리 후 웹 클라이언트로 반환

```python
from django.http import HttpResponse
import datetime

def current_datetime(request):
    now = datetime.datetime.now()
    html = "<html><body>It is now %s.</body></html>" %now
    return HttpResponse(html)
```

- 필요한 처리 후에 HttpResponse 객체를 반환

#### 3.3.5 Template - 화면 UI 정의

HTML 텍스트, 장고는 이를 해석해 최송 HTMl 텍스트 응답을 생성,이를 클라이언트에게 보내줌

템플릿 - *.html 파일 ,UI 모습을 템플릿 문법에 맞게 작성

유의점 - 템플릿 파일을 적절한 디렉토리에 위치

#### 3.3.6 MVT 코딩 순서

모델 - 뷰 - 템플릿 순, 하지만 굳이 순서 필요 x,

