# 07. Фильтрация Redux State c React Router Params
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/07-filtering-redux-state-with-react-router-params)

Теперь мы используем `Link`и React Router'а, поэтому, когда мы кликаем ссылку обновляется URL. Однако контент не обновляется, потому что видимый компонент `TodoList` в своей функции `mapStateToProps` все еще зависит от `visibilityFilter` в хранилище Redux вместо чтения его из URL.

#### `mapStateToProps` до:
```javascript
const mapStateToProps = (state) => ({
  todos: getVisibleTodos(
    state.todos,
    state.visibilityFilter
  )
})
```

Чтобы исправить это, мы добавим аргумент `ownProps` и прочитаем `visibilityFilter` из `ownProps`.

#### `mapStateToProps` после:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.filter
  )
})
```

Мы также обновим нашу функцию `getVisibleTodos`, чтобы использовать наше текущее соглашение для пропсов `'all'`, `'completed'` и `'after'`.

#### `getVisibleTodos` до:
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos;
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed);
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

#### `getVisibleTodos` после:
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'all':
      return todos;
    case 'completed':
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

## Обновление `App.js`

Поскольку компонент `VisibleTodoList` рендерится из приложения, нам нужно добавить проп `filter`, чтобы он был доступен в функции `mapStateToProps`  компонента `VisibleTodoList`.

Мы хотим, чтобы проп filter соответствовал текущему параметру `filter` в нашей конфигурации маршрута (`path='/(:filter)'`, которую мы установили в предыдущем уроке). React Router делает эти параметры доступными для компонента обработчика маршрута (the route handler component) в специальном пропе `params`, поэтому мы добавим `params` в качестве пропа в `App`. Теперь мы сможем читать фильтр из `params.filter`.

Поскольку параметр `filter` пуст в корневом пути, мы передадим `'all'` в `VisibleTodoList` как запасной вариант.

```javascript
const App = ({ params }) => (
  <div>
    <AddTodo />
    <VisibleTodoList
      filter={params.filter || 'all'}
    />
    <Footer />
  </div>
);
```

## `App.js` (react-router v4.0.0 или выше)
```javascript
const App = ({ match }) => (
  <div>
    <AddTodo />
    <VisibleTodoList
      filter={match.params.filter || 'all'}
    />
    <Footer />
  </div>
);
```

Теперь, когда наши фильтры видимости управляются React Router, нам больше не нужен `visibilityFilter` редюсер. Мы можем удалить его, а также убрать его из объявления `combineReducers()` в `index.js`.

[Резюме со 2:20 видео](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)


<p align="center">
<a href="./06-Navigating_with_React_Router_Link.md"><- Предыдущая</a>
<a href="./08-Using_withRouter_to_Inject_the_Params_into_Connected_Components.md">Следующая -></a>
</p>
