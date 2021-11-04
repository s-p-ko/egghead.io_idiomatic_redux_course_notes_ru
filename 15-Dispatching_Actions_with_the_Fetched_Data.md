# 15. Диспатчинг экшенов с полученными данными
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-dispatching-actions-with-the-fetched-data?series=building-react-applications-with-idiomatic-redux)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/15-dispatching-actions-with-fetched-data)

Продолжая с того места, на котором мы остановились, внутри `VisibleTodoList` мы извлечем общий код между хуками жизненного цикла в отдельный метод `fetchData`, где данные, которые мы хотим получить, зависят только от фильтра.

Мы вызываем этот метод, чтобы получить исходные данные из хука `componentDidMount()`. Мы также будем вызывать его всякий раз, когда `filter` изменяется внутри хука жизненного цикла `componentDidUpdate`.

#### `VisibleTodoList.js`
```javascript
class VisibleTodoList extends Component {
  componentDidMount() {
    this.fetchData();
  }

  componentDidUpdate(prevProps) {
    if (this.props.filter !== prevProps.filter) {
      this.fetchData();
    }
  }

  fetchData() {
    fetchTodos(this.props.filter).then(todos =>
      console.log(this.props.filter, todos)
    );
  }
  .
  .
  .
```

### Обновление `fetchData()`
Мы хотим, чтобы полученные todos стали частью `state` Redux стора, и единственный способ получить что-то в состояние (state) - это диспатчить экшен.

Мы заставим его вызывать коллбэк проп `receiveTodos` с только что полученными `todos`.

```javascript
fetchData() {
  fetchTodos(this.props.filter).then(todos =>
    this.props.receiveTodos(todos)
  );
}
```

Чтобы сделать его доступным внутри компонента, нам нужно передать функцию с именем `receiveTodos`, которая будет экшен криэйтером во втором аргументе `connect`. Поскольку имя функции совпадает с именем коллбэк пропа, мы можем использовать более короткую ES6 нотацию свойства объекта 

Мы также импортируем `receiveTodos` из файла, в котором определены все остальные экшен криэйтеры.

```javascript
// At the top of `VisibleTodoList.js`
import { toggleTodo, receiveTodos } from '../actions'

...

// At the bottom of `VisibleTodoList.js`
VisibleTodoList = withRouter(connect(
  mapStateToProps,
  { onTodoClick: toggleTodo, receiveTodos }
)(VisibleTodoList))
```

### Реализация `receiveTodos`

Теперь нам нужно реализовать `receiveTodos`

Внутри файла экшен крийтера (`src/actions/index.js`) мы создаем новый экспорт функции `receiveTodos`, которая принимает `response` сервера в качестве аргумента и возвращает объект с `type` (типом) `'RECEIVE_TODOS'` и `response` в виде поля.

#### Внутри `src/actions/index.js`
```javascript
export const receiveTodos = (response) => ({
  type: 'RECEIVE_TODOS',
  response,
});
```

Редюсеры, обрабатывающие этот экшен, должны будут знать, какому фильтру соответствует ответ, поэтому мы добавим `filter` в качестве аргумента в экшен криэйтер `receiveTodos` и передадим его как часть экшен объекта.

```javascript
export const receiveTodos = (filter, response) => ({
  type: 'RECEIVE_TODOS',
  filter,
  response,
});
```

### Обновление `VisibleTodoList` компонента с помощью фильтра

Вернувшись в `VisibleTodoList`, нам нужно обновить `fetchData`, чтобы передать фильтр через экшен криэйтора.

Мы используем ES6 синтаксис деструктуризации, чтобы получить `filter` и `receiveTodos` из `props`. Важно, сразу же деструктурировть `filter`, так как к моменту срабатывания коллбэка, `this.props.filter` мог измениться, потому что пользователь мог уйти.

#### Внутри `VisibleTodoList`
```javascript
fetchData() {
  const { filter, receiveTodos } = this.props;
  fetchTodos(filter).then(todos =>
    receiveTodos(filter, todos)
  );
}
```

### Пишем меньше шаблонов

По мере перемещения по приложению, компонент извлекает данные из своих хуков жизненного цикла и диспатчит, когда данные готовы, экшены Redux. Теперь давайте будем писать меньше шаблонов.

Мы можем начать с замены именованного импорта импортом пространства имен. Это означает, что любая функция, экспортированная из файла экшена, будет в объекте с именем `actions`, который мы передадим в качестве второго аргумента для connect'а.

```javascript
// At the top of `VisibleTodoList`

// Before: import { toggleTodo, receiveTodos } from '../actions'
import * as actions from '../actions' // After
.
.
.
// At the bottom of `VisibleTodoList`
VisibleTodoList = withRouter(connect(
  mapStateToProps,
  // Before: { onTodoClick: toggleTodo, receiveTodos}
  actions // After
)(VisibleTodoList))

```
 
Внутри функции `render()` `VisibleTodoList`'а мы деструктурируем `props`'ы, потому что экшен криэйтеру `toggleTodo` нужно передать имя пропа `onTodoClick`, но остальные пропсы могут быть переданы как есть.

Объект `...rest` содержит теперь все пропсы, кроме `toggleTodo`, поэтому мы их пропустим. Мы также передадим `toggleTodo` как проп `onTodoClick`, потому что этого ожидает компонент `TodoList`.

```javascript
// Inside of `VisibleTodoList`
  render() {
    const { toggleTodo, ...rest } = this.props;
    return (
      <TodoList
        {...rest}
        onTodoClick={toggleTodo}
      />
    );
  }
}
```

[Резюме с 3:30 видео](https://egghead.io/lessons/javascript-redux-dispatching-actions-with-the-fetched-data?series=building-react-applications-with-idiomatic-redux)


<p align="center">
<a href="./14-Fetching_Data_on_Route_Change.md"><- Предыдущая</a>
<a href="./16-Wrapping_dispatch_to_Recognize_Promises.md">Следующая -></a>
</p>