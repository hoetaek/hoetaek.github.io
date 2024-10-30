---
title: PHP
tags:
---
# Syntax
## Dynamic method calls
https://medium.com/@erlandmuchasaj/php-dynamic-method-calling-3c5dfbe816a2


## anonymous classses
```php
public function test_somthing(): void
{
    $dto = new class(
        ['arrayOfInts' => [1, 2]]
    ) extends DataTransferObject {
        /** @var int[] */
        public array $arrayOfInts;
	};
}
```
# Data type
## WeakMap
- 참조가 사라지면 가비티 콜렉팅이 되도록 하는 map
```php
class EntityCache
{
    public function __construct(
	    private WeakMap $cache
	) {}

	public function getSomethingWithCaching(object $object): object
	{
		if (! isset($this->cache[$object])){
			$this->cache[$object] = $this
				->computeSomethingExpensive($object);
		}
		
		return $this->cache[$object];
	
	}
}
```
## Array
### append

```php
$array = array("apple", "banana", "cherry");
$array[] = "date"; // Appending "date" to the array

print_r($array);
```
## String

### 변수값을 활용한 string
- 쌍따옴표를 사용하고 ${}안에 별수를 넣는다
```php
"put a string ${variable} inside"
```
