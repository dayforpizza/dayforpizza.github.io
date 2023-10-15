---
title: "React 01: To-do 앱 기본 컴포넌트 만들기"
excerpt: "React로 To-do 앱의 기본 컴포넌트를 작성해보자"
categories:
  - React
tags:
  - To-do
header:
  teaser: /assets/posts/images/2023-10-09-12-10-09.png
date: 2023-10-09
---

# 개발 환경
- VS Code
- Node.js: v20.7.0
- npm: v10.1.0
- React: v18.2.0

# 프로젝트 준비

## npx를 이용하여 React 프로젝트 생성
- 생성할 디렉토리로 이동하여 아래의 명령어 실행

```
npx create-react-app my-todo
```

## React 서버 시작
- my-todo 디렉토리로 이동하여 ```npm start``` 실행
- 서버가 시작되면 인터넷 브라우저를 통해 첫 화면을 확인할 수 있다.

![](/assets/posts/images/2023-10-09-12-10-09.png)


## 컴포넌트 파일 생성

> 컴포넌트란?
> 이론적으로는 재사용이 가능한 독립적인 모듈이라 하고 쉽게 말해서 조립식 컴퓨터는 CPU, 메모리, 그래픽 카드 등 독립된 단일 부품을 가지고 합쳐 하나의 컴퓨터를 만든다.
> 이러한 부품들을 컴포넌트라고 생각하면 된다.

### components 디렉토리 내 각각의 컴포넌트 파일을 생성
- src 폴더 안에 components 디렉토리를 생성 후 그 안에 아래 파일을 생성해주자

> **./components/**
> - Template.js: 뼈대가 되는 컴포넌트
> - TodoInsert.js
> - TodoItem.js: 리스트 안의 텍스트 내용
> - TodoList.js: 리스트

# 개발 시작
## 0. App.js
- src/App.js
- React와 Vue 등 JavaScript 프레임워크는 App.js 를 통해 앱을 표출해준다.

### 첫 페이지를 생성해보자.

```javascript
import './App.css';

const App = () => {
    return (
        <div> 안녕하세요. </div>
    );
};

export default App;
```

- 위 같이 App 변수를 지정해주면 "**안녕하세요**"란 내용이 출력해준다.

![](/assets/posts/images/2023-10-09-12-07-15.png)

## 1. Template 컴포넌트 작성
1) Template 모듈 정의
```javascript
const Template = () => {
    return (
        <div>
            <div>오늘의 할일 (0)</div>
            <div>이 곳에 할 일을 작성합니다.</div>
        </div>
    );
};

export default Template;
```

2) App.js에서 Template 모듈 삽입
- 모듈을 참조할 수 있도록 import

```javascript
import Template from './components/Template.js';
```

- Template 변수로 변경

```javascript
const App = () => {
    return (
        <Template> 안녕하세요. </Template>
    );
};
```

- 위 코드로 저장을 하면 "**안녕하세요**"란 내용은 없어지고 ```Template```에 있는 내용으로 대체된다.
- 객체를 지정해주자

3) children 객체 위치 지정
```javascript
const Template = ({children}) => {
    return (
        <div>
            <div>오늘의 할일 (0)</div>
            <div>{children}</div>
        </div>
    );
};
```
- 위 처럼 ```{children}``` 객체를 지정해주면 ```<Template> </Template>``` 사이에 내용이 해당 객체로 지정되어 출력된다.

![](/assets/posts/images/2023-10-09-12-27-27.png)

> 이제 ```{children}``` 객체에 ```TodoList``` 컴포넌트를 넣어주고자 한다.

## 2. TodoList 컴포넌트 작성
1) TodoList 모듈 삽입
```javascript
import TodoList from './components/TodoList.js';
```

2) Template 사이에 TodoList 태그 정의
```javascript
const App = () => {
    return (
        <Template>
            <TodoList />    //<TodoList> </TodoList>와 같은 문법
        </Template>
    );
};
```

3) TodoList 모듈 정의
- App.js 에서 ```{todos}``` 인자를 받아오는 변수 정의
> todos는 할 일 객체가 들어있는 배열

```javascript
const TodoList = ({todos}) => {
    return (
        <div>{todos.map(todo => (
            <div>{todo.text}</div>
            ))}
        </div>
    );
};

export default TodoList;
```
> ```todo```라는 배열 요소 함수를 받고 배열 중 ```.text``` 값을 가져온다

4) Hooks(useState) 함수를 이용하여 todos 배열 작성
```javascript
import {useState} from 'react';
```
> 참고: [Using the State Hook](https://ko.legacy.reactjs.org/docs/hooks-state.html)

- App 변수 안에 todos 배열 작성

```javascript
  const [todos, setTodos] = useState([  //setTodos: 할일 목록을 조작하는 함수
    {
      id: 1,
      text: "할일 1",
      checked: true
    },
    {
      id: 2,
      text: "할일 2",
      checked: false
    },
    {
      id: 1,
      text: "할일 3",
      checked: true
    }
  ]);
```

5) App의 ```TodoList```에서 ```todos``` 인자를 받아올 수 있게 정의
```javascript
const App = () => {
    return (
        <Template>
            <TodoList todos={todos} />    //<TodoList> </TodoList>와 같은 문법
        </Template>
    );
};
```

![](/assets/posts/images/2023-10-09-13-09-48.png)

## 3. TodoItem 컴포넌트 작성
1) ```TodoList```의 ```todo```객체에서 text 가져오기
```javascript
const TodoItem = ({ todo }) => {
    const { text } = todo;
    return <div>{text}</div>;
};

export default TodoItem;
```

2) ```TodoList```에 ```TodoItem```모듈 삽입
```javascript
import TodoItem from './TodoItem';
```
> 같은 디렉토리에 있기 때문에 ```./components/TodoItem```이 아닌 ```./TodoItem```을 입력해준다.

3) ```<TodoItem />``` 태그 정의
```javascript
const TodoList = ({ todos }) => {
    return (
        <div>
            {todos.map(todo => (
                <TodoItem todo={todo} key={todo.id} />
            ))}
        </div>
    );
};
```

> <font color="red">Warning: Each child is a list should have a unique "key" prop.</font>
> 리스트를 렌더링할 때는 반드시 key를 넣어줘야 한다. ```key={todo.id}```

## 소스 코드
### App.js
```javascript
import './App.css';
import {useState} from 'react';
import Template from './components/Template.js'
import TodoList from './components/TodoList.js'

const App = () => {
  const [todos, setTodos] = useState([
    {
      id: 1,
      text: "할일 1",
      checked: true
    },
    {
      id: 2,
      text: "할일 2",
      checked: false
    },
    {
      id: 1,
      text: "할일 3",
      checked: true
    }
  ]);
  return (
    <Template>
      <TodoList todos={todos} />
    </Template>
  );
};

export default App;
```

### Template.js
```javascript
const Template = ({children}) => {
    return (
        <div>
            <div>오늘의 할일 (0)</div>
            <div>{children}</div>
        </div>
    );
};

export default Template;
```

### TodoList.js
```javascript
import TodoItem from './TodoItem';


const TodoList = ({ todos }) => {
    return (
        <div>
            {todos.map(todo => (
                <TodoItem todo={todo} key={todo.id}/>
            ))}
        </div>
    );
};

export default TodoList;
```

### TodoItem.js
```javascript
const TodoItem = ({ todo }) => {
    const { text } = todo;
    return <div>{text}</div>;
};

export default TodoItem;
```


# 참고 사이트

- [(유튜브) React로 Todo App 만들기](https://youtube.com/playlist?list=PLyjjOwsFAe8J9tqYqO_y7Fr6FTSncZQZI&si=BQo8kkpz4ikXRGcE){:target="_blank"}
- [https://ko.legacy.reactjs.org/docs/introducing-jsx.html](https://ko.legacy.reactjs.org/docs/introducing-jsx.html){:target="_blank"}
- [https://react-icons.github.io/react-icons/](https://react-icons.github.io/react-icons/){:target="_blank"}