---
title: Django
---
# ORM & DB
## migration 파일들 초기화시키기
```sh
find . -path "*/migrations/*.py" -not -name "__init__.py" -delete  
find . -path "*/migrations/*.pyc" -delete
```

## Indexing

## M2M models
- many to many 관계의 모델은 매개 테이블을 만들어야 한다
- 예시로 Author, Book, 그리고 그 매개 테이블인 Authorship을 예시로 들어보자
```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=100)
    authors = models.ManyToManyField(Author, through="Authorship", related_name="books")

class Authorship(models.Model):
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    book = models.ForeignKey(Book, on_delete=models.CASCADE)
    contribution = models.CharField(max_length=100)
    date_joined = models.DateField()
```
- **through** 키워드를 사용하여 매개 테이블로 어떤 모델을 사용할지 지정할 수 있다. 이 예시에서는 Authorship 모델을 매개 테이블로 사용하고 있다.
- 또한, **related_name**을 통해 반대 방향의 관계(reversed relationship)에 사용할 이름을 설정할 수 있다. 여기서는 Author 모델에서 관련된 Book들을 조회할 때 author.books.all()처럼 books라는 이름을 사용할 수 있다.
## query 최적화하기
author의 book을 모두 받아올 때 최적화를 어떻게 할까?
prefetch_related를 사용하면 되지만 3개의 테이블을 사용하기 때문에 쿼리는 2개가 최소가 된다
```python
author = Author.objects.prefetch_related("books").get(id=1)
boods = author.books.all()
```
## signal 사용하는 방법
Django에서는 **Many-to-Many (M2M) 관계**에서 특정 이벤트가 발생할 때 신호(signal)를 사용하여 추가적인 작업을 수행할 수 있습니다. M2M 관계에 대해 신호를 사용하는 대표적인 경우는 관계가 추가되거나 제거될 때입니다. Django에서 M2M 필드에 대한 변화를 감지하려면 m2m_changed 신호를 사용합니다.

**m2m_changed 신호 사용 방법**

**1. 기본 구조**
m2m_changed 신호는 M2M 필드가 변경될 때 실행됩니다. 이 신호는 관계가 **추가되거나**, **제거되거나**, **초기화될 때** 실행됩니다.

**2. m2m_changed 신호 연결**
먼저, 신호를 연결하기 위해 Django의 m2m_changed 모듈을 import하고, 신호를 받을 함수를 정의합니다.

```python
from django.db.models.signals import m2m_changed
from django.dispatch import receiver
from .models import Book

# 신호 수신기 정의
@receiver(m2m_changed, sender=Book.authors.through)
def authors_changed(sender, instance, action, reverse, model, pk_set, **kwargs):
    """
    m2m_changed 시그널 수신기 함수
    :param sender: 신호를 보낸 모델
    :param instance: Book 인스턴스 (Book 객체)
    :param action: add, remove, clear 등의 액션
    :param reverse: 반대 방향에서 호출되는지 여부
    :param model: 연결된 모델 (Author 모델)
    :param pk_set: 변경된 관련 객체의 기본 키(primary key) 세트
    :param kwargs: 기타 추가 정보
    """
    if action == "post_add":
        print(f"Authors have been added to {instance.title}.")
        print(f"Added authors: {pk_set}")

    elif action == "post_remove":
        print(f"Authors have been removed from {instance.title}.")
        print(f"Removed authors: {pk_set}")

    elif action == "post_clear":
        print(f"All authors have been cleared from {instance.title}.")
```  


**3. 신호 파라미터 설명**
 - sender: 신호를 보낸 M2M 필드의 중간 모델 (through 테이블). 여기서는 Book.authors.through가 됩니다.
 - instance: 관계가 변경된 Book 객체.
 - action: 발생한 이벤트 유형으로, "pre_add", "post_add", "pre_remove", "post_remove", "pre_clear", "post_clear" 값 중 하나입니다.
 - "pre_add": 관계가 추가되기 전에 호출.
 - "post_add": 관계가 추가된 후 호출.
 - "pre_remove": 관계가 제거되기 전에 호출.
 - "post_remove": 관계가 제거된 후 호출.
 - "pre_clear": 관계가 초기화되기 전에 호출.
 - "post_clear": 관계가 초기화된 후 호출.
 - reverse: 신호가 reverse 관계에서 발생하는지 여부. 보통 기본 관계에서 발생하면 False, 반대 방향에서는 True.
 - model: 연결된 모델 (여기서는 Author 모델).
 - pk_set: 추가되거나 제거된 객체의 기본 키 세트 (set 객체).
 - kwargs: 기타 신호에 전달되는 추가 정보.

**4. 신호 작동 예시**
```python
# Author와 Book 모델 정의
author1 = Author.objects.create(name="Author 1")
author2 = Author.objects.create(name="Author 2")
book = Book.objects.create(title="Example Book")
# ManyToMany 관계 추가
book.authors.add(author1, author2)
# 출력: Authors have been added to Example Book.
#       Added authors: {1, 2}
# 관계 제거
book.authors.remove(author1)
# 출력: Authors have been removed from Example Book.
#       Removed authors: {1}
# 관계 초기화 (모든 관계 제거)
book.authors.clear()
# 출력: All authors have been cleared from Example Book.
```

**5. 주요 포인트**
 - **sender**는 ManyToMany 필드에서 사용하는 중간 테이블입니다. 이 예시에서는 Book.authors.through가 그 역할을 합니다.
 - action 값을 사용하여 관계가 추가되는지, 제거되는지, 초기화되는지를 파악할 수 있습니다.
 - 신호는 **관계가 변경된 후**(post_add, post_remove, post_clear) 또는 **변경되기 전**(pre_add, pre_remove, pre_clear)에 각각 실행될 수 있습니다.

**6. 주의사항**
- m2m_changed는 관계의 변경 사항을 실시간으로 감지하지만, 객체를 직접 변경하는 것이 아닌, 관계 자체의 변화를 추적합니다.
- 성능 상의 이유로 **큰 데이터셋**에서는 조심스럽게 사용해야 합니다. 예를 들어, 한 번에 너무 많은 객체를 add하거나 remove하면 신호가 여러 번 트리거될 수 있습니다.

# WSGI
## gunicorn
### worker 설정하는 방법
```
gunicorn work
```

### gunicorn 프로세스 끄는 방법
```sh
pkill gunicorn
```
**더 확실하게 끄는 방법**
```sh
ps aux | grep gunicorn | grep -v grep | awk '{print $2}' | xargs kill -9
```
# celery
## Installation
[[설치하는 법|celery 설치하기]]
## Usage

