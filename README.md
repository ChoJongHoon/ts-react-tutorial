# ts-react-tutorial

[velpert](https://velog.io/@velopert)님의 [리액트 컴포넌트 타입스크립트로 작성하기](https://velog.io/@velopert/create-typescript-react-component) 실습

# 프로젝트 생성

```bash
$ create-react-app ts-react-tutorial --typescript
```

위와 같이 `--typescript`가 있으면 타입스크립트 설정이 적용된 프로젝트가 생성됩니다.

> 이미 만든 프로젝트에 타입스크립트를 적용하기 [Adding TypeScript](https://create-react-app.dev/docs/adding-typescript/)

# 새로운 컴포넌트 만들기

**src/Greetings.tsx**

```tsx
import React from "react";

type GreetingsProps = {
  name: string;
};

const Greetings: React.FC<GreetingsProps> = ({ name }) => (
  <div>Hello, {name}</div>
);

export default Greetings;
```

컴포넌트의 props에 대한 타입을 선언 할 때에는 `type`을 써도 되고, `interface`를 써도 되지만 프로젝트에서 일관성은 유지해야 합니다.

## React.FC 의 장단점

`React.FC`를 사용할 때는 props의 타입을 Generics로 넣어서 사용합니다. 이렇게 `React.FC`를 사용해서 얻을 수 있는 이점은 두가지가 있습니다.

첫 번째는, props에 기본적으로 `children`이 들어가 있다는 것 입니다.

![image](https://user-images.githubusercontent.com/42956032/66898659-8f05d300-f034-11e9-859b-906152e0b25c.png)

> 중괄호 안에서 `Ctrl`+`Space`를 눌러보면 확인 할 수 있습니다.

두 번째는, 컴포넌트의 `defaultProps`, `propTypes`, `contextTypes` 를 설정할 때 자동완성이 될 수 있다는 것입니다.

![image](https://user-images.githubusercontent.com/42956032/66922652-714f6280-f062-11e9-80e8-170bc24ed52d.png)

한편, 단점도 존재하긴 합니다. `children`이 옵셔널 형태로 들어가 있다 보니까 어찌 보면 컴포넌트의 props의 타입이 명백하지 않습니다. 예를 들어 어떤 컴포넌트는 `children`이 무조건 있어야 하는 경우도 있을 것이고, 어떤 컴포넌트는 `children`이 들어가면 안 되는 경우도 있을 것입니다. 결국 그에 대한 처리를 하고 싶다면 Props 타입 안에 `children`을 명시해야 합니다.

예를 들자면 다음과 같습니다.

```tsx
type GreetingsProps = {
  name: string;
  children: React.ReactNode;
};
```

결국 `React.FC`에 `children` props가 기본적으로 들어있는건 어찌보면 장점이 아닙니다. 차라리, `React.FC`를 사용하지 않고 GreetingsProps 타입을 통해 `children`이 있다 없다를 명백하게 명시하는게 덜 헷갈립니다.

추가적으로, (아직까지는) `React.FC`를 사용하는 경우 `defaultProps`가 제대로 작동하지 않습니다.

**src/Greetings.tsx**

```tsx
import React from "react";

type GreetingsProps = {
  name: string;
  mark: string;
};

const Greetings: React.FC<GreetingsProps> = ({ name, mark }) => (
  <div>
    Hello, {name} {mark}
  </div>
);

Greetings.defaultProps = {
  mark: "!"
};

export default Greetings;
```

그리고 App에서 해당 컴포넌트를 렌더링 한다면

**src/App.tsx**

```tsx
import React from "react";
import Greetings from "./Greetings";

const App: React.FC = () => {
  return <Greetings name="Hello" />;
};

export default App;
```

![image](https://user-images.githubusercontent.com/42956032/66923538-f6874700-f063-11e9-9c00-e9878283643b.png)

`mark`를 `defaultProps`로 넣었음에도 불구하고 `mark`값이 없다며 제대로 작동하지 않습니다.

`React.FC`를 쓰면서 `defaultProps`를 사용하려면 결국 코드를 다음과 같이 작성하는 수 밖에 없습니다.

**src/Greetings.tsx**

```tsx
import React from "react";

type GreetingsProps = {
  name: string;
  mark: string;
};

const Greetings: React.FC<GreetingsProps> = ({ name, mark = "!" }) => (
  <div>
    Hello, {name} {mark}
  </div>
);

// 결국 무의미해진 defaultProps?
Greetings.defaultProps = {
  mark: "!"
};

export default Greetings;
```

바로, 위와 같이 비구조화 할당을 하는 과정에서 기본 값을 설정해주는거죠. 이렇게 하면 아래에 있는 `defaultProps`는 참 무의미해집니다.

> 난 이렇게 해도 `src/App.tsx`에서 에러가 난다...

반면, 만약 `React.FC`를 생략하면 어떻게 될까요?

**src/Greeting.tsx**

```tsx
import React from "react";

type GreetingsProps = {
  name: string;
  mark: string;
};

const Greetings = ({ name, mark }: GreetingsProps) => (
  <div>
    Hello, {name} {mark}
  </div>
);

Greetings.defaultProps = {
  mark: "!"
};

export default Greetings;
```

`React.FC`를 생략하면 오히려 아주 잘 작동합니다! 이러한 이슈 때문에 `React.FC`를 사용하지 말라는 [팁](https://medium.com/@martin_hotell/10-typescript-pro-tips-patterns-with-or-without-react-5799488d6680#78b9)도 존재합니다.

취향에 따라 화살표 함수도 만약 사용하지 않는다면 다음과 같은 형태가 됩니다.

**src/Greetings.tsx**

```tsx
import React from "react";

type GreetingsProps = {
  name: string;
  mark: string;
};

function Greetings({ name, mark }: GreetingsProps) {
  return (
    <div>
      Hello, {name} {mark}
    </div>
  );
}

Greetings.defaultProps = {
  mark: "!"
};

export default Greetings;
```

화살표 함수를 쓸 지, `function` 키워드를 사용해서 선언을 할 지는 여러분의 자유입니다! 다만, `React.FC` 의 사용은 아직까지는 저는 권장하지 않습니다. 언젠간 이 이슈가 해결되리라 생각합니다.
