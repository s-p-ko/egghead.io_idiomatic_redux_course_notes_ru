# 23. Избегайте состояний гонки с помощью Thunk
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/23-dispatching-actions-conditionally-with-thunks)

Когда мы увеличиваем задержку в нашем фэйквом API клиенте до пяти секунд, мы замечаем проблему. Мы не проверяем, загружается ли уже вкладка перед запуском запроса, а потом возвращается множество `receiveTodos` экшенов, что может привести к состоянию гонки.

Чтобы исправить это, мы можем досрочно выйти из `fetchTodos` экшен криэйтера, если уже знаем, что получаем todos для данного фильтра..

Внутри `fetchTodos` мы добавим `if`, чтобы проверить, выполняем ли мы сейчас выборку (fetching), используя наш `getIsFetching` селектор, который принимает состояние стора и фильтр в качестве аргументов. Если он вернет true, мы выйдем из thunk'а раньше, без диспатчинга каких-либо экшенов.

#### Обновление `fetchTodos`
```javascript
export const fetchTodos = (filter) => (dispatch) => {
  if (getIsFetching(getState(), filter)) {
    return;
  }
  /// rest of fetchTodos
```

Селектор `getIsFetching` определяется внутри файла редюсера верхнего уровня, поэтому нам нужно импортировать его как  `import` from `reducers`.

```javascript
import { getIsFetching } from '../reducers';
```

Мы также используем `getState()`, который не определен в этом файле. Он принадлежит объекту `store`, но у нас нет доступа к нему напрямую от экшен криэйтера.

### Обновление `thunk` мидлвара

Мы можем сделать так, чтобы мидлвар `thunk` внутри `configureStore.js` внедрял не только `store.dispatch()` функцию внутри thunk экшенов, но также `store.getState` функцию. Таким образом, мы можем получить его как второй аргумент после `dispatch` внутри `thunk` экшен криэйтера.

```javascript
// Inside configureStore.js
const thunk = (store) => (next) => (action) =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action);
```

```javascript
// Add `getState` as a second parameter to `fetchTodos`
export const fetchTodos = (filter) => (dispatch, getState) => {
```

С этими изменениями `fetchTodos` экшен криэйтор диспатчит экшены  условно. Если мы запустим приложение, мы не сможем заставить его делать более трех одновременных запросов (по одному для каждого типа фильтра).

Флаг `isFetching` сбрасывается только тогда, когда возвращаются соответствующие `receiveTodos` экшены, и тогда мы можем запросить новые todos. Это хороший способ избежать ненужных сетевых операций и потенциальных состояний гонки.

### Обновление `fetchTodos`

Поскольку возвращаемое значение thunk'а является промисом, мы изменим наш ранний `return` на промис, который разрешается немедленно. Нам делать этого не обязательноо, но для вызывающего кода это удобно.

#### Внутри `actions/index.js`
```javascript
export const fetchTodos = (filter) => (dispatch, getState) => {
  if (getIsFetching(getState(), filter)) {
    return Promise.resolve();
  }
  // rest of fetchTodos
```

Сам `thunk` мидлвар не использует этот промис, но он становится значением  `return`'а для диспатчинга этого экшен криэйтера, поэтому мы можем использовать его внутри компонента чтобы запланировать некоторый код после завершения асинхронного действия.

#### Внутри `VisibleTodoList`
```javascript
fetchData() {
  const { filter, fetchTodos } = this.props;
  fetchTodos(filter).then(() => console.log('done!'));
}
```

### Introducing `redux-thunk`

`redux-thunk` является мидлваром, аналогичным тому, что мы только что реализовали. Чтобы установить его, запустите

`npm install --save redux-thunk`.

Установив `redux-thunk`, мы можем удалить только что написанную версию thunk мидлвара и вместо этого импортировать `thunk` из `redux-thunk`.

[Резюме со 2:52 в видео](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)


<p align="center">
<a href="./22-Dispatching_Actions_Asynchronously_with_Thunks.md"><- Предыдущая</a>
<a href="./24-Displaying_Error_Messages.md">Следующая -></a>
</p>