---
emoji: 🐯
title: react-router v6에 대해서 알아보자
date: '2021-11-17 19:00:00'
author: JaeMeDev
tags: react react-router
categories: front-end
---

![react-router](./img/reactrouter.png)

## react-router v6?

최근 `react`에서 라우팅을 관리하기 위해 사용하는 라이브러리인 `react-router`가 v6로 업데이트되었습니다. 업데이트 기념으로 마이그레이션 작업을 하면서 겪게된 문제점과 v6버전을 사용할 때의 약간의 팁을 포스팅해보려 합니다.

## Migration 준비과정

[공식문서](https://reactrouter.com/) 에서는 `react-router`를 v6 버전으로 업그레이드하기 전 진행해야 할 건 크게 두 가지라고 알려주고 있습니다.

1. React v16.8 이상으로 업그레이드
2. React Router v5.1로 업그레이드

Migration 작업을 하기 전 먼저 위 요구에 맞게 프로젝트 환경을 설정해줍니다.

## react-router V6로 Migration 해야하는 이유

크게 바뀐 점은 이전 버전 대비 번들 사이즈가 많이 줄어들었습니다. 이전 대비 약 70% 정도 크기가 줄었다고 하는데요. 이는 웹 성능 최적화에 큰 도움을 주기 때문에 마이그레이션이 꼭 필요하다고 생각됩니다. 

## 바뀐 점 1 : switch -> routes 로 네이밍

기존에 Route를 감싸는 부모 요소인 `Switch`가 `Routes`로 이름이 변경되었습니다. 네이밍이 직관적으로 변경되었다고 생각됩니다.

```jsx
<Routes>
  <Route />
</Routes>
```

## 바뀐 점 2 : exact 옵션 삭제

v6에서는 더 이상 Route의 `exact` 옵션이 필요하지 않습니다. 모든 경로가 기본적으로 정확히 일치하게 되었기 때문인데요.
만약 하위 경로가 있어 더 많은 URL을 일치시키려 한다면 `*`를 사용하여 매칭시킬 수 있습니다.

```jsx
<Route path="setting/*" />
```

## 바뀐 점 3 : StaticRouter의 import 위치 변경

import 경로가 변경되었습니다.

```jsx
import { StaticRouter } from 'react-router-dom/server';
```

## 바뀐 점 4 : render -> element

이전 버전에서는 컴포넌트를 렌더링하기 위해 `render` 속성에 컴포넌트를 반환하는 함수를 넣어줘 컴포넌트를 렌더링하였습니다.
v6에서는 `element` 속성을 통해 컴포넌트를 바로 넣어줄 수 있도록 변경되었습니다.

```jsx
<Route path="webtoon" element={<Webtoon />} />
```

## 바뀐 점 5 : Redirect 컴포넌트 제거

다른 분들 포스팅에서는 이 내용이 많이 빠져있던데 `<Redirect/>`컴포넌트가 v6 버전에서부터는 지원하지 않습니다. 그렇기 때문에 `<Redirect/>`를 사용한 곳은 수정이 필요합니다. 공식문서에서 v6 버전으로 업데이트하기 전 v5.1 버전으로 업데이트하는 과정에서는 이런 식으로 수정하라고 권유합니다.

```jsx
// v5.1 전 사용방법
<Rediect from="/freetime" to="/free/freetime" />

// v5.1 업데이트 시 수정
<Route path="/freetime" render={() => <Redirect to="/free/freetime" />} />
```

위의 작업이 끝났다면 v6에서 지원하는 `Navigate` 컴포넌트를 사용하여 마이그레이션을 진행해주면 됩니다. 이렇게 진행을 하면 마이그레이션 작업 중 발생하는 에러를 최소화 할 수 있으며 코드의 변경 또한 쉽습니다.

```jsx
<Route path="/freetime" element={<Navigate replace to="/free/freetime" />} />
```

## 바뀐 점 6 : 중첩 라우팅의 변화

v6 이전에 중첩 라우팅을 사용하려면 복잡하였는데 v6에서는 훨씬 더 간결하게 사용 가능해졌습니다. 중첩영역 렌더링 요소에 `Outlet` 속성을 제공하면 됩니다.

<br>

밑에 코드는 /setting/myinfo에 접근 시 MyInfo 컴포넌트가 렌더링 되는 예제입니다.

```jsx
const App = () => {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="setting" element={<Setting />}>
          <Route path="myinfo" element={<MyInfo />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
};

const Setting = () => {
  return (
    <div>
      <Outlet />
    </div>
  );
};
```


## 바뀐 점 7 : Optional parameters 의 지원 중지

개인적으로 이번 업데이트에서 단점이라 생각하는 부분 중 하나는 Optional parameters의 지원이 중지라고 생각합니다.

이전버전에서 `?`를 붙여 parameter 가 있거나 없거나를 명시해주었던 문법이 사라졌습다. 이슈를 보니 많은 유저들이 해당 기능이 사라진 것에 대해
분노가 있는 것 같습니다. (물론 저 포함입니다.🤬)

`Optional parameters` 중지에 의한 문제는 중첩 Route를 사용해서 해결할 수 있습니다.

밑은 검색 Route를 마이그레이션 할 때의 코드이다. 다른 포스팅에서 많이 확인이 불가능했기 때문에 꼭 참조해서 수정하도록 하자.

```jsx
// v5
<Route path="/search/:keyword?" component={SearchPage} />

// v6
<Route path="/search" element={<SearchPage/>}>
  <Route path=":keyword" element={<SearchPage/>} />
</Route>
```

## 바뀐 점 8 : useHistory -> useNavigate

v6에서는 `useHistory` 훅이 `useNavigate`로 이름이 변경되었습니다. 이름 뿐만아니라 기존에 push, replace, goBack 등 메소드로 동작하는
부분도 변경되었습니다. `react-native`와 네이밍을 비슷하게 하면서 맞춰가려는 것인지 모르겠지만 사용 방법이 직관적이지 않아서 개인적으로 좋지 못한 것 같습니다.

state를 변경하는 부분은 뒤의 인자에 넣어주면 됩니다.

```jsx
const navigate = useNavigate();

// push
navigate('/');

// replace
navigate('/', { state: { showLoginPop: true }, replace: true });

// goBack
navigate(-1);
```

## 바뀐 점 9 : TypeScript Hook

TypeScript를 사용할 경우 사용 방법이 변경되었습니다.

다른 것도 많이 변경되었지만 `useParams`의 경우 밑에처럼 입력해주던 것이 v6에서는 이런 식으로 변경되었습니다.

```tsx
// 이전 버전
const { name } = useParams<{ name: string }>();

// v6
const { id } = useParams<'id'>();
```

## 이밖에 바뀐점

사실 위에 적은 내용 이외에 `useRoutes`등 변경된 점은 많습니다. 다 정리하기에는 내용이 많아 중요한 점만 정리해서 적은 것뿐이며, 추가로 변경된 것을 확인하고 싶은 분들은 공식문서에 들어가서 확인하면 됩니다.

## 마이그레이션 시 주의 사항 및 꿀팁

마이그레이션을 하는 경우 소요되는 시간은 짧지만 귀찮은 작업이 많습니다. 그렇기에 공식문서를 정독하면서 마이그레이션을 하길 권장합니다.
블로그 같은 것보다 공식문서를 영어가 안 된다면 번역기를 써서 확인해보면 금방 마이그레이션 할 수 있을 것입니다.

다만 `Redirect`나 `Optional Parameters`의 경우 사용하는 코드가 다를 것이기 때문에 주의해서 변경해야 되며, 꼭 수정 후에는 테스트를 통해
확인해봐야 합니다.

제가 드릴 꿀팁은 `useNavigate`를 여러 custom hook으로 만들어서 사용하면 직관적으로 사용할 수 있습니다.

```jsx
export const useGoPush = () => {
  const navigate = useNavigate();
  return (url, state) => navigate(url, { state });
};

export const useGoReplace = () => {
  const navigate = useNavigate();
  return (url, state) => navigate(url, { state, replace: true });
};

export const useGoBack = () => {
  const navigate = useNavigate();
  return () => navigate(-1);
};
```

## 마지막으로

마이그레이션은 충분히 검토해 본 후 진행하는 것이 중요합니다. 장점만 보고 마이그레이션 하기에는 리스크가 클 수 있습니다.
현재 코드에서 어느 정도 수정이 필요한지 꼭 여러 번 검토 한 후 마이그레이션을 진행하면 훨씬 효율적인 마이그레이션이 될 것 같습니다.

<br>

