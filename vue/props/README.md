# Passing Props
- 공통된 부모 컴포넌트에서 동일한 데이터를 관리할 때 사용한다.
- Pass Props: 부모 => 자식
    - 데이터를 전달한다.
- Emit Event: 자식 => 부모
    - 자신에게 일어난 일을 알린다.
- 하향식 단방향 바인딩을 한다.

## props 보내기
- 부모 컴포넌트에서 보낸 props를 사용하기 위해서는 자식 컴포넌트에서 props를 명시적으로 선언해야한다.
- 부모 컴포넌트에서 자식 컴포넌트에게 props 보내기
```vue
<script setup>
import Child from '@/components/Child.vue'
</script>

<template>
  <div>
    <Child my-message="Hello Message!"/>
  </div>
</template>
```
Props를 선언하는 방식은 2가지가 있다.
1. 문자열 배열을 사용하는 방식(`defineProps([''])`)
```vue
<script setup>
defineProps(["myMessage"]);
</script>

<template>
    <div>
        {{ myMessage }}
    </div>
</template>
```
2. 객체를 사용하는 방식
```vue
<script setup>
defineProps({
  myMessage: String
});
</script>

<template>
  <div>
    {{ myMessage }}
  </div>
</template>
```
- 객체 속성의 key: props의 이름
- 객체 속성의 value: 데이터 타입에 해당하는 생성자 함수(Number, String 등)
- 객체 선언 문법을 사용하는 것을 권장들 한다고 한다.
> 참고로 자식 컴포넌트에서 Props를 선언하고 참조할 때는 camelCase를 사용하고, 부모 컴포넌트에서 props를 전달할 때는 kebab-case로 표기한다.

## Dynamic Props
```vue
<script setup>
import Child from "@/components/Child.vue";
import { ref } from "vue";

const hello = ref("Hi Hello");
</script>

<template>
  <div>
    <Child my-message="Hello Message!" :dynamic-props="hello" />
  </div>
</template>
```
```vue
<script setup>
defineProps({
  myMessage: String,
  dynamicProps: String,
});
</script>

<template>
  <div>
    {{ myMessage }}
  </div>
  <div>
    {{ dynamicProps }}
  </div>
</template>
```

