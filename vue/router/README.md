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