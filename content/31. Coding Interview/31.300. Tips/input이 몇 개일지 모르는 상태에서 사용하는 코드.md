```python
import sys

lines = sys.stdin.readlines() 
for line in lines:
	A, B = map(int, line.split())
	print(A+B)
```

```python
while True:
    try:
        print(input())
	except EOFError:
		break
```
