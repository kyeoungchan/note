# v-bind
- Vue.js 상의 값을 HTML의 "속성" 값에 바인딩할 때 쓴다.
- 따라서 HTML에서 속성 이름을 argument로 준다.
- 참고로, `v-bind`를 생략해서 `:href="..."` 식으로 나타낼 수 있다.
    - `v-bind`만 콜론으로 생략된다!!
- 그리고 속성 이름도 동적으로 주어지기도 한다.(참고로만 알아두자. 아래에 `v-bind:[key]`를 보면 된다.
```vue
<img v-bind:src="imageSrc"/>
<button v-bind:[key]="value"></button>
<img :src="imageSrc"/>
```
## Class and Style Bindings
### Class 속성 활용
- 아래처럼 클래스를 동적으로 변환하여 CSS 스타일을 적용할 수 있다.
    ```vue
    const isOn = ref(false)
    <div :class="{ on: isOn }">Hello</div>
    ```
- 아래처럼 다수 개의 클래스를 적용할 수도 있다.
    ```vue
    const isOn = ref(false)
    const isIn = ref(true)
    <div class="staticClass" :class="{ on: isOn, in: inIn }">Hello</div>
    ```
  결과
  `<div class="static in">Hello</div>
- inline 방식으로 말고 객체에 말아서 객체를 한 번에 전달할 수도 있다.
    ```vue
    const isOn = ref(false)
    const isIn = ref(true)

    const helloObj = ref({
        on: isOn,
        in: inIn
    })

    <div class="staticClass" :class="helloObj">Hello</div>
    ```

### Style 속성 활용
- 클래스 대신에 style 속성을 활용해서 CSS를 적용하는 방식이다.
    ```vue
    <!-- style binding -->
    <div :style="{ fontSize: size + 'px' }"></div>
    <div :style="[styleObjectA, styleObjectB]"></div>
    ```
