---
title: Django Restframework
publish: false
tags:
---
# 1. 필터링과 검색

## 1.1 django-filter 통합

```python
# settings.py
INSTALLED_APPS = [
    ...
    'django_filters',
]

REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ]
}

# filters.py
import django_filters
from .models import Post

class PostFilter(django_filters.FilterSet):
    title = django_filters.CharFilter(lookup_expr='icontains')
    created_at = django_filters.DateTimeFromToRangeFilter()
    category = django_filters.NumberFilter(field_name='category__id')

    class Meta:
        model = Post
        fields = ['title', 'created_at', 'category', 'status']

# views.py
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    filterset_class = PostFilter
    search_fields = ['title', 'content']
    ordering_fields = ['created_at', 'title']
```

## 1.2 커스텀 필터 백엔드

```python
from rest_framework import filters

class IsOwnerFilterBackend(filters.BaseFilterBackend):
    def filter_queryset(self, request, queryset, view):
        return queryset.filter(author=request.user)

class PostViewSet(viewsets.ModelViewSet):
    filter_backends = [
        IsOwnerFilterBackend,
        filters.SearchFilter,
        filters.OrderingFilter
    ]
```

# 2. 페이지네이션

## 2.1 페이지네이션 설정

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 
        'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}

# Custom Pagination
from rest_framework.pagination import PageNumberPagination

class CustomPageNumberPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'size'
    max_page_size = 100
    
    def get_paginated_response(self, data):
        return Response({
            'links': {
                'next': self.get_next_link(),
                'previous': self.get_previous_link()
            },
            'count': self.page.paginator.count,
            'total_pages': self.page.paginator.num_pages,
            'current_page': self.page.number,
            'results': data
        })

class PostViewSet(viewsets.ModelViewSet):
    pagination_class = CustomPageNumberPagination
```

## 2.2 커서 페이지네이션

```python
from rest_framework.pagination import CursorPagination

class PostCursorPagination(CursorPagination):
    ordering = '-created_at'
    page_size = 10
    
class PostViewSet(viewsets.ModelViewSet):
    pagination_class = PostCursorPagination
```

# 3. 응답 커스터마이징

## 3.1 커스텀 렌더러

```python
from rest_framework.renderers import JSONRenderer

class CustomJSONRenderer(JSONRenderer):
    def render(self, data, accepted_media_type=None, renderer_context=None):
        if renderer_context is not None:
            response = renderer_context['response']
            
            # 응답 데이터 래핑
            wrapped_data = {
                'status': 'success' if response.status_code < 400 else 'error',
                'code': response.status_code,
                'data': data
            }
            
            # 에러 응답 처리
            if response.status_code >= 400:
                wrapped_data['error'] = data
                wrapped_data['data'] = None
                
            return super().render(wrapped_data, accepted_media_type, 
                                renderer_context)
```

## 3.2 응답 미들웨어

```python
# middleware.py
class APIResponseMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        
        if hasattr(response, 'data') and isinstance(response.data, dict):
            response.data = {
                'status': 'success',
                'data': response.data
            }
            
        return response

# settings.py
MIDDLEWARE = [
    ...
    'myapp.middleware.APIResponseMiddleware',
]
```

# 4. 비동기 처리

## 4.1 비동기 뷰

```python
from rest_framework.decorators import action
from rest_framework.response import Response
import asyncio
import aiohttp

class PostViewSet(viewsets.ModelViewSet):
    @action(detail=False, methods=['get'])
    async def async_action(self, request):
        async with aiohttp.ClientSession() as session:
            async with session.get('https://api.example.com/data') as resp:
                data = await resp.json()