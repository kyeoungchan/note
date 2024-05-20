# Router
## Routing
- 네트워크에서 경로를 선택하는 프로세스다.
- Vue에서 다른 페이지 간의 전환과 경로를 선택하고 관리할 수 있도록 하는 기술을 뜻한다.
## Vue에서 Router 사용하기
프로젝트 구조상
1. App.vue에서의 구조가 변한다.
```vue
<template>
    <header>
        <RouterLink to="/">홈으로</RouterLink>
        <RouterLink to="/board">게시판</RouterLink>
    </header>
    <RouterView/>
</template>
```
2. router 디렉토리가 생기고, 그 안에는 index.js가 들어간다.
    - index.js에는 라우팅과 관련된 설정들이 들어간다.
3. views 디렉토리가 생긴다.
    - RouterView 위치에 렌더링될 컴포넌트들이 존재한다.
    - 일반 컴포넌트들과의 구분을 위하여 뒤에 View를 붙여서 이름을 정한다.

<br>


index.js
```javascript
const router = createRouter({
    routes: [
        {
            path: '/',
            name: 'home',
            component: HomeView
        },
        {
            path: '/board',
            name: 'board',
            component: BoardView
        },
        // ...
    ]
})
```

App.vue
```vue
<RouterLink to="/">홈으로</RouterLink>
<RouterLink to="/board">게시판</RouterLink>
```

### Named Routes
위의 코드처럼 링크로 연결하는 대신 name attribute로 경로를 지정한다.

App.vue
```vue
import HomeView from "@/views/HomeView.vue"
import BoardView from '@/views/BoardView.vue'

<RouterLink :to="{ name: 'home' }">홈으로</RouterLink>
<RouterLink :to="{ name: 'board' }">게시판</RouterLink>
```

### 매개변수를 활용하여 동적으로 경로 지정하기
index.js
```vue
import UserView from '@/views/UserView.vue'

const router = createRouter({
    routes: [
        {
            path: '/user/:id',
            name: 'user',
            component: UserView
        }
    ]
})
```
UserView.vue
```vue
<template>
    <div>
        <h1>UserView {{ $route.params.id }}번 페이지</h1>
    </div>
</template>
```
요즘에는 아래와 같이 Composition API 방식으로 작성하는 것을 권장한다.
UserView.vue
```vue
import { ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const userId = ref(route.params.id)

<template>
    <div>
        <h1>UserView {{ userId }}번 페이지</h1>
    </div>
</template>
```

## 프로그래밍 방식의 navigation
1. 다른 위치로 이동하기 : `router.push()`

```vue
<RouterLink :to="{ name: 'home' }">홈으로</RouterLink>
```
위 아래 코드는 동치다.
```vue
import { useRouter } from 'vue-router'

const router = useRouter()

const toHome = () => router.push({ name: 'home' })
<button @click="toHome">홈으로</button>
```

### router.push 호출하는 다양한 방법
```vue
router.push('/my-path')

router.push({ path: '/my-path'})

router.push({ name: 'myName', params: { param1: 'hello' } })

router.push({ name: 'myName', query: { param2: 'hello2' } })
```
2. 현재 위치 바꾸기 : `router.replace()`
- push()와 거의 동일하다.
- 다만 stack에 새로운 항목을 push하지 않고 바로 url을 이동시키기 때문에 뒤로가기를 사용할 수 없게 이동시키는 방법이다.

## Navigation Guard
### Globally 전역 가드
- 애플리케이션 전역에서 동작시킬 때 사용한다.
- index.js에서 정의한다.
- `router.beforeEach()`: 다른 URL로 이동하기 직전에 실행되는 함수다.
```js
router.beforeEach((to, from) => {
    // ...
   return false
})
```
- to: 이동할 URL 정보가 담긴 Route 객체다.
- from: 현재 URL 정보가 담긴 Route 객체다.
- `return false`: 다른 URL로의 이동을 취소한다.
- `return { name: 'Hello' }`: 해당 Route를 호출하고, return이 없다면 `to` 객체의 Route로 이동한다. 
### Per-route 라우터 가드
- 특정 route에서만 동작시킬 때 사용한다.
- index.js의 각 routes에서 정의한다.
- `router.beforeEnter()`: 다른 Route에서 들어올 때만 실행된다. (자신에서 자신으로는 파라미터나 쿼리가 바뀌어서 다시 호출되든 상관없다.)

```json5
{
   path: '/member/:id',
   name: 'member',
   beforeEnter: (to, from) => {
     // ...
    return
    false
   }
}
```
### In-component 컴포넌트 가드
- 특정 컴포넌트 내에서만 동작시킬 때 사용한다.
- 해당 컴포넌트의 Script에서 정의한다.
- `onBeforeRouteLeave()`: 현재 Route에서 다른 Route로 이동하기 전에 실행된다.
- `onBeforeRouteUpdate()`: 이미 렌더링된 컴포넌트에서 업데이트되기 전에 실행된다.