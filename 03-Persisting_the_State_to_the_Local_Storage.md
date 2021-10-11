# 03. Сохранение состояния в Local Storage

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage)

В этом примере мы хотим иметь возможность сохранять состояние приложения с помощью `localStorage` API браузера.

Мы напишем функцию `loadState()` и новую модель `localStorage.js`.

Функция `loadState` будет искать в `localStorage` по ключу, извлекать строку и пытаться парсить ее как JSON. Этот код нужно заключить в `try/catch`, так как браузер пользователя может запрещать использование `localStorage` API.

#### `localStorage.js`

```javascript
export const loadState = () => {
  try {
    const serializedState = localStorage.getItem("state")
    if (serializedState === null) {
      return undefined
    }
    return JSON.parse(serializedState)
  } catch (err) {
    return undefined
  }
}
```

У нашей функции `loadState`, если `serializedState` имеет значение `null`, это означает, что ключ не существует, поэтому мы возвращаем `undefined`, что позволяет самим редюсерам устанавливать состояние.

Однако, если строка `serializedState` существует, мы будем использовать `JSON.parse(serializedState)`, чтобы вернуть ее в `state` объекта.

Поскольку у нас есть функция для загрузки состояния, у нас должна быть функция для сохранения состояния в localStorage:

```javascript
export const saveState = (state) => {
  try {
    const serializedState = JSON.stringify(state)
    localStorage.setItem("state", serializedState)
  } catch (err) {
    // Ignore write errors.
  }
}
```

Функция `saveState` принимает `state` как аргумент и делает точно противоположную к `loadState` функции вещь.

Сначала мы используем `JSON.stringify(state)` для сериализации состояния (state) в строку. Это будет работать только в том случае, если состояние сериализуемо, но если вы следуете предлагаемым Redux'ом практикам, вам незачем беспокоится.

Мы отлавливаем любые ошибки, поскольку либо `JSON.stringify()`, либо `localStorage.setItem()` может завершиться ошибкой.

### Использование `localStorage.js`

Вернитесь в наш `index.js` файл, в нём мы импортируем только что написанные функции:

```javascript
import { loadState, saveState } from "./localStorage"
```

Чтобы сохранять наше состояние в любое время, когда хранилище (`store`) изменяется, мы будем использовать его метод `subscribe()`, чтобы добавить слушателя, который будет вызываться при любом изменении состояния, передавая текущее состояние хранилища в функцию `saveState`:

```javascript
// Inside of index.js ...
store.subscribe(() => {
  saveState(store.getState())
})
```

Теперь наше состояние сохраняется при перезагрузках. Однако отслеживаются не только наши полные и неполные элементы Todo, но и фильтр видимости. Это не идеально, поскольку в большинстве случаев мы хотели бы сохранить только данные, а не UI (пользовательский интерфейс).

Чтобы исправить это, мы настроим `store.subscribe()`:

```javascript
store.subscribe(() => {
  saveState({
    todos: store.getState().todos,
  })
})
```

Теперь, когда мы сохраняем ту часть состояния, что относится к `todos`, при обновлении страницы наш `visibilityFilter` получит от своего редюсера дефолтное значение `'SHOW_ALL'`.

### Но у нас есть баг ...

С учетом того, как в настоящее время написан наш код, когда мы пытаемся добавить новый элемент Todo, React выдаст ошибку: `Encountered two children with the same key, 0` ("Встречены два дочерних элемента с одним и тем же ключом, 0").

Это потому, что в нашем компоненте `TodoList` мы используем `todo.id` в качестве ключа, который назначается в экшн криэйтере `addTodo` внутри actions.js. Экшн криэйтер `addTodo` использует локальную переменную `nextTodoId`, которой по умолчанию присвоено значение `0`.

##### `addTodo` До:

```javascript
let nextTodoId = 0

export const addTodo = (text) => ({
  type: "ADD_TODO",
  id: (nextTodoId++).toString(),
  text,
})
```

Чтобы обойти это, мы будем использовать npm модуль `node-uuid`:

`$ npm install --save node-uuid`

Чтобы использовать модуль, мы импортируем `v4` с `node-uuid` и вызовим его вместо `(nextTodoId++)`. _Примечание: `v4` - просто название стандарта._

##### `addTodo` После:

```javascript
import { v4 } from "node-uuid"

export const addTodo = (text) => ({
  type: "ADD_TODO",
  id: v4(),
  text,
})
```

Теперь все наши элементы Todo сохраняются во время перезагрузки.

### Троттлинг\* `saveState()`

##### \*_от англ._ "throttling"

Теперь мы вызываем `saveState()` внутри слушателя подписки, и он вызывается всякий раз при изменении состояния хранилища. Мы не хотим вызывать его слишком часто, потому что он использует дорогостоящую операцию `stringify`.

Исправим это с помощью другого модуля npm - `lodash`, который включает утилиту `throttle`.

`$ npm install --save lodash`

#### `store.subscribe()` До:

```javascript
store.subscribe(() => {
  saveState({
    todos: store.getState().todos,
  })
})
```

Обернув наш коллбэк в вызов `throttle`, мы можем гарантировать, что внутренняя функция, которую мы передаем, не будет вызываться чаще, чем указанное нами количество миллисекунд.

Мы импортируем только `throttle` прямо из ` lodash`, вместо того, чтобы использовать всю библиотеку для использования одной функции.

#### `store.subscribe()` После:

```javascript
// top of index.js
import throttle from 'lodash/throttle'
.
.
.
store.subscribe(throttle(() => {
  saveState({
    todos: store.getState().todos
  })
}, 1000))
```

Теперь, даже если стор обновляется очень быстро, у нас есть гарантия, что мы будем писать в `localStorage` не чаще одного раза в секунду.

#### [Резюме на 6:05 видео](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage#/tab-transcript)

<p align="center">
<a href="./02-Supplying_the_Initial_State.md"><- Предидущая</a>
<a href="./04-Refactoring_the_Entry_Point.md">Следующая -></a>
</p>
