# Computed
- 미리 계산된 속성을 정의하는 함수다.
- computed 속성은 의존하고 있는 데이터들이 변경될 때마다 추적해서 반환값을 업데이트한다.
```vue
<div>
    <input type="checkbox" id="checkBoxId" v-model="metFirst">
    <label for="checkBoxId">만난 적 있나요?</label>
</div>
<div>{{sayWhat}}</div>

const { createApp, ref, computed } = Vue;
const metFirst = ref(true)
const sayWhat = computed(() => metFirst ? 'hello' : 'i saw you')
```
실행결과  
![computed_check_box.png](../../res/computed_check_box.png)

## computed를 사용하는 이유
- 의존하고 있는 데이터를 캐시한다.
- 따라서 데이터가 캐시될 때만 함수를 실행하므로 일반 method와는 다르다.