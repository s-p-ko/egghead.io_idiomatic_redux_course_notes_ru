# 04. Рефакторинг точки входа

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux)

В этом уроке мы извлечем логику, необходимую для создания хранилища и подписки на него, чтобы сохранить состояние в отдельном файле.

#### `index.js` До:

```javascript
import "babel-polyfill"
import React from "react"
import { render } from "react-dom"
import { Provider } from "react-redux"
import { createStore } from "redux"
import throttle from "lodash/throttle"
import todoApp from "./reducers"
import App from "./components/App"
import { loadState, saveState } from "./localStorage"

const persistedState = loadState()
const store = createStore(todoApp, persistedState)

store.subscribe(
  throttle(() => {
    saveState({
      todos: store.getState().todos,
    })
  }, 1000)
)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
)
```

Мы назовем новый файл `configureStore.js`, и начнем с создания функции `configureStore`, которая будет содержать логику создания и сохранения хранилища.

Мы делаем это, потому что в этом случае нашему приложению не нужно точно знать, как создается хранилище (store) и есть ли у нас обработчики подписки. Оно может просто использовать возвращенное хранилище (store) в файле `index.js`.

#### `configureStore.js`

```javascript
import { createStore } from "redux"
import throttle from "lodash/throttle"
import todoApp from "./reducers"
import { loadState, saveState } from "./localStorage"

const configureStore = () => {
  const persistedState = loadState()
  const store = createStore(todoApp, persistedState)

  store.subscribe(
    throttle(() => {
      saveState({
        todos: store.getState().todos,
      })
    }, 1000)
  )

  return store
}

export default configureStore
```

Экспортируя `configureStore` вместо просто `store`, мы сможем создать столько экземпляров хранилища для тестирования, сколько захотим.

#### `index.js` После:

```javascript
import "babel-polyfill"
import React from "react"
import { render } from "react-dom"
import Root from "./components/Root"
import configureStore from "./configureStore"

const store = configureStore()
render(<Root store={store} />, document.getElementById("root"))
```

Обратите внимание, что мы также извлекли корневой отрендеренный элемент в отдельный компонент `Root`. Он принимает `store` как проп (prop) и будет определен в отдельном файле в нашей папке `src/components`.

---

#### `Root` Component

```javascript
import React, { PropTypes } from "react"
import { Provider } from "react-redux"
import App from "./App"

const Root = ({ store }) => (
  <Provider store={store}>
    <App />
  </Provider>
)

Root.propTypes = {
  store: PropTypes.object.isRequired,
}

export default Root
```

Мы определили функциональный компонент без сохранения состояния, который просто принимает `store` как проп и возвращает `<App />` внутри `Provider`'а `response-redux`'а.

Теперь внутри `index.js` мы можем удалить наш импорт `Provider`'а, равно как и заменить импорт компонента `App` импортом компонента `Root`.

[Резюме с 1:45 видео.](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux#/tab-transcript)

<p align="center">
<a href="./03-Persisting_the_State_to_the_Local_Storage.md"><- Предидущая</a>
<a href="./05-Adding_React_Router_to_the_Project.md">Следующая -></a>
</p>
