---
date: 2025-05-10
publish: true
tags:
---
Pandas는 Python에서 데이터 분석을 위한 강력한 라이브러리입니다. DataFrame은 Pandas의 핵심 데이터 구조 중 하나로, 엑셀 시트나 SQL 테이블과 유사한 2차원 테이블 형태의 데이터를 다룰 때 매우 유용합니다. DataFrame에서 특정 열들만 선택하여 새로운 DataFrame을 만드는 것은 데이터 분석 과정에서 매우 흔한 작업입니다. 이 아티클에서는 다양한 방법을 사용하여 원하는 열을 선택하는 방법을 알아보겠습니다.

### 1. 대괄호(`[]`)와 열 이름 리스트 사용

가장 기본적인 방법은 대괄호 안에 선택하고 싶은 열 이름들의 리스트를 전달하는 것입니다.

```
import pandas as pd

# 예시 DataFrame 생성
data = {'col1': [1, 2, 3, 4],
        'col2': ['A', 'B', 'C', 'D'],
        'col3': [10.1, 11.2, 12.3, 13.4],
        'col4': [True, False, True, False]}
df = pd.DataFrame(data)

print("원본 DataFrame:")
print(df)
print("-" * 30)

# 'col1'과 'col3' 선택
selected_cols_df = df[['col1', 'col3']]
print("선택된 열 DataFrame ('col1', 'col3'):")
print(selected_cols_df)
```

**설명:**

- `df[['col1', 'col3']]`와 같이 대괄호 안에 또 다른 대괄호로 묶인 리스트 형태로 열 이름을 전달하면, 해당 열들로 구성된 새로운 DataFrame이 반환됩니다.
    
- 만약 하나의 열만 선택하고 싶다면, `df['col1']`과 같이 리스트 없이 열 이름만 전달할 수 있습니다. 이 경우 Pandas Series 객체가 반환됩니다. DataFrame 형태로 반환받고 싶다면 `df[['col1']]`과 같이 리스트 형태로 전달해야 합니다.
    

### 2. `loc` 인덱서 사용

`loc` 인덱서는 레이블 기반의 인덱싱을 제공합니다. 행과 열을 동시에 선택할 때 유용하며, 열만 선택할 때는 모든 행을 선택한다는 의미로 `:`를 사용합니다.

```
import pandas as pd

# 예시 DataFrame 생성 (위와 동일)
data = {'col1': [1, 2, 3, 4],
        'col2': ['A', 'B', 'C', 'D'],
        'col3': [10.1, 11.2, 12.3, 13.4],
        'col4': [True, False, True, False]}
df = pd.DataFrame(data)

print("원본 DataFrame:")
print(df)
print("-" * 30)

# loc를 사용하여 'col2'와 'col4' 선택
selected_cols_loc_df = df.loc[:, ['col2', 'col4']]
print("loc로 선택된 열 DataFrame ('col2', 'col4'):")
print(selected_cols_loc_df)

# loc를 사용하여 단일 열 'col1' 선택 (DataFrame 형태로 반환)
single_col_loc_df = df.loc[:, ['col1']]
print("\nloc로 선택된 단일 열 DataFrame ('col1'):")
print(single_col_loc_df)

# loc를 사용하여 단일 열 'col1' 선택 (Series 형태로 반환)
single_col_loc_series = df.loc[:, 'col1']
print("\nloc로 선택된 단일 열 Series ('col1'):")
print(single_col_loc_series)
```

**설명:**

- `df.loc[:, ['col2', 'col4']]`에서 `:`는 모든 행을 의미하고, `['col2', 'col4']`는 선택할 열 이름의 리스트입니다.
    
- `loc`를 사용할 때도 단일 열을 선택하면 기본적으로 Series가 반환되지만, 열 이름을 리스트로 감싸면 DataFrame으로 반환됩니다.
    

### 3. `iloc` 인덱서 사용 (정수 위치 기반)

`iloc` 인덱서는 정수 위치 기반의 인덱싱을 제공합니다. 열의 이름 대신 열의 순서(0부터 시작)를 사용하여 선택합니다.

```
import pandas as pd

# 예시 DataFrame 생성 (위와 동일)
data = {'col1': [1, 2, 3, 4],        # 0번째 열
        'col2': ['A', 'B', 'C', 'D'], # 1번째 열
        'col3': [10.1, 11.2, 12.3, 13.4],# 2번째 열
        'col4': [True, False, True, False]}# 3번째 열
df = pd.DataFrame(data)

print("원본 DataFrame:")
print(df)
print("-" * 30)

# iloc를 사용하여 0번째와 2번째 열 선택 ('col1', 'col3')
selected_cols_iloc_df = df.iloc[:, [0, 2]]
print("iloc로 선택된 열 DataFrame (0번째, 2번째):")
print(selected_cols_iloc_df)

# iloc를 사용하여 0번째부터 2번째 전까지의 열 선택 (0번째, 1번째 열)
selected_cols_slice_iloc_df = df.iloc[:, 0:2] # 0, 1 인덱스 선택
print("\niloc 슬라이싱으로 선택된 열 DataFrame (0번째, 1번째):")
print(selected_cols_slice_iloc_df)
```

**설명:**

- `df.iloc[:, [0, 2]]`는 모든 행과 0번째, 2번째 위치의 열을 선택합니다.
    
- `df.iloc[:, 0:2]`와 같이 슬라이싱을 사용하여 연속된 범위의 열을 선택할 수도 있습니다. 이 경우 0번째부터 2번째 _이전_까지, 즉 0번째와 1번째 열이 선택됩니다.
    

### 4. `filter()` 메소드 사용

`filter()` 메소드는 특정 조건에 맞는 열 이름을 선택할 때 유용합니다. `items`, `like`, `regex` 등의 매개변수를 사용할 수 있습니다.

```
import pandas as pd

# 예시 DataFrame 생성
data = {'ID_A': [1, 2, 3],
        'Value_A': [10, 20, 30],
        'ID_B': [4, 5, 6],
        'Value_B': [40, 50, 60],
        'Misc': [7, 8, 9]}
df = pd.DataFrame(data)

print("원본 DataFrame:")
print(df)
print("-" * 30)

# items 매개변수: 정확한 열 이름 리스트로 선택
selected_cols_filter_items = df.filter(items=['ID_A', 'Value_B'])
print("filter(items=...)로 선택된 열:")
print(selected_cols_filter_items)
print("-" * 30)

# like 매개변수: 특정 문자열이 포함된 열 선택
selected_cols_filter_like = df.filter(like='ID')
print("filter(like='ID')로 선택된 열:")
print(selected_cols_filter_like)
print("-" * 30)

# regex 매개변수: 정규 표현식에 맞는 열 선택
# 'Value'로 시작하는 열 선택
selected_cols_filter_regex = df.filter(regex='^Value')
print("filter(regex='^Value')로 선택된 열:")
print(selected_cols_filter_regex)
```

**설명:**

- `df.filter(items=['ID_A', 'Value_B'])`: `items` 매개변수는 대괄호 방식과 유사하게 정확한 열 이름 리스트를 받아 선택합니다.
    
- `df.filter(like='ID')`: `like` 매개변수는 'ID'라는 문자열이 열 이름에 포함된 모든 열을 선택합니다.
    
- `df.filter(regex='^Value')`: `regex` 매개변수는 정규 표현식을 사용하여 'Value'로 시작하는 모든 열을 선택합니다.
    

### 요약

|   |   |   |
|---|---|---|
|**방법**|**사용 예시**|**설명**|
|대괄호 (`[]`)|`df[['colA', 'colB']]`|가장 간단하고 직관적인 방법.|
|`loc` 인덱서|`df.loc[:, ['colA', 'colB']]`|레이블 기반 선택. 행과 열을 동시에 지정할 때 유용.|
|`iloc` 인덱서|`df.iloc[:, [0, 1]]`|정수 위치 기반 선택. 열의 순서를 알 때 사용.|
|`filter()` 메소드|`df.filter(like='prefix')`|특정 패턴(문자열 포함, 정규식 등)에 맞는 열을 선택할 때 유용.|

어떤 방법을 사용할지는 상황과 개인의 선호도에 따라 다릅니다. 단순하게 몇 개의 열을 이름으로 선택할 때는 대괄호 방식이 편리하고, 조건에 따라 동적으로 열을 선택해야 할 때는 `filter`나 `loc`와 함께 조건식을 사용하는 것이 효과적일 수 있습니다. `iloc`은 열의 순서가 중요하거나 이름이 너무 길 때 유용하게 사용될 수 있습니다.

이 가이드가 Pandas DataFrame에서 원하는 열을 선택하는 데 도움이 되기를 바랍니다!