---
publish:
---

# Installation
```sh
composer require spatie/laravel-event-sourcing

php artisan vendor:publish --provider="Spatie\EventSourcing\EventSourcingServiceProvider" --tag="event-sourcing-migrations"
php artisan migrate

php artisan vendor:publish --provider="Spatie\EventSourcing\EventSourcingServiceProvider" --tag="event-sourcing-config"
```

# Usage
### event 만들기 
```sh
php artisan make:storable-event
```
### agregate 만들기
```sh
php artisan make:aggregate
```
### projector 만들기
```sh
php artisan make:projector
```

([링크](https://spatie.be/docs/laravel-event-sourcing/v7/advanced-usage/preparing-events))