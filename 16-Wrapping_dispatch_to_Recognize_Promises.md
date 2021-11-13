# 16. Обертывание `dispatch()` для распознавания промисов
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-recognize-promises)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/16-wrapping-dispatch-to-recognize-promises)

Сам по себе 'кшен криэйтор `receiveTodos` не очень полезен, так как каждый раз, когда мы его вызываем, мы хотим сначала получить `todos`. Поскольку `fetchTodos` и `receiveTodos` принимают одни и те же аргументы, было бы хорошо сгруппировать этот код в один экшен криэйтор.

#### Существующий `fetchData()` внутри `VisibleTodoList`
```javascript
fetchData() {
  const { filter, receiveTodos } = this.props;
  fetchTodos(filter).then(todos =>
    receiveTodos(filter, todos)
  );
}
```

### Рефакторинг наших экшен криэйтеров

Начнем с импорта фейкового API в наш файл экшен криэйтеров (`src/actions/index.js`).

`import * as api from '../api'`

Теперь добавим асинхронный экшен криэйтор с именем `fetchTodos`. Он принимает `filter` в качестве аргумента, а затем вызывает с ним метод API `fetchTodos`.

Я использую `then` - метод промиса для преобразования результата промиса из `response` в объект экшена, сгенерированного `receiveTodos` с учетом `filter` и `response`.

#### Внутри `src/actions/index.js`
```javascript
export const fetchTodos = (filter) =>
  api.fetchTodos(filter).then(response =>
    receiveTodos(filter, response)
  );
```

`receiveTodos` возвращает объект экшена синхронно, но `fetchTodos` возвращает промис, который разрешается (resolves) посредством объекта экшена.

Теперь можно не экспортировать `receiveTodos`  из наших экшен криэйторов, так как можем изменить компоненты, чтобы напрямую использовать `fetchTodos`.

### Обновление `VisibleTodoList`

Вернувшись в наш файл компонента, мы можем использовать проп `fetchTodos`, введенный `connect`'ом. Это соответствует новому, только что написанному асинхронному экшен криэйтеру `fetchTodos`.

Теперь можно удалить `import { fetchTodos } from '../api'`, так как с этого момента будем использовать экшен криэйтор `fetchTodos`, который вводится в пропсы с помощью `connect`.

```javascript
fetchData() {
  const { filter, fetchTodos } = this.props;
  fetchTodos(filter);
}
```

### Вспоминая то, что мы только что сделали...

Экшен криэйтор `fetchTodos` вызывает функцию `fetchTodos` из API, но затем преобразует ее результат в экшен Redux'а, созданный `receiveTodos`'ом.

Но Redux, по дефолту, позволяет диспатчить только простые объекты, а не промисы. Мы можем научить его распознавать промисы, используя тот же трюк, который мы использовали в `addLoggingToDispatch()` внутри `configureStore.js` файла (напомним, что функция `addLoggingToDispatch` принимает `dispatch`  из `store` и возвращает новую версию `dispatch`, который регистрирует каждый экшен и `state`).

### Добавление поддержки промисов

Внутри `configureStore.js` мы создадим функцию `addPromiseSupport()`, которая принимает `store` и возвращает версию `dispatch`, что поддерживает промисы.

Сначала мы возьмем функцию `rawDispatch`, которая определена в `store`, чтобы мы могли вызвать ее позже. Мы возвращаем функцию, которая имеет тот же API, что и диспатч функция, то есть она выполняет экшен.

```javascript
const addPromiseSupportToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    if (typeof action.then === 'function') {
      return action.then(rawDispatch);
    }
    return rawDispatch(action);
  };
};
```

Поскольку мы не знаем, является ли `action` реальным экшеном или промисом экшена, мы проверяем, есть ли у него метод `then`, являющийся функцией. Если да, то мы знаем, что это промис. Если экшен является промисом, мы ждем, пока он разрешится в объект экшена, который мы передадим посредством `rawDispatch`.

В противном случае мы просто вызовем `rawDispatch` сразу с полученным `action` объектом.

Наша новая функция `addPromiseSupportToDispatch` позволяет нам диспатчить как экшены, так и промисы, которые разрешаются в экшенах.

Чтобы закончить, нам нужно вызвать нашу новую функцию еще раз, прежде чем возвращать стор в приложение.

```javascript
// At the bottom of `configureStore.js`
const configureStore = () => {
  const store = createStore(todoApp);

  if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
  }

  store.dispatch = addPromiseSupportToDispatch(store);

  return store;
};
```

Если мы запустим приложение сейчас, мы все равно увидим, что экшен `'RECEIVE_TODOS'`  диспатчится, когда ответ будет готов. Но компонент использует более удобный API, который инкапсулирует асинхронную логику в асинхронном экшен криэйторе.

### Порядок имеет значение

Помните, что важен порядок, в котором мы переопределяем функцию `dispatch` внутри `configureStore`.

Если мы изменим его так, что вызовем `addPromiseSupportToDispatch` до `addLoggingToDispatch`, первым будет напечатан экшен, а затем будет обработан промис.

Это даст нам тип экшена `undefined`, и мы увидим промис вместо экшена, что не очень полезно.

[Резюме с 4:15 видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-recognize-promises)


<p align="center">
<a href="./15-Dispatching_Actions_with_the_Fetched_Data.md"><- Предыдущая</a>
<a href="./17-The_Middleware_Chain.md">Следующая -></a>
</p>