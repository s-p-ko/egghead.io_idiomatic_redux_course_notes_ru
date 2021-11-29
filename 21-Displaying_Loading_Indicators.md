# 21. Отображение индикаторов загрузки
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/21-displaying-loading-indicators)

Когда данные извлекаются асинхронно, мы хотим показать пользователю какую-то визуальную подсказку. Мы добавим к функции `render` условие, которое гласит, что если мы получаем данные и у нас нет задач для отображения, мы вернем индикатор загрузки из функции `render` `VisibleTodoList`'а.

Мы возьмем `todos` и `isFetching` из `props`. Поскольку `todos` - единственный дополнительный проп, который нам нужно передать списку, то вместо использования спред оператора, мы просто передадим `todos` напрямую.

Наша `mapStateToProps` функция уже вычисляет `visibleTodos` и включает `todos` в пропсы. Нам нужно сделать нечто подобное для `isFetching`. `getIsFetching` принимает текущее состояние приложения и `filter` для извлекаемых `todos`. Он объявлен вместе с другими селекторами верхнего уровня в файле редюсера верхнего уровня.

#### Внутри `VisibleTodoList.js`
```javascript
render() {
    const { isFetching, toggleTodo, todos } = this.props;
    if (isFetching && !todos.length) {
      return <p>Loading...</p>;
    }

    return (
      <TodoList
        todos={todos}
        onTodoClick={toggleTodo}
      />
    );
  }

  /// PropType Declarations...

  const mapStateToProps = (state, { params }) => {
    const filter = params.filter || 'all';
    return {
      isFetching: getIsFetching(state, filter),
      todos: getVisibleTodos(state, filter),
      filter,
    };
  };
```

### Обновление корневого редюсера

Перейдя к файлу корневого редюсера (`src/reducers/index.js`), мы добавим еще одну экспортированную именованную функцию селектора для `getIsFetching`. Она принимает `state` и `filter`» в качестве аргументов и делегирует другому селектору определение, выбирается ли список в данный момент.

Мы передадим state этого списка из `state.listByFilter`, но мы еще не написали `getIsFetching`.

```javascript
export const getIsFetching = (state, filter) =>
  fromList.getIsFetching(state.listByFilter[filter]);
```

### Обновление `createList.js`

Перед созданием нашего нового селектора `getIsFetching` нам нужно изменить форму состояния списка. Вместо того, чтобы предполагать, что `state` является массивом `id`s'ов, мы предположим, что это объект, который содержит этот массив как свойство.

Теперь мы можем добавить еще один селектор под названием `getIsFetching`, который читает `state.isFetching`.

```javascript
export const getIds = (state) => state.ids;
export const getIsFetching = state => state.isFetching;
```

Мы хотим, чтобы наш редюсер отслеживал оба этих поля, поэтому вместо того, чтобы усложнять существующий редюсер `createList`, мы переименуем его в `ids`, потому что он управляет только идентификаторами (`id`s'ами).

### Создание `isFetching`

Во-первых, в верхней части нашего файла нам нужно добавить импорт для утилиты `combReducers` из Redux:

```javascript
import { combineReducers } from 'redux';
```

Теперь мы можем создать редюсер `isFetching`, который будет управлять только флагом `isFetching`  `state`'а.

Его начальное состояние - `false`, и он выглядит так же, как любой другой редюсер. Мы включаем `action` `type`, и если это `'REQUEST_TODOS'`, мы вернем `true`, потому что мы начали выборку.

Если это `'RECEIVE_TODOS'`, мы вернем `false`, потому что операция завершена. Для любого неизвестного действия мы вернем текущй `state`.

Мы включим то же условие, что и наш `ids` редюсер, который игнорирует любые действия с фильтром, который не соответствует аргументу `filter` для `createList`.

#### Внутри `createList.js`
```javascript
const isFetching = (state = false, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'REQUEST_TODOS':
        return true;
      case 'RECEIVE_TODOS':
        return false;
      default:
        return state;
    }
  };
```

Обратите внимание, что мы обрабатываем `REQUEST_TODOS` экшен, но никуда его не диспатчим.

### Добавление 'REQUEST_TODOS' экшен  криэйтера

В нашем файле экшен криэтора (`src/actions/index.js`) мы добавим новую экспортируемую функцию с именем `requestTodos`, которая принимает `filter` и возвращает объект экшена, описывающий `'REQUEST_TODOS'` экшен с соответствующим фильтром

```javascript
export const requestTodos = (filter) => ({
  type: 'REQUEST_TODOS',
  filter,
});
```

Каждый экспортируемый экшен криэйтор будет доступен в `props` `VisibleToDoList` компонента.

### Обновление `fetchData` внутри `VisibleToDoList`

Мы можем деструктурировать`requestTodos` из `props` и вызвать его прямо перед запуском асинхронной операции `fetchToDos`.

```javascript
fetchData() {
  const { filter, fetchTodos, requestTodos } = this.props;
  requestTodos(filter);
  fetchTodos(filter);
}
```

[Резюме с 3:51 в видео](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)

<p align="center">
<a href="./20-Refactoring_the_Reducers.md"><- Предыдущая</a>
<a href="./22-Dispatching_Actions_Asynchronously_with_Thunks.md">Следующая -></a>
</p>
