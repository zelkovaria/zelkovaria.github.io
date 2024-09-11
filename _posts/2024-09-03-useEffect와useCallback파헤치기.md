---
layout: single
title: "트러블슈팅) useEffect와 useCallback 그리고 무한 렌더링 파헤치기"
categories: React
tag:
  [
    WEB,
    front-end,
    리액트,
    React,
    useCallback,
    useEffect,
    무한렌더링,
    the dependencies of useEffect Hook change on every render.,
    클로저,
    리렌더링,
  ]
sidebar:
  nav: "counts"
---

# useEffect의 동작 원리

## 트러블 슈팅

> ☄️ The 'getNotices' function makes the dependencies of useEffect Hook (at line 41) change on every render. Move it inside the useEffect callback. Alternatively, wrap the definition of 'getNotices' in its own useCallback() Hook.eslintreact-hooks/exhaustive-deps

마주했던 오류는 위와 같다. 에러의 내용을 천천히 살펴보면 `getNotices` 가 매번 새로운 참조를 가리키고, useEffect의 종속성 배열에 포함될 경우 netNotices가 `렌더링` 될 때마다 useEffect가 `무한히 실행`되면서 반복된다는 내용임을 알 수 있다.

💡 정확히는 useEffect에 의해 컴포넌트가 리렌더링이 될 경우 `컴포넌트 내부에 있는 함수`가 `재선언`이 된다. 때문에 컴포넌트 내부 함수는 컴포넌트가 리렌더링되면서 재선언이 되고, 이는 곧 메모리 주소가 바뀌게 되는 원인이 된다. 이때, useEffect에 있는 dependency에 함수가 존재하면 이전 함수와 비교를 하는데, 함수의 비교란 메모리 주소의 비교를 의미한다. 때문에 재선언된 함수는 이전에 선언되 함수와 메모리 주소가 다르므로 결과 또한 다르다 판단하여 useEffect 내부 콜백을 실행하게 된다

### 원인을 좀 더 자세히 이해해보자 🤔

- 함수또한 변수와 마찬가지로 재선언이 될 때마다 메모리 주소가 바뀐다. 이때, useEffect에 있는 getNotices는 재선언이 되는 거지 재실행이 되는 것이 아님에 주의해야한다!
  - 리렌더링 ≠ 재실행 = 재정의
- 리렌더링이 발생하면 곧 함수의 메모리 주소가 바뀌고 dependency에 getNotices가 존재하므로 또다시 재선언이 되고 해당 굴레가 반복이 된다
  - [dependency] ⇒ 메모리 주소가 바뀌면 리렌더링이 된다. 때문에 메모리 주소가 바뀐 getNotices는 다시 리렌더링을 발생시키고 동시에 무한 굴레가 반복된다
- 현재 문제점은 무한 렌더링을 발생시키는 구조가 `getNotices`에 있다는 것이다. 이는 곧, 함수 내부에서 state를 변경한다는 것과 같다.
  - useEffect가 getNotices를 재선언하는 동시에 finally에 있는 setIsLoading에 의해 변하는 구조 ⇒ “`isLoading`”자체가 원인이 된다

이러한 오류를 해결하는 방법을 공부하며 정리한 결과는 아래와 같다

1. <span style="color:#f8d374; font-weight: bold;">getNotices가 외부에 정의되면 새로운 참조를 가리키기 때문이므로 useEffect 내부에서 정의</span>해주기<br/>
2. <span style="color:#f8d374; font-weight: bold;">useCallback으로 감싸주기</span>(이를 통해 함수가 종속성 값들이 변경될 때만 새로 정의가 가능하다)<br/>
3. <span style="color:#f8d374; font-weight: bold;">useEffect의 디펜던시를 빈배열로</span> 하기<br/>

처음에 작성했던 코드는 아래 코드 블럭처럼 적어가는 도중에 에러가 떴다 ㅎ.. 차근차근 뜯어가며 공부해보자 🧐

```tsx
// 문제의 코드
const ViewAllNotice = () => {
const [notices, setNotices] = useState<NoticeType[]>([]);
const [isSorted, setIsSorted] = useState<SortKey>('desc');
const [page, setPage] = useState(0);
const [totalPage, setTotalPage] = useState(0);
const [isLoading, setIsLoading] = useState(false);
const sortOptions: SortOptions = { desc: '최신순', asc: '오래된순' };
const SIZE = 4;

const fetchNotices = async (isSorted: string, page: number, SIZE: number) => {
  const response = await Axios.get(
    `/announcements?sort=${isSorted}&page=${page}&size=${SIZE}`
  );
  return response.data;
};

const getNotices = async () => {
  try {
    const response = await fetchNotices(isSorted, page, SIZE);
    setTotalPage(response.totalPage);
    setNotices(response.simpleAnnouncements);
  } catch (err) {
    alert('올바른 동작을 해주세요');
    setIsLoading(true);
  } finally {
    setIsLoading(false);
  }
};

useEffect(() => {
  getNotices();
}, [getNotices]);
```

## **1️⃣ useEffect 내부에 함수 정의하기**

말 그대로 useEffect 내부에 사용하고자 하는 함수를 정의해주면 된다!

## 2️⃣ useCallback으로 감싸주기

```tsx
const getNotices = useCallback(async () => {
    try {
      const response = await fetchNotices(isSorted, page, SIZE);
      setTotalPage(response.totalPage);
      setNotices(response.simpleAnnouncements);
    } catch (err) {
      alert('올바른 동작을 해주세요');
      setIsLoading(true);
    } finally {
      setIsLoading(false);
    }
  }, [**isSorted, page**]);

  useEffect(() => {
    getNotices();
  }, [getNotices]);
```

- useCallback은 memoization 즉, 재선언이 되는 것을 막기위해 사용이 된다
- useCallback의 dependency에 들어가는 값은 해당 값이 바뀔 때마다 리렌더가 되는 기준이 된다

⇒ 무한렌더를 발생시키는 구조를 지닌 getNotices에서 계속해서 state가 변하는 값인 isSorted와 page를 디펜던시에 넣어줌으로 디펜던시의 값이 바뀔 때만 리렌더가 발생할 수 있도록 한다

- useCallback → 메모이제이션(재정의를 막으려고) → 디펜던시 값이 바뀔 때마다 `리렌더` → 컴포넌트 단위(그 안에 있는 것들은 함수) ⇒ 리렌더는 화면에 그려주는 애들만 렌더링이고 나머지는 선언이 된다

![image](https://github.com/user-attachments/assets/e3b7d9fe-6694-4d2f-b706-20b9b95ee12d)

아무 문제없이 잘 작동함을 확인할 수 있다!

## 3️⃣ useEffect의 dependency를 []로 처리하기

- dependency를 빈 배열로 두는 경우 맨 처음 마운팅될 때만 getNotices()가 정의되도록 한다. 현재 밑줄은 eslint 규칙 위반으로 인해 뜨는 내용인데, `React Hook useEffect has a missing dependency: 'getNotices'. Either include it or remove the dependency array.eslint[react-hooks/exhaustive-deps]` 다음 내용과 같다. 해당 rule은 내 의도에 맞는 dependency를 녹여낼 것이라면 warn 처리하면 해결이 가능하다!

![image](https://github.com/user-attachments/assets/d011afcf-433b-4e27-9cad-8b8ea2a01618)

- dependency의 deps 외에는 문제가 없음을 확인할 수 있었다!

### 🏃‍♀️‍➡️ 더 나아가기

useCallback을 사용하여 감을 좀 더 익혀보는 연습을 해봤다

1번

```tsx
import { useCallback, useEffect, useState } from "react";

const Example = () => {
  const [state, setState] = useState(0);

  const someFetch = () => {
    console.log("현재 state", state);
  };

  const handleClick = () => {
    setState((prev) => prev + 1);
  };

  useEffect(() => {
    someFetch();
  }, [someFetch]);

  return (
    <>
      <button onClick={handleClick}>증가버튼!</button>
    </>
  );
};

export default Example;
```

2번

```tsx
import { useCallback, useEffect, useState } from "react";

const Example = () => {
  const [state, setState] = useState(0);

  const someFetch = useCallback(() => {
    console.log("현재 state", state);
  }, []);

  const handleClick = () => {
    setState((prev) => prev + 1);
  };

  useEffect(() => {
    someFetch();
  }, [someFetch]);

  return (
    <>
      <button onClick={handleClick}>증가버튼!</button>
    </>
  );
};

export default Example;
```

1번 코드와 2번 코드별 실행 결과는 아래와 같다

![1번](https://github.com/user-attachments/assets/ad9fba2c-431d-41cb-9547-1e26be064134)

1번

![2번](https://github.com/user-attachments/assets/b779f702-2ba9-4b4e-bc16-180577a7c7a4)

2번

1번 코드는 버튼을 클릭할 때마다 state 값이 변경되지 않는 이유는 아래와 같다

1. someFetch는 매번 렌더링 될 때마다 새롭게 생성이 된다
2. useEffect는 someFetch가 변경될 때마다 동작한다
3. 결과적으로 `someFetch`가 매 렌더링마다 새로 생성되므로 `useEffect`가 매번 호출되는데, 이때 **클로저**의 개념을 확인할 수 있다.
   > `클로저`는 함수가 선언됐을 때의 환경을 기억하는 것을 의미하는데, 현재 someFetch가 선언될 때의 state 값은 0이므로 계속하여 0이라는 값을 기억하고 있는 것이다

2번은 1과 달리 useCallback으로 메모이제이션이 되어있음을 확인할 수 있다.

1. someFetch는 state 값이 변할 때만 새롭게 생성된다
2. useEffect는 someFetch가 변경될 때마다 동작한다
3. `someFetch`는 `state` 값이 변경될 때만 새로 생성되므로 `useEffect`는 `state`가 변경될 때만 호출된다

마지막으로 정리하자면, `useCallback` 없이 `someFetch`가 `useEffect`의 종속 배열에 포함되면, `someFetch`는 매 렌더링마다 새로 생성되어 초기값 0이 계속 찍힌다. `useCallback`을 사용하면 `someFetch`가 의존성 배열의 값이 변경될 때만 새로 생성이 되므로 효율적인 메모리 관리를 할 수 있게 된다

참고로 아래 코드또한 state 값이 변할 때마다 리렌더링이 되므로 2번의 코드와 같은 결과를 갖는다

```tsx
import { useCallback, useEffect, useState } from "react";

const Example = () => {
  const [state, setState] = useState(0);

  const someFetch = useCallback(() => {
    console.log("현재 state", state);
  }, [state]);

  const handleClick = () => {
    setState((prev) => prev + 1);
  };

  useEffect(() => {
    someFetch();
  }, [someFetch]);

  return (
    <>
      <button onClick={handleClick}>증가버튼!</button>
    </>
  );
};

export default Example;
```
