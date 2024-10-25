---
title: Laravel
---
# Database
## CLI
### migrate 관련 명령어
- step 옵션을 주지 않으면 모든 마이그레이션 파일에 대해 실행된다
```sh
php artisan migrate
php artisan migrate:rollback --step=1
```

- 새로 다시 마이그레이션하기
```sh
php artisan migrate:refresh
php artisan migrate:reset
```

> [!warning] refresh와 reset의 차이
> refresh는 모든 migration을 rollback한 후 다시 실행
> reset은 모든 migration을 rollback

- db 채우기
```sh
php artisan db:seed
```

## Migration file
- dropForeign을 사용하면 외래키 설정만 제거할 수 있다
예시: `$table->dropForeign(['user_id']);`
