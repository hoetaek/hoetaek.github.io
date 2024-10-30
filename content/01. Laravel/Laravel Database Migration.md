---
title: Laravel Database Migration
publish: false
tags:
---
# 개요
Database Migration은 데이터베이스 스키마의 버전 관리 시스템입니다. Git이 코드를 관리하듯이, Migration은 데이터베이스 구조의 변경 사항을 추적하고 관리합니다.

# 기본 명령어

## Migration 실행하기
```bash
# 모든 미실행 마이그레이션 실행
php artisan migrate

# 특정 경로의 마이그레이션만 실행
php artisan migrate --path=/database/migrations/specific

# 특정 데이터베이스에 대해 실행
php artisan migrate --database=mysql2

# 프로덕션 환경에서 강제 실행
php artisan migrate --force
```

## Migration 되돌리기
```bash
# 마지막 마이그레이션 그룹 롤백
php artisan migrate:rollback

# 특정 단계만큼 롤백
php artisan migrate:rollback --step=2

# 모든 마이그레이션 롤백
php artisan migrate:reset

# 모든 마이그레이션 롤백 후 다시 실행
php artisan migrate:refresh

# 모든 테이블과 데이터 삭제 후 다시 마이그레이션
php artisan migrate:fresh
```

## Seeding 명령어
```bash
# 모든 시더 실행
php artisan db:seed

# 특정 시더만 실행
php artisan db:seed --class=UserSeeder

# 마이그레이션과 시딩 함께 실행
php artisan migrate --seed
php artisan migrate:refresh --seed
```

# Migration 파일 작성

## 기본 구조
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('users');
    }
};
```

## 자주 사용하는 컬럼 타입
```php
// 기본 컬럼
$table->id();                  // AUTO_INCREMENT primary key
$table->string('name');        // VARCHAR
$table->text('description');   // TEXT
$table->integer('count');      // INTEGER
$table->boolean('is_active');  // BOOLEAN
$table->timestamp('created_at'); // TIMESTAMP

// 외래 키
$table->foreignId('user_id')
      ->constrained()
      ->onDelete('cascade');
```

## 외래 키 제약조건 관리
```php
// 외래 키 추가
$table->foreign('user_id')
      ->references('id')
      ->on('users')
      ->onDelete('cascade');

// 외래 키 삭제
$table->dropForeign(['user_id']);

// 외래 키 이름 지정
$table->foreign('user_id', 'fk_user_id')
      ->references('id')
      ->on('users');
```

# 실전 사용 예시

## 1. 관계형 테이블 생성
```php
// create_posts_table.php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('content');
        $table->foreignId('user_id')->constrained();
        $table->timestamps();
        $table->softDeletes();
    });
}
```

## 2. 테이블 수정
```php
// add_status_to_posts_table.php
public function up()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->string('status')->default('draft')->after('content');
    });
}

public function down()
{
    Schema::table('posts', function (Blueprint $table) {
        $table->dropColumn('status');
    });
}
```

## 3. 복합 인덱스 추가
```php
Schema::table('posts', function (Blueprint $table) {
    $table->index(['user_id', 'created_at']);
});
```

# 주의사항

1. 운영 환경 마이그레이션
   - 항상 `down()` 메소드 구현
   - 데이터 손실 가능성 검토
   - 대용량 테이블 마이그레이션 시 성능 고려

2. 외래 키 제약조건
   ```php
   // 잘못된 예시
   $table->dropForeign('user_id'); // 오류 발생

   // 올바른 예시
   $table->dropForeign(['user_id']);
   ```

3. 컬럼 수정
   ```php
   // 길이 변경 시
   $table->string('name', 100)->change();

   // Nullable 설정 변경 시
   $table->string('name')->nullable()->change();
   ```

# Seeding 활용

## 기본 시더 작성
```php
// database/seeders/UserSeeder.php
public function run()
{
    DB::table('users')->insert([
        'name' => 'Admin User',
        'email' => 'admin@example.com',
        'password' => Hash::make('password'),
    ]);
}
```

## Factory 활용
```php
// database/factories/UserFactory.php
public function definition()
{
    return [
        'name' => fake()->name(),
        'email' => fake()->unique()->safeEmail(),
        'password' => Hash::make('password'),
    ];
}

// Seeder에서 사용
User::factory()->count(50)->create();
```

# 실무 Best Practices

1. 명명 규칙
   ```php
   // 테이블 생성
   create_users_table
   
   // 테이블 수정
   add_status_to_users_table
   update_name_field_in_users_table
   ```

2. 트랜잭션 활용
   ```php
   Schema::connection('mysql')->transaction(function () {
       Schema::create('table1', function (Blueprint $table) {
           // ...
       });
       
       Schema::create('table2', function (Blueprint $table) {
           // ...
       });
   });
   ```

3. 대용량 데이터 처리
   ```php
   // Chunk를 사용한 시딩
   User::chunk(1000, function ($users) {
       foreach ($users as $user) {
           // 처리 로직
       }
   });
   ```

# 결론

Laravel의 마이그레이션 시스템은 다음과 같은 이점을 제공합니다:
1. 데이터베이스 스키마 버전 관리
2. 팀 협업 환경에서의 일관성
3. 자동화된 데이터베이스 구조 관리
4. 테스트 환경 구축 용이성

이러한 기능들을 적절히 활용하여 안정적인 데이터베이스 관리가 가능합니다.