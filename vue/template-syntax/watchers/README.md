# Watch
- 감시하는 데이터가 변경될 때마다 콜백함수를 호출하는 역할을 한다.
- 구조는 다음과 같다.
```vue
watch(var, (newVal, oldVal) => {
    // method contents
}
```
- var: 감시하는 변수
- newVal: 바뀐 값
- oldVal: 바뀌기 전의 값

```vue
<button @click="cnt++">cnt Add</button>
<p>cnt: {{ cnt }}</p>

const { createApp, ref, watch } = Vue;

const cnt = ref(0)

const cntWatch = watch(cnt, (newVal, oldVal) => console.log(`newVal: ${newVal}, oldVal: ${oldVal}`))
```

## Compted vs Watchers
둘다 데이터의 변화를 감지하고 처리할 때 사용한다.
- computed의 경우, 의존하는 데이터의 값의 변화에 따른 변경된 값을 반환한다.
    - 즉, return이 필요한 경우 사용한다.
- watchers의 경우, 의존하는 데이터의 값이 변화했을 때 어떤 작업을 수행하는데, 꼭 return을 할 필요가 없다.
    - 따라서 의존하는 데이터는 template에서 알고 있어야하지만, watchers 함수에 대해서는 template이 몰라도 된다.
    - 위의 코드에서도 cntWatch는 js 코드 내부에서만 알고 있다.