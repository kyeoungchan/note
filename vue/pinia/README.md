# pinia
- 서로 다른 컴포넌트들이 동일한 상태에 종속되어있는 경우를 해결하기 위해 등장했다.
- 전역에서 참조할 수 있는 저장소의 역할을 한다.
- Pinia는 Vue의 공식 상태 관리 라이브러리다.
- stores 디렉토리에 js파일로 관리한다.
## pinia의 구성 요소
1. state: ref()
2. getters: computed()
3. actions: function()

stores/hello.js
```js
import { ref, computed } from 'vue'
import { defineStore} from 'pinia'

export const useHelloStore = defineStore('hello', () => {
    const myState = ref(0)
    const tripleMyState = computed(() => myState.value * 2)
    const addOne = () => myState.value++
    
    return { myState, tripleMyState, addOne }
})
```

다른 컴포넌트에서 호출할 때
```js
import { useHelloStore } from '@/stores/hello'

const store = useHelloStore()

// 상태 참조
console.log(store.myState)

// getters 참조
console.log(store.tripleMyState)

// actions 호출
store.addOne()
```