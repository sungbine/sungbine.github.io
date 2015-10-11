# CH4. Django의 핵심기능
##4.1 Admin 사이트 꾸미기
장고의 admin 사이트는 데이터베이스에 들어 있는 데이터를 쉽게 관리 할 수 있도록 데이터의 생성/조회/변경/삭제 등의 기능을 제공한다. ==프로세스의 기동/정지/조회 등의 프로세스 관리기능은 제공하지 않는다!==

###4.1.1 데이터 입력 및 수정
레코드 상세보기 화면에서는 

* Save
* Save and continue editing
* Save and add another
* Delete
* History

기능을 제공하고 있다.(설명은 생략)

###4.1.2 필드 순서변경
필드 순서를 변경하고 싶다면 admin.py

```
class QuestionAdmin(admin.ModelAdmin):
	field = ['pub_date', 'question_text'] #필드 순서변경

admin.site.register(Question, QuestionnAdmin)
admin.site.register(Choice)
```
위처럼 QuestionAdmin 클래스를 ModelAmdin을 상속받아 정의해, field를 생성하고 admin.site.register()함수의 두 번째 인자로 등록하면 순서변경이 가능하다.


###4.1.3 각 필드를 분리해서 보여주기
필드를 분리해서 보여 주는 경우

```
class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		('Question Statemnet', {'fields' : ['question_text']}),
		('Date Information', {'fields' : ['pub_date']}),
		]
```

위처럼 하면 admin 화면에서 두개 의 필드를 다른 셋처럼 보여준다.

###4.1.4 필드 접기
```
class QuestionAdmin(admin.ModelAdmin):
	fieldsets = [
		('Question Statemnet', {'fields': ['question_text'], 'classes': ['collapse']}),
		('Date Information', {'fields': ['pub_date']}),
	]
```

###4.1.5 외래키 관계화면
외래키를 가진 Choice 모델의 경우 항목 추가시 Question을 선택해야만 항목을 선택할 수 있다. 항목이 많아질 경우 매우 번거로워 지는문제가 발생한다.

###4.1.6 Question 및 Choice를 한 화면에서 변경하기


```
class ChoiceInline(admin.StackedInline):
    model = Choice
    extra = 2

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {'fields': ['question_text'], 'classes': ['collapse']}),
        ('Date Information', {'fields': ['pub_date']}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
admin.site.register(Choice)
```
위처럼 변경하면 하나의 화면에서 두개의 모델을 한번에 편집 가능하다.
extra 변수는 추가될 항목을 미리 2개까지 작성해둔다라는 뜻이다.


###4.1.7 테이블 형식으로 보여주기
항목이 많아질 경우에는 화면이 길어져서 불편할 수 있다.

```
class ChoiceInline(admin.TabularInline)
```

으로 변경하면 테이블 형식으로 보여준다.


###4.1.8 레코드 리스트 항목 지정하기
Admin사이트의 첫 페이지에서 테이블명을 클릭하면, 해등 테이블의 레코드 리스트가 나온다.
여기에 나오는 레코드 리스트의 제목은 model.py 에서 지정했던 \_\_unicode\_\_ 함수에 정의된 내용이 나타난다.
레코드 리스트에 나타나는 추가항목을 지정하고 싶다면 아래처럼하면 된다.

```
inlines = [ChoiceInline]
list_display = ('question_text', 'pub_date')
```

QuestionAdmin 클래스에 위 한줄을 추가하면 두개의 컬럼을 확인 할 수 있다.



###4.1.9 list_filter

```
list_filter = ['pub_date']

```
를 추가하면 필터링 기능까지 추가된다.(pub_date은 date타입이기 때문에 any date, today 등의 필터링 옵션을 제공한다.)


###4.1.10 search_fields
```
search_fields = ['question_text']
```

를 지정하면 검색할 수 있는 필드를 제공한다. (박스에 단어를 입력하면 장고가 LIKE 쿼리를 이용해 해당 필드에서 검색한다. 여러개 지정할 수 있다.)


###4.1.11 Admin 사이트 템플릿 수정
Admin 사이트 템플릿은 장고의 템플릿 시스템을 사용한다.
Admin 사이트 모양을 변경하기 위해서는 장고의 기본 Admin 템플릿 파일을 프로젝트로 복사해야한다.
새로운 템플릿 디렉토리를 생성하고 settings.py에 등록해야한다.
`
 python -c "import sys; sys.path = sys.path[1:]; import django; print django.__path__"
 `
 위 명령을 실행하면 장고 위치를 알 소 있다.
 
 그리고 django/contrib/admin/templates/admin/base_site.html 을 복사해 우리가 만든 templates/admin 폴더에 넣고 setting.py 에 등록해아한다.==(꼭 admin폴더에 넣어야한다!!)==
 
 하지만
 `TEMPLATE_DIRS = [os.path.join(BASE_DIR, 'templates')]` 책에 나온 대로 추가하면 (1.8버전 이상의 장고에서는) 아래 그림과 같은 에러가 발생한다.
 
이땐 

```
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    }
]
```
 위와 같이 바뀐 설정을 해줘야한다. 하나의 속성이라도 없어지게 되면 에러가 발생한다.
 
이 템플릿 파일을 이용하면 내가 원하는 admin 화면을 꾸밀 수 있다.


##4.2 장고 파이썬 쉘로 데이터 조작하기
Admin 사이트는 장고의 장점중 하나가 바로 데이터의 관리가 편리하다. 장고는 추가적으로 파이썬 쉘을 이용하여 데이터를 관리 할 수 있는 API도 제공하고 있다.

**사용하는 이유**

* 인터넷이 느려 admin 접속이 어려울 때
* 복잡한 조건의 검색하는 것처럼 Admin 사이트보다 더 다양한 데이터 관리 명령이 가능


**쉘 시작**

`$ python manage.py shell` 

* 위의 명령을 이용하면 장고 파이썬 쉘을 실행
* 위 명령역시 파이썬의 쉘을 기동하는 것과 동일
* 차이점은 장고 파이썬 쉘은 manage.py 모듈에서 정의한 DJANGO\_SETTINGS\_MODULE 속성을 이용해 미리 mysite/settings.py 모듈을 임포트 하는 것

**장고의 클래스는 곧 테이블이고 객체가 곧 레코드 이다.**


###4.2.1 Create - 데이터 생성/입력

* 객체 생성  > save()
* save() 명령을 실행하기 전에는 메모리만 반영된 상태
* 내부적으로 INSERT문을 실행한다.

###4.2.2 Read - 데이터 조회
* QuerySet 객체를 이용
* QuerySet은 데이터베이스 테이블로부터 꺼내 온 객체의 collection
* QuerySet은 필터를 가질 수 있음(조건을 지정해 검색/추출 할 수있음)
* SQL 용어도 QuerySet 은 SELECT문에 해당, 필터는 WHERE, LIMIT에 해당
* object객 체를 이용해야 QuerySet을 얻을 수 있음

`Question.objects.all()` 은 `Question테이블.레코드.모두` 라고 해석 할 수있다.

* filter() : 주어진 조건에 맞는 객체들을 담고 있는 QuerySet collection 반환
* exclude() : 주어진 조건에 맞지 않는 객체들을 담고 있는 QuerySet collection 반환

```
#example
Question.objects.filter(
	quetion_text_startwith='What'
).exclude(
	pub_date_gte=datetime.date.today()
).filter(
	pub_date_gte=datetime(2005, 1, 30)
)
```

처럼 사용가능.

```
one_entry = Question.object.get(pk=1)
Question.obejct.all()[:5]    # 4번까지
Question.object.all()[5:10]  # 5부터 9번까지
Question.object.all()[:10:2] # 뭐지...........
```
 와 같은 파이썬 배열의 슬라이싱 문법(?) 사용가능
 
### 4.2.3 Update - 데이터 수정
* 필드값 수정 후  save() 메소드 호출
* SQL의 UPDATE에 해당

```
q.question_text = 'What is your favorite hobby ?'
q.save()
```

여러 개의 객체를 동시에 수정할 경우
```
Question.objects.filter(pub_date_year=2007).update(question_text='Everything is the same')
```

###4.2.4 - 데이터 삭제
```
Question.objects.filter(pub_date_year=2005).delete()
Question.objects.all().delete()
Question.objects.delete()  #사용불가
```
장고에서 세번째 명령은 허용하지 않는다.

###4.2.5 polls 애플리케이션의 데이터 실습
```
from pools.models import Question, Choice

#timezone은 settings.py에 타임존 셋팅이 잘 되어있어야 한다.

from django.utils import timezone 
q = Question(question_text = "what's up?", pub_date=timezone.now())
q.save()
```

```
from pools.models import Question, Choice

Question.objects.filter(id=1)
Question.objects.filter(question_text_startwith='what')

from django.utils import timezone
current_year = timezone.now().year
Question.objects.get(pub_date_year=current_year)

Question.objects.get(id=100) #exception 발생 존재하지 않는 데이터

Question.objects.get(id=1)
Question.objects.get(pk=1) #위 구문과 동일

q = Question.object.get(pk = 1)

#질문에 해당하는 답변 모두 조회
q.choice_set.all()

#create() 함수를 호출하면 Choice 개게를 생성해서 데이터베이스에 저장하고, 
#choice_set 리스트에 추가한 다음 해당 객체(choice)를 반환한다.
q.choice_set.create(choice_text=`Sleeping', votes=0)

#생성된 객체는 c 에 들어감
c = q.choice_set.create(choice_text=`Eating', votes=0)

#연결된 Question객체를 조회 할 수 있음
c.question

q.choice_set.all()
q.choice_set.count()


#언더바2개 __를 이용해 연결된 객체간의 관계를 표현 가능
Choice.objects.filter(question__pub_date__year=current_year)

#choice_set에서 하나만 삭제
c = q.choice_set.filter(choice_text__startswith='Reading')
c.delete();
```

##4.3 템플릿 시스템