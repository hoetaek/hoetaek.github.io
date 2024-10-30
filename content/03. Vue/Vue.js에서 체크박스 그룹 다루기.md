---
publish: true
tags:
---
체크박스 그룹을 만들고 선택된 값들을 배열로 관리하는 방법을 알아보자.

# 기본 사용법

```javascript
<template>
  <div>
    <div v-for="item in items" :key="item.id">
      <input 
        type="checkbox"
        :value="item.value"
        v-model="selectedItems"
      >
      <label>{{ item.label }}</label>
    </div>
    
    <p>선택된 항목: {{ selectedItems }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const items = ref([
  { id: 1, value: 'apple', label: '사과' },
  { id: 2, value: 'banana', label: '바나나' },
  { id: 3, value: 'orange', label: '오렌지' }
])

const selectedItems = ref([]) // 선택된 값들이 이 배열에 저장됨
</script>
```

# 고급 활용

## 1. 전체 선택 기능
```javascript
<template>
  <div>
    <!-- 전체 선택 체크박스 -->
    <div>
      <input 
        type="checkbox"
        :checked="isAllSelected"
        @change="toggleAll"
      >
      <label>전체 선택</label>
    </div>

    <!-- 개별 체크박스 -->
    <div v-for="item in items" :key="item.id">
      <input 
        type="checkbox"
        :value="item.value"
        v-model="selectedItems"
      >
      <label>{{ item.label }}</label>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const items = ref([
  { id: 1, value: 'apple', label: '사과' },
  { id: 2, value: 'banana', label: '바나나' },
  { id: 3, value: 'orange', label: '오렌지' }
])

const selectedItems = ref([])

const isAllSelected = computed(() => 
  items.value.length === selectedItems.value.length
)

const toggleAll = () => {
  selectedItems.value = isAllSelected.value 
    ? [] 
    : items.value.map(item => item.value)
}
</script>
```

## 2. 비활성화 상태 관리
```javascript
<template>
  <div v-for="item in items" :key="item.id">
    <input 
      type="checkbox"
      :value="item.value"
      v-model="selectedItems"
      :disabled="isItemDisabled(item)"
    >
    <label :class="{ 'text-gray-400': isItemDisabled(item) }">
      {{ item.label }}
    </label>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const items = ref([
  { id: 1, value: 'apple', label: '사과', status: 'active' },
  { id: 2, value: 'banana', label: '바나나', status: 'inactive' },
  { id: 3, value: 'orange', label: '오렌지', status: 'active' }
])

const selectedItems = ref([])

const isItemDisabled = (item) => item.status === 'inactive'
</script>
```

## 3. 선택 항목 감시 및 처리
```javascript
<script setup>
import { ref, watch } from 'vue'
import { updateSelections } from '@/api/selections'

const selectedItems = ref([])

watch(selectedItems, async (newVal) => {
  try {
    await updateSelections(newVal)
  } catch (error) {
    console.error('선택 항목 업데이트 실패:', error)
  }
})
</script>
```

# 실제 활용 예제: 권한 관리

```javascript
<template>
  <div class="space-y-4">
    <div 
      v-for="group in permissionGroups" 
      :key="group.id"
      class="border rounded p-4"
    >
      <div class="flex items-center space-x-2 mb-3">
        <input
          type="checkbox"
          :checked="isGroupSelected(group)"
          @change="toggleGroup(group)"
          class="form-checkbox"
        >
        <h3 class="font-bold">{{ group.name }}</h3>
      </div>

      <div class="ml-6 space-y-2">
        <div 
          v-for="permission in group.permissions"
          :key="permission.id"
          class="flex items-center space-x-2"
        >
          <input
            type="checkbox"
            :value="permission.id"
            v-model="selectedPermissions"
            :disabled="permission.locked"
            class="form-checkbox"
          >
          <div>
            <label class="text-sm">{{ permission.name }}</label>
            <p class="text-xs text-gray-500">{{ permission.description }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const permissionGroups = ref([
  {
    id: 'users',
    name: '사용자 관리',
    permissions: [
      {
        id: 'users:read',
        name: '사용자 조회',
        description: '사용자 목록 및 상세 정보 조회',
        locked: false
      },
      {
        id: 'users:write',
        name: '사용자 수정',
        description: '사용자 정보 생성 및 수정',
        locked: false
      },
      {
        id: 'users:delete',
        name: '사용자 삭제',
        description: '사용자 계정 삭제',
        locked: true
      }
    ]
  }
])

const selectedPermissions = ref([])

const isGroupSelected = (group) => {
  const availablePermissions = group.permissions
    .filter(p => !p.locked)
    .map(p => p.id)
  
  return availablePermissions.every(p => 
    selectedPermissions.value.includes(p)
  )
}

const toggleGroup = (group) => {
  const availablePermissions = group.permissions
    .filter(p => !p.locked)
    .map(p => p.id)
  
  if (isGroupSelected(group)) {
    selectedPermissions.value = selectedPermissions.value
      .filter(p => !availablePermissions.includes(p))
  } else {
    selectedPermissions.value = [
      ...selectedPermissions.value,
      ...availablePermissions
    ]
  }
}
</script>
```

# 컴포넌트 추출과 재사용

```javascript
// CheckboxGroup.vue
<template>
  <div class="checkbox-group">
    <div class="group-header">
      <input
        type="checkbox"
        :checked="isAllSelected"
        @change="toggleAll"
      >
      <label>{{ title }}</label>
    </div>
    
    <div class="group-items">
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  modelValue: {
    type: Array,
    required: true
  },
  options: {
    type: Array,
    required: true
  },
  title: {
    type: String,
    default: '전체 선택'
  }
})

const emit = defineEmits(['update:modelValue'])

const isAllSelected = computed(() => 
  props.options.length === props.modelValue.length
)

const toggleAll = () => {
  emit('update:modelValue', 
    isAllSelected.value 
      ? [] 
      : props.options.map(option => option.value)
  )
}
</script>
```

```js
// 사용 예시
<template>
  <CheckboxGroup
    v-model="selectedFruits"
    :options="fruits"
    title="과일 선택"
  >
    <div v-for="fruit in fruits" :key="fruit.id">
      <input
        type="checkbox"
        :value="fruit.value"
        v-model="selectedFruits"
      >
      <label>{{ fruit.label }}</label>
    </div>
  </CheckboxGroup>
</template>

<script setup>
import { ref } from 'vue'
import CheckboxGroup from './CheckboxGroup.vue'

const fruits = ref([
  { id: 1, value: 'apple', label: '사과' },
  { id: 2, value: 'banana', label: '바나나' },
  { id: 3, value: 'orange', label: '오렌지' }
])

const selectedFruits = ref([])
</script>
```