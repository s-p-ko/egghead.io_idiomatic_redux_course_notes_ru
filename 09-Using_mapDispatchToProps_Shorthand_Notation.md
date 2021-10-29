# 09. Использование сокращенной записи `mapDispatchToProps()`
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/09-using-mapdispatchtoprops-shorthand-notation)

Функция `mapDispatchToProps` позволяет вставлять определенные пропсы в компонент React'а, который может отправлять действия (_диспатчить экшины_). Например, компонент `TodoList` вызывает свой `onTodoClick` - коллбэк проп -  с `id` `todo`.

#### Внутри `TodoList`
```javascript
      <Todo
        key={todo.id}
        {...todo}
        onClick={() => onTodoClick(todo.id)}
      />
```

Внутри `mapDispatchToProps` в нашем компоненте` VisibleTodoList` мы указываем, что когда `onTodoClick()` вызывается с `id`, мы хотим диспатчить (_т.е. отправить_) экшен `toggleTodo` с этим `id`. Экшен криэйтор `toggleTodo` использует этот `id` для создания объекта экшена, который будет отправлен.

#### `VisibleTodoList` `mapDispatchToProps`
```javascript
const mapDispatchToProps = (dispatch) => ({
  onTodoClick(id) {
    dispatch(toggleTodo(id));
  },
});

const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList));
```

Существует более короткий способ указать `mapDispatchToProps`, когда аргументы для коллбэк пропа в точности совпадают с аргументами экшен криэйтора.

Вместо того, чтобы передавать функцию, мы можем передать объектное сопоставление (object mapping) имен коллбэк пропов, которые мы хотим внедрить, и функций экшен криэйтора, которые создают соответствующие экшены.

Это довольно распространенный случай, поэтому часто вам не нужно писать `mapDispatchToProps`, а вместо этого передать эту карту (map) в объект.

#### `VisibleTodoList` после:
```javascript
const VisibleTodoList = withRouter(connect(
  mapStateToProps,
  { onTodoClick: toggleTodo }
)(TodoList));
```

[Резюме с 1:13 видео](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)

<p align="center">
<a href="./08-Using_withRouter_to_Inject_the_Params_into_Connected_Components.md"><- Предыдущая</a>
<a href="./10-Colocating_Selectors_with_Reducers.md">Следующая -></a>
</p>
