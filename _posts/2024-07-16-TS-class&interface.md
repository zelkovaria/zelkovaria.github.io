---
layout: single
title: "TypeScript 추상클래스와 인터페이스"
categories: TypeScript
tag: [TS, front-end, 타입스크립트, class, interface, 인터페이스]
sidebar:
  nav: "counts"
---

### 추상클래스

- 추상클래스는 다른 클래스가 상속받을 수 있는 클래스
  - 추상클래스의 인스턴스는 만들 수 없음
- 추상메소드를 만들려면, 메소드를 클래스 안에서 구현하지 않으면 됨

  ```tsx
  // JS ver
  class User {
  	constructor(firstname, lastName, nickName) {
  		this.firstName = firstName;
  		this.lastName = lastName;
  		this.nickName = nickName;
  	}
  }
  // TS ver
  class User {
  	constructor() {
  		protected firstName: string,
  		protected lastName: string,
  		protected nickName: string,
  		// 대신 필드가 어떠한 보호 등급인지(접근 제어자), 이름, 타입만 써주면 된다!
  	} {}

  	abstract getNickName():void
  }

  class Player extendsUser {
  	getNickName() {
  		console.log(this.nickName);
  	}
  }
  ```

  - 추상메소드는 내가 추상 클래스를 상속받는 모든 것들이 구현을 해야하는 메소드를 의미함
  - 메서드의 call signature만 작성해둬야함

  | 구분      | 선언한 클래스 내 | 상속받은 클래스 내 | 인스턴스 |
  | --------- | ---------------- | ------------------ | -------- |
  | private   | ⭕               | ❌                 | ❌       |
  | protected | ⭕               | ⭕                 | ❌       |
  | public    | ⭕               | ⭕                 | ⭕       |

- 추상클래스는 내가 직접적으로 인스턴스를 만들지 못하는 클래스지만 그 클래스를 상속할 수 있음
  - 추상 메소드가 있는 경우, 추상 클래스를 상속받는 클래스에서 내가 추상 메소드를 구현해야함

```tsx
// 사전 만들어보기
type Words = {
  [key: string]: string;
  // property에 대해서 미리 알진 못하지만 타입은 알고 있을 때 쓰면 됨
};

class Dict {
  private words: Words;
  constructor() {
    this.words = {};
  }
  add(word: Word) {
    if (this.words[word.term] === undefined) {
      // 주어진 단어가 아직 사전에 존재하지 않을 때
      this.words[word.term] === word.def;
    }
  }
}

class Word {
  constructor(public term: string, public def: string) {}
}

const kimchi = new Word("kimchi", "한국의 음식");

const dict = new Dict();

dict.add(kimchi); // 정상 작동
```

- type은 object의 모양을 알려주는 데 쓸 수도 있고 타입이 어떤 형식인지도 알려줄 수 있음
  즉, `type`을 이용해 타입스크립트에서 만들고 싶은 무수히 많은 종류의 타입을 설명하면 됨!(특정 값의 타입으로 지정도 가능함)

### 인터페이스

- 인터페이스는 오로지 오브젝트의 모양을 특정해주기 위한 목적 하나만을 가지고 있음

  - 하나는 type을 쓰고 오브젝트의 모양을 써주는 방법

    ```tsx
    type Team = "red" | "blue" | "yellow";
    type Health = 1 | 5 | 10;

    type Player = {
      nickname: string;
      team: Team;
      health: Health;
    };
    ```

  - 하나는 interface

    ```tsx
    type Team = "red" | "blue" | "yellow";
    type Health = 1 | 5 | 10;
    //인터페이스로는 이런거 못함

    interface Player {
      nickname: string;
      team: Team;
      health: Health;
    }
    ```

  ### type vs interface

  - 차이점: type 키워드는 interface 키워드에 비해 좀 더 활용할 수 있는게 더 많음<br/>
    → 오브젝트의 모양을 정해줄 수 있음<br/>
    → 특정 값들로만 제한할 수도 있음<br/>
    → 타입 alias를 만들 수 있음<br/>
    ⇒ 인터페이스는 객체 지향 프로그래밍의 개념을 활용해서 디자인되었고, 타입은 좀 더 유연함(타입 alias 가능, 지정한 값들로만 타입 지정이 가능 등등)

    ```tsx
    // interface
    interface User {
      name: string;
    }

    interface Player extends User {}

    const hyein: Player = {
      name: "hyein",
    };
    ```

    ```tsx
    // type
    type User = {
      name: string;
    };

    type Player = User & {};

    const hyein: Player = {
      name: "hyein",
    };
    ```

  - 인터페이스로는 property들을 축적시킬 수 있음(같은 인터페이스에 다른 이름을 가진 property들을 쌓을 수 있음)
    → 인터페이스도 타입으로 쓸 수 있음!!

    ```tsx
    interface User {
      health: number;
    }
    interface User {
      age: number;
    }
    interface User {
      name: string;
    }

    const hyein: User = {
      name: "hyein",
      age: 26,
      health: 10,
    };
    // 과 같이 property들이 축적이 가능하나, type으로 지정하면 불가능함
    ```

- 내가 TS에서 추상클래스를 만들면 JS에서 일반적인 클래스로 바뀌어버림
  - 우리는 표준화된 property와 메소드를 갖도록 해주는 청사진을 만들기 위해 추상클래스를 사용함<br/>
    → 인터페이스는 컴파일하면 JS로 바뀌지 않고 사라짐(컴파일되지 않는다는 소리임)
- 💡 인터페이스를 쓸 때 클래스가 특정 형태를 따르도록 어떻게 강제할까?

  - 현재 내가 하고싶은 건 추상클래스를 인터페이스로 바꾸는 것

  ```tsx
  abstract class User {
  	constructor(
  		protected firstName: string,
  		protected lastName: string
  	) {}
  	abstract sayHi(name: string): string
  	abstract fullName(): string
  }

  class Player extends User {
  	fullName() {
  		return `${this.firstName} ${this.lastName}`
  	}
  	sayHi(name: string) {
  		return `Hello ${name}. My name is ${this.fullName()}
  	}
  }
  ```

  ```tsx
  interface User {
  	firstName: string,
  	lastName: string,
  	sayHi(name: string): string,
  	fullName(): string
  }
  // extends를 쓰면 JS로 코드가 바뀜
  // interface가 타입스크립트에서만 존재할 수 있도록 implement 사용
  // -> 파일 사이즈 줄어듦
  class Player implements User {
  	constructor(
  		public firstName: string,
  		public lastName: string
  		// 인터페이스는 상속할 때 property를 private으로 만들지 못함
  	){}
  	fullName() {
  		return `${this.firstName} ${this.lastName}`
  	}
  	sayHi(name: string) {
  		return `Hello ${name}. My name is ${this.fullName()}
  	}
  }
  ```

  - `어댑터 패턴`과 같은 디자인 패턴을 사용하여 팀과 함께 일할 때 인터페이스를 만들어두고 팀원이 원하는 각자의 방식으로 클래스를 상속하도록 할 수 있음 (모두가 같은 인터페이스를 사용하면 같은 property, method를 갖게 됨)<br/>
    ⇒ 즉, 인터페이스는 오브젝트의 모양을 결정지을 수도 있지만, 클래스의 모양을 특정짓기도 함

- 클래스나 오브젝트의 모양을 정의하고 싶으면 interface, 그 외에는 type을 쓰라고들 많이 함(인터페이스를 상속시키는 방법은 직관적으로 보이긴 함)
- 다형성: 다른 모양의 코드를 가질 수 있게 해줌 → 제네릭을 사용
  - 제네릭은 placeholder 타입을 쓸 수 있게해줌(concrete 타입이 아님)
  - 제네릭은 다른 타입에 물려줄 수 있음(제네릭을 클래스로 보내고, 클래스는 제네릭을 인터페이스로 보낸 뒤에 인터페이스는 제네릭을 사용함)
