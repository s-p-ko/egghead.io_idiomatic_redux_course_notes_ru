# 02\. Обеспечение начального состояния

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/02-supplying-the-initial-state)

Когда вы создаете Redux store (встречаются термины "стор", "хранилище"), его начальное состояние определяется корневым редюсером (root reducer).

В этом случае корневой редюсер является результатом вызова `todos` и `visibilityFilter` в `combineReducers`.

##### _Внутри `src/index.js`_

```javascript
const store = createStore(todoApp)
```

##### _Внутри `src/reducers/index.js`_

```javascript
const todoApp = combineReducers({
  todos,
  visibilityFilter,
})
```

Поскольку редюсеры автономны, каждый из них указывает собственное начальное состояние (initial state) как дефолтное значение аргумента `state`.

В `const todos = (state = [], action) ...` дефолтное значение state - пустой массив.

Дефолтное значение state в `const visibilityFilter = (state = 'SHOW_ALL', action) ...` - строка.

Следовательно, нашим начальным состоянием объединенного редюсера будет объект, содержащий пустой массив под ключом `todos` и строку `'SHOW_ALL'` под ключом `visibilityFilter` :

##### Начальное состояние:

```javascript
const todoApp = combineReducers({
  todos,
  visibilityFilter,
})
```

Однако иногда нам нужно синхронно загрузить данные в хранилище (store) до запуска приложения.

Например, с предидущей сессии могут остаться Todos.

Redux позволяет передавать `persistedState` как второй аргумент функции `createStore`:

```javascript
const persistedState = {
  todos: [
    {
      id: 0,
      text: "Welcome Back!",
      completed: false,
    },
  ],
}

const store = createStore(todoApp, persistedState)
```

При передаче в `persistedState` перезапишутся значения по умолчанию, установленные в редюсере. В этом примере мы снабдили `todos` массивом, но, поскольку мы не указали значение для `visibilityFilter`, то по умолчанию будет использовано `'SHOW_ALL'`.

Поскольку `persistedState`, который мы передали в качестве второго аргумента в `createStore()`, был получен из самого Redux'а, инкапсуляция редюсеров не нарушается.

Может возникнуть соблазн указать начальное состояние для всего вашего хранилища (store) внутри `persistedState`, но это не рекомендуется. Если бы вы это сделали, было бы труднее тестировать и менять ваши редюсеры позже.

#### [Резюме с 1:42 видео](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)

<p align="center">
  <a href="./01-Simplifying_the_Arrow_Functions.md"><- Предыдущая</a>
  <a href="./03-Persisting_the_State_to_the_Local_Storage.md">Следующая -></a>
</p>
