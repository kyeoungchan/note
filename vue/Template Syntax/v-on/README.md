# v-on
- DOM 요소에 Event Listener를 바인딩할 때 쓴다.
- `v-bind:`의 약어가 `:`라면, `v-on:`의 약어는 `@`이 된다.
- Inline handlers 방식과 Method handlers 방식으로 나뉜다.

## Inline handlers
- 단일 구문을 실행하거나 메서드를 호출할 때 사용한다.
```vue
const cnt = ref(0)
const sayCnt = (cnt) => {
    console.log(cnt)
}
<button @click="cnt++">cnt increament</button>
<button @click="sayCnt(cnt)">sayCnt Call</button>
```
참고로 위의 예시글에서 cnt를 인자로 전달하지 않아도 cnt가 호출된다.
- 다만, cnt.value로 호출해야한다.
- 왜냐하면 HTML에서 cnt를 호출할 때는 값으로서 호출이 되지만 파라미터로 전달하지 않고 js 내부에서 cnt를 호출할 때는 ref() 객체로 호출하기 때문에 `.value`로 출력하지 않으면 객체가 출력된다.
- 그리고 sayCnt()에서 괄호를 생략해도 된다.
    - 그러면 아래에서 또 다룰 Method handlers 방식이다.
```vue
const cnt = ref(0)
const sayCnt = () => {
    console.log(cnt.value)
}
<button @click="cnt++">cnt increament</button>
<button @click="sayCnt()">sayCnt Call</button>
<button @click="sayCnt">sayCnt Call</button>
```
- 아래에 나오는 Method handlers에서는 DOM Event 객체가 자동으로 전달되지만, Inline handlers에서 event 인자에 접근하려면, `$event`를 적용하면 된다.
```vue
const helloMethod = (msg, event) => {
    console.log(event)
    console.log(event.currentTarget)
    console.log(`Hello ${msg}!`)
}

<button @click="helloMethod('Hello Vue', $event)">Hello</button>
```
실행결과 console
```text
PointerEvent {isTruest: true, ...
<button>Hello</button>
Hello Hello Vue!
```

## Method handlers
- 복잡한 구문들을 실행할 때 사용하는 것이 Method handlers다.
- DOM Event 객체는 자동으로 파라미터에 적용돼서 전달된다.
```vue
const msg = ref('Hello Vue')
const helloMethod = (event) => {
    console.log(event)
    console.log(event.currentTarget)
    console.log(`Hello ${msg.value}!`)
}

<button @click="helloMethod">Hello</button>
```
실행결과 console
```text
PointerEvent {isTruest: true, ...
<button>Hello</button>
Hello Hello Vue!
```

## Key Modifiers
```vue
<input @keyup.enter="onEnter" />
```
엔터 키를 눌렀을 때 onEnter 이벤트를 호출한다.
