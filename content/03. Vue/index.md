---
title: Vue
---
v-for와 v-model을 input checkbox와 함께 사용하려면?
- input checkbox에 value를 설정한다. v-model을 ref([])와 연결시키면 체크된 값들로 이루어져 있을 거이다

# select multiple 여러 개 클릭만으로 선택하는 방법
```js
document.addEventListener("DOMContentLoaded", function () {  
    const selectElement = document.getElementById("board")  
  
    selectElement.addEventListener("mousedown", function (event) {  
        event.preventDefault()  
  
        const originalScrollTop = selectElement.scrollTop  
        const clickedOption = event.target  
  
        if (clickedOption.tagName === "OPTION") {  
            clickedOption.selected = !clickedOption.selected  
        }  
  
        selectElement.focus()  
        setTimeout(() => {  
            selectElement.scrollTop = originalScrollTop  
        }, 0)  
  
        return false  
    })  
}