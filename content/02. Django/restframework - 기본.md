---
title: Django Restframework
publish: false
tags:
---
# 1. Serializer 핵심 개념

## 1.1 기본 Serializer 패턴

```python
# 잘못된 방식 - 너무 많은 필드를 직접 정의
class UserSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    username = serializers.CharField()
    email = serializers.EmailField()
    date_joined = serializers.DateTimeField()
    # ... 수많은 필드들

# 올바른 방식 - ModelSerializer 활용
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'date_joined']
        read_only_fields = ['date_joined']
```

## 1.2 중첩 Serializer 처리

```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ['id', 'content']

class PostSerializer(serializers.ModelSerializer):
    # 잘못된 방식 - N+1 쿼리 문제 발생
    comments = CommentSerializer(many=True)

    # 올바른 방식 - select_related/prefetch_related 활용
    comments = serializers.SerializerMethodField()

    def get_comments(self, obj):
        comments = obj.comments.all()  # prefetch_related로 최적화 필요
        return CommentSerializer(comments, many=True).data

    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'comments']
```

## 1.3 Serializer 유효성 검사

```python
class PostSerializer(serializers.ModelSerializer):
    title = serializers.CharField(
        max_length=100,
        validators=[UniqueValidator(queryset=Post.objects.all())]
    )

    def validate_title(self, value):
        # 커스텀 필드 수준 유효성 검사
        if 'django' not in value.lower():
            raise serializers.ValidationError(
                "제목에 'django'가 포함되어야 합니다."
            )
        return value

    def validate(self, data):
        # 객체 수준 유효성 검사
        if len(data['content']) < len(data['title']):
            raise serializers.ValidationError(
                "내용은 제목보다 길어야 합니다."
            )
        return data
```

# 2. ViewSet과 Router

## 2.1 ViewSet 패턴

```python
# 기본 ViewSet 패턴
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    permission_classes = [IsAuthenticated]
    
    # 잘못된 방식 - 필터링 없는 쿼리셋
    # queryset = Post.objects.all()
    
    # 올바른 방식 - 동적 쿼리셋
    def get_queryset(self):
        return Post.objects.filter(
            author=self.request.user
        ).select_related('author').prefetch_related('comments')

    # 커스텀 액션 추가
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        post = self.get_object()
        post.publish()
        return Response({'status': 'published'})
```

## 2.2 Router 설정

```python
# urls.py
from rest_framework.routers import DefaultRouter
from django.urls import path, include

router = DefaultRouter()
router.register(r'posts', PostViewSet)
router.register(r'comments', CommentViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]
```

# 3. 인증과 권한

## 3.1 인증 설정

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

## 3.2 커스텀 권한 클래스

```python
from rest_framework import permissions

class IsAuthorOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        # 읽기 권한은 모든 요청에 허용
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # 쓰기 권한은 게시물 작성자에게만 허용
        return obj.author == request.user

# ViewSet에 적용
class PostViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated, IsAuthorOrReadOnly]
```

## 3.3 JWT 인증 구현

```python
# urls.py
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view()),
    path('api/token/refresh/', TokenRefreshView.as_view()),
]

# settings.py
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(hours=1),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'ROTATE_REFRESH_TOKENS': False,
    'BLACKLIST_AFTER_ROTATION': True,
}
```

# 4. 성능 최적화

## 4.1 쿼리 최적화

```python
class PostViewSet(viewsets.ModelViewSet):
    # 잘못된 방식
    queryset = Post.objects.all()
    
    # 올바른 방식 - 필요한 관계만 미리 로드
    def get_queryset(self):
        return Post.objects.select_related(
            'author', 
            'category'
        ).prefetch_related(
            'tags',
            Prefetch(
                'comments',
                queryset=Comment.objects.select_related('author')
            )
        )
```

## 4.2 캐싱 구현

```python
from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page

class PostViewSet(viewsets.ModelViewSet):
    @method_decorator(cache_page(60 * 15))  # 15분 캐시
    def list(self, *args, **kwargs):
        return super().list(*args, **kwargs)
```

# 5. 에러 처리

## 5.1 커스텀 예외 처리

```python
from rest_framework.exceptions import APIException
from rest_framework import status

class PostNotFound(APIException):
    status_code = status.HTTP_404_NOT_FOUND
    default_detail = '게시물을 찾을 수 없습니다.'
    default_code = 'post_not_found'

# 전역 예외 처리
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
    response = exception_handler(exc, context)
    
    if response is not None:
        response.data['status_code'] = response.status_code
        
    return response

# settings.py
REST_FRAMEWORK = {
    'EXCEPTION_HANDLER': 'myapp.utils.custom_exception_handler'
}
```

