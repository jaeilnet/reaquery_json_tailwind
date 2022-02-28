dockerfiler 작성

docker-compose 작성

docker-compose 명령어

docker-compose up 실행
docker-compose rm 생성한 컨테이너 일괄 삭제
docker-compose down 네트워크 정보, 볼륨,컨테이너 일괄 정지 및 삭제
docker-compose down --rmi all 네트워크 정보, 볼륨, 컨테이너 일괄 정지 및 이미지삭제

server 2개연다

docker exec -it 70f0a35f7c3e sh

wget http://server:4000

http://server:4000/todo

# 과제 2 Todo 만들기 (`Ts` + `React_query` + `json-server` + `MUI`)

## 미리보기

### 로그인

![login](./img/json_todo.png)

### 투두 목록

![todoList](./img/json_todoList.png)

# 리액트 쿼리 [공식문서](https://react-query.tanstack.com/overview)

## 간단한 소개

리액트 쿼리도 리덕스와 마찬가지로 상태관리 라이브러리지만 리덕스를 대체 할 수 있는 상태 라이브러리로 요즘 핫합니다.

대체 가능한 이유는,

### 1. 데이터 가져오기

리덕스는 스토어 - 액션 타입 - 액션 함수 - 비동기 미들웨어 - 초기state 설정 - 리듀스 -> 컴포넌트에서 디스패치 - 컴포넌트에서 state 값 가져오기.. 와 같은 아주 `복잡하고 다소 반복적일수도 있으며 방대한 코드양`을 요구합니다.

하지만 리액트 쿼리는 쿼리 hooks(useQuuery, useMutation)의 콜백으로 fetch 함수만 넣어주면 로딩, 에러, 데이터 등은 기본적으로 반환 값으로 가져 올 수 있고 이외에도 다양한 옵션(기능)들이 있어서 리덕스와 대조되어 비교적 쉽고 코드량도 엄청 줄어들었습니다.

### 2. 서버상태 관리

기존에 리덕스와 같은 라이브러리는 서버상태와 클라이언트 상태를 같이 사용했지만 리액트 쿼리에서는 서버와 클라이언트 상태를 따로 취급합니다.

리액트 쿼리는 서버상태관리에 대해서 다루고 있는데,
`서버상태관리`라고 하면 `캐싱`, `로딩상태`, `api통신에러 상태` 등이 있을 수 있습니다.

---

## Todo에서 리액트 쿼리 사용

`useQuery`는 CURD 중 단순히 읽어오는 R 데이터에 적합합니다.

```js
// todoList 를 가져오는 useQuery`

// key 값과 fetch 함수를 넣음
const { data, isLoading, isError } = useQuery("getTodo", getApi);

const addTodoMutation = useMutation(addTodoAPI, {
  // fetch 함수가 성공했을 경우
  onSuccess: (data) => {
    // 데이터를 그대로 컴포넌트에 뿌려도 되지만 state에 담아둔다.
    setTodoList(todoList.concat(data?.data));

    queryClient.invalidateQueries("todo");
  },
  // fetch 실패 했을 경우
  onError: (err) => {
    console.log(err, "err");
  },
});
```

`useMutation`는 데이터변형이 이뤄지는 fetch에 적합하다. CURD 중 R 을 제외한 CUD 에 적합하다.

```js
// todo 작성

// fetch 함수를 넣어준다.
const addTodoMutation = useMutation(addTodoAPI, {
  onSuccess: (data) => {
    setTodoList(todoList.concat(data?.data));

    // 쿼리 무효화 이전의 쿼리 key가 있다면 지워준다.
    // 이렇게 해야지 보여주는 데이터가 리프레시 된다.
    queryClient.invalidateQueries("todo");
  },
  onError: (err) => {
    console.log(err, "err");
  },
});

// addTodo 함수
const onAddTodo = (contents: string): void => {
  if (contents.trim().length < 0) {
    return window.alert("투두를 입력하세요");
  }

  // mutation이 일어나야하는 부분에서 mutation.mutate(파라미터)를 호출해주면 된다.
  addTodoMutation.mutate(contents);
};
```

---

## 리액트 쿼리는 서버상태관리에 적합하지만 전역상태관리는 ??

리덕스는 스토어에 서버에서 받아온 값을 저장하여 컴포넌트에 전역으로 사용 할 수 있었습니다.

하지만 리액트 쿼리는 데이터를 스토어에 담지도 않고 mutation으로 요청한 쿼리를 바로바로 렌더링하면 되니까 전역적으로 사용 할 필요를 없게 됬지만 `전역관리`를 해야 하는 데이터가 있을 수 있습니다.

login 상태유지나 modal state 또는 데이터를 가지고 있어야하는 경우? 가 있을 수 있는데 이럴때는 다른 `라이브러리와 조합`을 해야합니다.

이런 경우에는 redux 를 쓰거나 react hooks 중 useContext 나 recoil 등이 있습니다.

---

## Context + react_query

todo에서 login 상태를 header 에도 가지고 있고 다른 컴포넌트에도 물론 사용합니다.

최상위에서 props 로 내려줄까도 생각했지만 드릴링은 최대한 피하려고 했습니다.

그래서 사용한 방법이 `useContext` 입니다.

```js
import React, { useEffect, useState } from "react";
import { createContext } from "react";
import { useMutation } from "react-query";
import { loginCheckAPI, postLoginAPI } from "../api";

interface AuthContextType {
  isLogin: boolean;
  userId: string;
  onSubmitLogin: (userId: string, password: string) => void;
  onLogout: () => void;
}

// createContext 만들기
export const loginContext = createContext({
  isLogin: false,
  userId: "",
  onSubmitLogin: (userId: string, password: string) => {},
  onLogout: () => {},
});

export const AuthContext: React.FC = ({ children }) => {
  // 로그인상태 state
  const [isLogin, setIsLogin] = useState < boolean > false;
  // user 정보 state
  const [userId, setUserId] = useState < string > "";

  // 로그인 뮤테이션
  const loginMutation = useMutation(postLoginAPI, {
    onSuccess: (data) => {
      // 로그인 true
      setIsLogin(true);
      setUserId(data.data.userId);

      localStorage.setItem("isLogin", "loginTrue");
    },
    onError: (err) => {
      console.log(err, "loginError");
    },
  });

  // 로그인 함수
  const onSubmitLogin = (userId: string, password: string): any => {
    loginMutation.mutate({
      userId,
      password,
    });
  };

  // 로그아웃 (따로 api통신은 안함)
  const onLogout = () => {
    if (!isLogin) {
      return;
    }

    // token 제거
    localStorage.removeItem("isLogin");

    // 로그아웃 했으니 login state 는 False
    setIsLogin(false);
  };

  useEffect(() => {
    // 토큰 값 체크
    const getToken = localStorage.getItem("isLogin");

    // 토큰 값이 있으면 로그인 true
    if (getToken) {
      setIsLogin(true);
    }
  }, []);

  const todoContextValue: AuthContextType = {
    isLogin: isLogin,
    userId: userId,
    onSubmitLogin: onSubmitLogin,
    onLogout: onLogout,
  };

  return (
    <loginContext.Provider value={todoContextValue}>
      {children}
    </loginContext.Provider>
  );
};
```

context 로 로그인상태, user정보 등을 전역으로 관리했습니다.

---

## json-server

프론트에서 api 통신을 테스트 해볼 수 있는 라이브러리입니다.

src 폴더 안에 db.json 을 작성하고 script 를 추가하여 사용했습니다.

```js
{
  "todo": [
    {
      "contents": "리액트",
      "status": false,
      "id": 1
    },
    {
      "contents": "리액트 쿼리",
      "status": false,
      "id": 2
    },
    {
      "contents": "머티리얼 디자인",
      "status": false,
      "id": 3
    },
    {
      "contents": "도커",
      "status": false,
      "id": 4
    },
    {
      "contents": "123",
      "status": false,
      "id": 5
    },
    {
      "contents": "124",
      "status": false,
      "id": 7
    }
  ],
},

// package.json

 "scripts": {
    "server": "npx json-server -H 0.0.0.0 db.json --port 4000"
  }
```
