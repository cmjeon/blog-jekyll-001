---
layout: single
title: "생활코딩의 'react 2022 개정'"
categories:
  - "react"
tags:
  - "생활코딩"
---

## 생활코딩의 'react 2022 개정'

### 강의와 따라하기 Repository

- [https://www.youtube.com/playlist?list=PLuHgQVnccGMCOGstdDZvH41x0Vtvwyxu7](https://www.youtube.com/playlist?list=PLuHgQVnccGMCOGstdDZvH41x0Vtvwyxu7)
- [https://github.com/cmjeon/my-react-app-001](https://github.com/cmjeon/my-react-app-001)

### 1. 수업소개

### 2. 실습환경구축

### 3. 소스코드수정방법

#### 배포용 버전 만들기

```shell
# 배포판 만들기
$ yarn run build
```

build 디렉토리가 생긴다.
serve 명령어를 설치한다.

```shell
$ npm install -g serve
```

serve 를 실행한다.

```shell
$ serve -s build
```

### 4. 컴포넌트만들기

뜸근없지만 React UI Framework

Material UI [https://mui.com/core/](https://mui.com/core/)

Semantic UI [https://react.semantic-ui.com/](https://react.semantic-ui.com/)

### 5. props

함수에 파라미터를 통해 변수 전달

```javascript
// ...
function Header(props) {
  return (
    <header>
      // highlight-next-line
      <h1><a href="/">{props.title}</a></h1>
    </header>
  )
}

// ...

function App() {
  
  // ...
  
  return (
    <div className="App">
      // highlight-next-line
      <Header title="WEB"></Header>
      // ...
    </div>
  );
}

export default App;
```

### 6. 이벤트

이벤트를 추가했더니 컴포넌트의 가독성이 떨어짐

```javascript
function Nav(props) {
  const lis = props.topics.map(topic => {
    // highlight-start
    return <li key={topic.id}><a id={topic.id} href={`/read/${topic.id}`} onClick={event => {
      event.preventDefault()
      props.onChangeMod(event.target.id)
    }}>{topic.title}</a></li>
    // highlight-end
  })
  return (
    <nav>
      <ol>
        {lis}
      </ol>
    </nav>
  )
}
```

### 7. state

useState 의 리턴값은 배열이다.

배열의
- 0 번째 객체 : 상태의 값을 읽을 때 쓰는 데이터
- 1 번째 객체 : 상태의 값을 변경할 때 쓰는 함수

setMode 를 호출하면
1. app 의 function 이 새롭게 호출
2. 호출의 결과로 변경된 mode 로 인해 return 이 렌더링 됨

> app 의 function 2번 호출되는 문제가 보임

```javascript
import { useState } from 'react';

// ...

function App() {
  // highlight-start
  const _mode = useState('WELCOME') // 상태의 초기값
  const mode = _mode[0] // 상태의 값
  const setMode = _mode[1] // 상태를 변경할 수 있는 함수
  // const [mode, setMode] = useState('WELCOME') // 이렇게 축약가능
  // highlight-end
  let content = null;
  if(mode === 'WELCOME') {
    content = <Article title="WELCOME" body="Hello, WEB"></Article>
  } else if(mode === 'READ') {
    content = <Article title="READ" body="is ..."></Article>
  }
  return (
    <div className="App">
      <Header title="WEB" onChangeMode={() => {
        // highlight-next-line
        setMode('WELCOME')
      }}></Header>
      <Nav topics={topics} onChangeMode={(_id) => {
        // highlight-next-line
        setMode('READ')
      }}></Nav>
      {content}
    </div>
  );
}

export default App;
```

사용자 선택에 의해 렌터링이 바뀌는 케이스
1. 사용자가 nav 의 링크를 선택
2. props.onChangeMode 호출
3. onChangeMode 에 등록된 setMode, setId 실행
4. app.js 의 함수에 화면 렌더링

```javascript
import { useState } from 'react';

function Nav(props) {
  const lis = props.topics.map(topic => {
    return <li key={topic.id}><a id={topic.id} href={`/read/${topic.id}`} onClick={event => {
      event.preventDefault()
      // highlight-start
      props.onChangeMode(Number(event.target.id))
      // highlight-end
    }}>{topic.title}</a></li>
  })
  return (
    <nav>
      <ol>
        {lis}
      </ol>
    </nav>
  )
}

function Article(props) {
  return (
    <article>
      <h2>{props.title}</h2>
      {props.body}
    </article>
  )
}

function App() {
  const _mode = useState('WELCOME') // 상태의 초기값
  const [mode, setMode] = useState('WELCOME')
  const [id, setId] = useState(null)
  const topics = [
    { id:1, title:'HTML', body:'HTML is ...' },
    { id:2, title:'CSS', body:'CSS is ...' },
    { id:3, title:'javascript', body:'javascript is ...' }
  ]
  let content = null;
  if(mode === 'WELCOME') {
    content = <Article title="WELCOME" body="Hello, WEB"></Article>
  } else if(mode === 'READ') {
    const selTopic = topics.find(topic => {
      return topic.id === id
    })
    content = <Article title={selTopic.title} body={selTopic.body}></Article>
  }
  return (
    <div className="App">
      <Header title="WEB" onChangeMode={() => {
        setMode('WELCOME')
        }}></Header>
      <Nav topics={topics} onChangeMode={(_id) => {
        // highlight-start
        setMode('READ')
        setId(_id)
        // highlight-end
      }}></Nav>
      {content}
    </div>
  );
}

export default App;
```

### 8. Create

```javascript
// ...

// highlight-start
function Create(props) {
  return (
    <article>
      <h2>Create</h2>
      <form onSubmit={event => {
        event.preventDefault()
        const title = event.target.title.value
        const body = event.target.body.value
        props.onCreate(title, body)
      }}>
        <p><input type="text" name="title" placeholder="title" /></p>
        <p><textarea name="body" placeholder="body"></textarea></p>
        <p><input type="submit" value="Create"></input></p>
      </form>
    </article>
  )
}
// highlight-end

function App() {
  // ...
  if(mode === 'WELCOME') {
    // ...
  } else if(mode === 'CREATE') {
    // highlight-start
    content = <Create onCreate={(title, body) => {
      const newTopic = {
        id: nextId,
        title: title,
        body: body
      }
      const newTopics = [...topics, newTopic]
      setTopics(newTopics)
      setMode('READ')
      setId(nextId)
      setNextId(nextId+1)
    }}></Create>
    // highlight-end
  }
  return (
    <div className="App">
      // ...
      // highlight-start
      <a href="/create" onClick={event => {
        event.preventDefault()
        setMode('CREATE')
      }}>Create</a>
      // highlight-end
    </div>
  );
}

export default App;
```

### 9. Update

```javascript
// ...

// highlight-start
function Update(props) {
  const [title, setTitle] = useState(props.title)
  const [body, setBody] = useState(props.body)
  return (
    <article>
      <h2>Update</h2>
      <form onSubmit={event => {
        event.preventDefault()
        const title = event.target.title.value
        const body = event.target.body.value
        props.onUpdate(title, body)
      }}>
        <p><input type="text" name="title" placeholder="title" value={title} onChange={event=>{
          setTitle(event.target.value)
        }}/></p>
        <p><textarea name="body" placeholder="body" value={body} onChange={event=>{
          setBody(event.target.value)
        }}></textarea></p>
        <p><input type="submit" value="Update"></input></p>
      </form>
    </article>
  )
}
// highlight-end

function App() {
  
  // ...
  
  if(mode === 'WELCOME') {
    // ...
  } else if(mode === 'UPDATE') {
    // highlight-start
    const selTopic = topics.find(topic => {
      return topic.id === id
    })
    content = <Update title={selTopic.title} body={selTopic.body} onUpdate={(title, body)=>{
      console.log('id', selTopic.id)
      console.log('title', title)
      console.log('body', body)
      const newTopics = [...topics]
      const index = newTopics.findIndex(topic=> {
        return topic.id === selTopic.id
      })
      newTopics[index] = {
        id: selTopic.id,
        title: title,
        body: body
      }
      setTopics(newTopics)
    }}></Update>
    // highlight-end
  }

  return (
    <div className="App">
      <Header title="WEB" onChangeMode={() => {
        setMode('WELCOME')
        }}></Header>
      <Nav topics={topics} onChangeMode={(_id) => {
        setMode('READ')
        setId(_id)
      }}></Nav>
      {content}
      <ul>
        <li>
          <a href="/create" onClick={event => {
            event.preventDefault()
            setMode('CREATE')
          }}>Create</a>
        </li>
        {contextControl}
      </ul>
    </div>
  );
}

export default App;
```

### 10.Delete

```javascript
// ...

function App() {
  
  // ...
  
  if(mode === 'WELCOME') {
    content = <Article title="WELCOME" body="Hello, WEB"></Article>
  } else if(mode === 'READ') {
    // highlight-start
    const selTopic = topics.find(topic => {
      return topic.id === id
    })
    content = <Article title={selTopic.title} body={selTopic.body}></Article>
    contextControl = <>
      <li>
        <a href={"/update/"+id} onClick={event=>{
          event.preventDefault()
          setMode('UPDATE')
        }}>Update</a>
      </li>
      <li>
        <button onClick={()=>{
          const newTopics = topics.filter(topic=>{
            return topic.id !== id
          })
          setTopics(newTopics)
          setMode('WELCOME')
        }}>Delete</button>
      </li>
    </>
    // highlight-end
  } else if(mode === 'CREATE') {
    // ...
  }

  return (
    <div className="App">
      <Header title="WEB" onChangeMode={() => {
        setMode('WELCOME')
        }}></Header>
      <Nav topics={topics} onChangeMode={(_id) => {
        setMode('READ')
        setId(_id)
      }}></Nav>
      {content}
      <ul>
        <li>
          <a href="/create" onClick={event => {
            event.preventDefault()
            setMode('CREATE')
          }}>Create</a>
        </li>
        {contextControl}
      </ul>
    </div>
  );
}

export default App;
```
