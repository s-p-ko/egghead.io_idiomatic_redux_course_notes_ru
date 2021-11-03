# 14. Получение данных при изменении роута (маршрута)
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-fetching-data-on-route-change)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/14-fetching-data-on-route-change)

Для начала удалим из `index.js` (нашей точки входа) тестовый `fetchTodos` API вызов. Потому, что хотим получить todos из компонента `VisibleTodoList`.

Начнем с импорта `fetchTodos` в `VisibleTodoList.js`

`import { fetchTodos } from '../api';`


Компонент `VisibleTodoList` генерируется `connect` и `withRouter` вызовами, каждый из которых генерирует промежуточный компонент, внедряющий свойства.

Хорошее место для вызова `fetchTodos` API было бы внутри `componentDidMount()` - хука жизненного цикла. Хотя мы не можем игнорировать хуки жизненного цикла сгенерированных компонентов! Это означает, что нам нужно создать новый React компонент.

### Создание нового React компонента

Внутри `VisibleTodoList.js` мы импортируем `React` и `Component` - базовый класс React'а. Затем мы объявим классовый компонент React'а назвав его `VisibleTodoList`, который расширяет базовый класс `Component`.

```javascript
import React, { Component } from 'react';
// other imports...

class VisibleTodoList extends Component {
  render() {
    return <TodoList {...this.props} />;
  }
}
.
.
.
```

Мы все еще, по-прежнему, хотим визуализировать презентационный компонент `TodoList`. Единственная цель добавления этого нового класса - добавить хуки жизненного цикла (lifecycle hooks). Любые пропсы будут переданы в `TodoList`.

Теперь, когда `VisibleTodoList` определен как вышеуказанный класс, мы не можем объявить другую константу с тем же именем, поэтому мы переназначаем привязку `VisibleTodoList`, чтобы она указывала на обернутый компонент. Мы также изменим вызов `connect()`, чтобы вместо него обернуть наш новый класс.

```javascript
VisibleTodoList = withRouter(connect(
  mapStateToProps,
  { onTodoClick: toggleTodo }
)(VisibleTodoList));

export default VisibleTodoList;
```

Компонент, сгенерированный вызовом `connect()`, будет рендерить класс `VisibleTodoList`, который мы определили. Результатом вызовов `connect` и `withRouter` обертки является окончательный компонент `VisibleTodoList`, который мы экспортируем из файла.

### Добавление хуков жизненного цикла

Когда компонент монтируется, мы хотим получить todos для текущего фильтра.

Будет удобно иметь фильтр напрямую доступным в качестве пропа, поэтому мы изменим `mapStateToProps`, чтобы вычислить `filter` из `params`, как это делалось и раньше, но мы также передадим его как одно из свойств `return` объекта. Таким образом, мы получим теперь  и `todos`, и сам `filter` внутри  `VisibleTodoList` компонента.

#### Обновление `mapStateToProps`
```javascript
const mapStateToProps = (state, { params }) => {
  const filter = params.filter || 'all';
  return {
    todos: getVisibleTodos(state, filter),
    filter,
  };
};
```

_Примечание: объяснения в видео требуют версии `react-router` до 4.0.0. Начиная с этой версии были внесены некоторые изменения, требующие немного другого синтаксиса:_

#### Обновление `mapStateToProps` (react-router v4.0.0 или выше)
```javascript
const mapStateToProps = (state, { match }) => {
  const filter = match.params.filter || 'all';
  return {
    todos: getVisibleTodos(state, filter),
    filter,
  };
};
```

Возвращаясь к методу жизненного цикла, мы можем использовать `this.props.filter` внутри `componentDidMount`. Когда todos получены, `fetchTodos` возвращает промис. Мы можем использовать метод `then` для доступа к решенным (сделанным) `todos` и регистрировать текущий `filter` и `todos`, которые мы только что получили от фейкового бэкэнда.

#### Реализация `componentDidMount`
```javascript
class VisibleTodoList extends Component {
  componentDidMount() {
    fetchTodos(this.props.filter).then(todos =>
      console.log(this.props.filter, todos)
    );
  }
```

В нашей текущей реализации при запуске приложения будет отображаться печатаемый фильтр `all` и соответствующие `todos`.

Но при изменении фильтров ничего не произойдет, потому что `componentDidMount` запускается только один раз. Чтобы исправить это, нам нужно добавить второй хук жизненного цикла `componentDidUpdate`.

#### Реализация `componentDidUpdate`
```javascript
// inside the `VisibleTodoList` below `componentDidMount()`
componentDidUpdate(prevProps) {
   if (this.props.filter !== prevProps.filter) {
     fetchTodos(this.props.filter).then(todos =>
       console.log(this.props.filter, todos)
     );
   }
 }
```

`componentDidUpdate` принимает как аргумент предыдущие пропсы. Затем мы сравниваем текущее и предыдущее значения фильтра. Если текущий фильтр не такой как предыдущиий, мы вызываем `fetchTodos()` для текущего фильтра.

[Резюме с 3:36 видео](https://egghead.io/lessons/javascript-redux-fetching-data-on-route-change)


<p align="center">
<a href="./13-Adding_a_Fake_Backend_to_the_Project.md"><- Предыдущая</a>
<a href="./15-Dispatching_Actions_with_the_Fetched_Data.md">Следующая -></a>
</p>