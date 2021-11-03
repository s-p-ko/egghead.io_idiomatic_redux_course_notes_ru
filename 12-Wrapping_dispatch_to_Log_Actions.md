# 12. Обёртываем `dispatch()` в логи экшенов

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/12-wrapping-dispatch-to-log-actions)

Теперь, когда наша форма состояния (state shape) более сложная, мы хотим переопределить функцию `store.dispatch` добавив несколько  `console.log()` операторов, чтобы можно было видеть, как на состояние влияют экшены.

Начнем с создания новой функции с именем `addLoggingToDispatch`, которая принимает `store` в качестве аргумента. Она обернет предоставленный Redux'ом `dispatch`, поэтому она прочитает необработанный диспатч из `store.dispatch`.

`addLoggingToDispatch()` вернет другую функцию с той же сигнатурой, что и диспатч, который является аргумнтом единственного экшена. Некоторые браузеры, такие как Chrome, поддерживают использование `console.group()` для группировки нескольких log операторов под одним заголовком, и мы передаем `action.type`, чтобы сгруппировать несколько log'ов по типу экшена.

Мы будем логировать предыдущее `state` перед диспатчем экшенов вызывая `store.getState()`. Затем мы залогируем сам экшен, чтобы видеть, какой экшен вызывает изменение.

Чтобы сохранить контракт `dispatch` функции в точности, мы объявим `returnValue` и вызовем  `rawDispatch()` функцию с экшеном. Теперь вызывающий код не сможет отличить нашу функцию от функции, предоставляемой Redux'ом, за исключением того, что мы также логгируем (регистрируем) некоторую информацию.

Мы также логируем следующее состояние, потому что стор (store) гарантированно обновится после вызова диспатча. Мы можем использовать `store.getState()` для получения следующего состояния после диспатча.

#### `configureStore.js`
```javascript
const addLoggingToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    console.group(action.type);
    console.log('prev state', store.getState());
    console.log('action', action);
    const returnValue = rawDispatch(action);
    console.log('next state', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  }
}

const configureStore = () => {
  const persistedState = loadState();
  const store = createStore(todoApp, persistedState);

  store.dispatch = addLoggingToDispatch(store);
  .
  .
  .
```


### Последние штрихи

Chrome предлагает API для стилизации `console.log()`'ов, поэтому мы можем добавить несколько цветов в наши логи. Эти модификации окрашивают предыдущее состояние в серый цвет, экшен - в синий, а следующее состояние - в зеленый.

```javascript
console.log('%c prev state', 'color: gray', store.getState());
console.log('%c action', 'color: blue', action);
const returnValue = rawDispatch(action);
console.log('%c next state', 'color: green', store.getState());
```

Если API `console.group()` доступен не во всех браузерах, мы просто возвращаем необработанный диспатч как есть.

```javascript
// At the top of `addLoggingToDispatch()` after `rawDispatch` is declared
if (!console.group) {
  return rawDispatch
}
```

Поскольку все логировать на продакшене - не лучшая идея, мы добавляем шлюз внутри `configureStore()`, указывая этим, что если `process.env.NODE_ENV` - не продакшен, то мы собираемся запустить этот код. В противном случае мы просто оставим стор как есть.

#### Inside `configureStore`
```javascript
const configureStore = () => {
    const persistedState = loadState();
    const store = createStore(todoApp, persistedState);

    if (process.env.NODE_ENV !== 'production') {
      store.dispatch = addLoggingToDispatch(store);
    }
    .
    .
    .
```

Логирование экшена, который мы диспатчим вместе с состоянием до и после диспатча, позволяет легко находить любые ошибки.

[Резюме со 2:18 видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)


<p align="center">
<a href="./11-Normalizing_the_State_Shape.md"><- Предыдущая</a>
<a href="./13-Adding_a_Fake_Backend_to_the_Project.md">Следующая -></a>
</p>