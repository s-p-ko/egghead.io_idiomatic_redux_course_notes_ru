# 08. Использование `withRouter()` для вставки параметров в подключенные компоненты
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/08-using-withrouter-to-inject-params-into-connected-components)

Сейчас мы читаем `params.filter`, передаваемый Router'ом внутри компонента `App`. Мы можем получить доступ к параметрам оттуда, потому что маршрутизатор внедряет проп `params` в любой компонент обработчика маршрута (Route handler component), указанный в конфигурации маршрута. В этом случае `filter` передается внутрь `params`.

Но, сам компонент `App` не использует `filter`, а передает его в `VisibleTodoList`, для вычисления им текущего видимого Todos.

Передача `params` с верхнего уровня обработчиков маршрутов (Route Handlers) становится утомительной, поэтому удалим проп `filter` и найдем вместо этого способ прочитать параметры текущего маршрутизатора в самом `VisibleTodoList`.


#### `App.js` до:
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

#### `App.js` после:
```javascript
const App = () => (
  <div>
    <AddTodo />
    <VisibleTodoList />
    <Footer />
  </div>
);
```

## Обновление  `VisibleTodoList`
Начнем с импорта `withRouter` из React Router (version 3+):

`import { withRouter } from 'react-router'`

`withRouter` принимает компонент React и возвращает другой компонент React, который внедряет связанные с маршрутизатором пропсы, такие как `params`, в ваш компонент.

Мы хотим, чтобы `params` был доступен внутри `mapStateToProps`, поэтому нужно обернуть результат в  `connect()`  так, чтобы подключенный компонент получил `params` в качестве пропа.

Также нужно будет изменить `mapStateToProps`, чтобы читать `filter` прямо из `ownProps.params`, а не из `ownProps`.

Как и раньше, укажем запасное значение для `'all'`.

#### `VisibleTodoList` до:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.filter
  ),
});

// mapDispatchToProps ...

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList);
```

#### `VisibleTodoList` после:
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.params.filter || 'all'),
});

// mapDispatchToProps ...

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

`mapStateToProps` можно сделать еще более компактным читая `params` прямо из определения аргумента благодаря синтаксису деструктуризации ES6:

```javascript
const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all'),
});
```

_Примечание: объяснения в видео требуют версии `react-router` до 4.0.0. Начиная с этой версии были внесены некоторые изменения, требующие немного другого синтаксиса:_

#### `VisibleTodoList` после (react-router v4.0.0 или выше):
```javascript
const mapStateToProps = (state, ownProps) => ({
  todos: getVisibleTodos(
    state.todos,
    ownProps.match.params.filter || 'all'),
});

// mapDispatchToProps ...

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

`mapStateToProps` можно сделать короче, читая `params` прямо из определения аргумента благодаря синтаксису деструктуризации ES6:

```javascript
const mapStateToProps = (state, { match }) => ({
  todos: getVisibleTodos(state.todos, match.params.filter || 'all'),
});
```

[Резюме с 1:51 видео](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components#/tab-transcript)


<p align="center">
<a href="./07-Filtering_Redux_State_with_React_Router_Params.md"><- Предыдущая</a>
<a href="./09-Using_mapDispatchToProps_Shorthand_Notation.md">Следующая -></a>
</p>