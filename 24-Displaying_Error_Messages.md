# 24. Отображение Error Messages
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-displaying-error-messages)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/24-handling-network-errors)

Иногда API запросы терпят неудачу, и мы будем имитировать это, «бросками» (`throw`ing) в фэйковом API клиенте, чтобы он возвращал отклоненный промис. Если мы запустим приложение, индикатор загрузки зависнет, потому что для флага `isFetching` будет установлено значение `true`, но нет соответствующего `receiveTodos` экшена, чтобы снова вернуть его в значение `false`.

### Исправляем это

Мы начнем с некоторой чистки внутри файла экшен криэйтера (`actions/ index.js`).

`requestTodos` экшен никогда не используется вне `fetchTodos`, поэтому мы можем встроить литерал объекта `requestTodos` внутрь него. Мы можем сделать то же самое с `receiveTodos`, скопировав и вставив его в `fetchTodos`, куда он и диспатчиться. Мы также добавим обработчик отклонения в качестве второго аргумента в наш  `Promise.then` метод.

#### Внутри `fetchTodos`
```javascript
return api.fetchTodos(filter).then(
  response => {
    dispatch({
      type: 'RECEIVE_TODOS',
      filter,
      response,
    });
  },
  error => {
    // To be filled in
  }
);
```

### Переименование экшенов для ясности

Поскольку экшен криэйтор `fetchTodos` диспатчит несколько экшенов, мы переименуем их, чтобы они были более согласованными:
   * `'REQUEST_TODOS'` превращается в `'FETCH_TODOS_REQUEST'` для запроса todos
   * `'RECEIVE_TODOS'` превращается в `'FETCH_TODOS_SUCCESS'` для успешного получения todos
   * Добавьте `'FETCH_TODOS_FAILURE'` в наш обработчик `error`, когда не удается получить todos.

Нашему обработчику `error` также будут переданы два дополнительных блока данных: `filter` и `message`, которые можно прочитать с помощью `error.message`, если он указан. Мы будем использовать фразу `'Something went wrong.'` как запасной вариант.

#### Обновленный `fetchTodos` `return`
```javascript
return api.fetchTodos(filter).then(
  response => {
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
  error => {
    dispatch({
      type: 'FETCH_TODOS_FAILURE',
      filter,
      message: error.message || 'Something went wrong.',
    });
  }
);
```

Теперь наш экшен криэйтор `fetchTodos` обрабатывает все случаи, и мы можем удалить старые экшен криэйторы, которые теперь встроены (`requestTodos` и `receiveTodos`).

### Обновление наших редюсеров

Поскольку мы изменили типы экшенов, теперь нам нужно изменить соответствующие редюсеры.

Наш `ids` редюсер должен обрабатывать `FETCH_TODOS_SUCCESS` вместо `RECEIVE_TODOS`.

Редюсер `isFetching` должен обрабатывать `FETCH_TODOS_REQUEST` вместо `REQUEST_TODOS` и `FETCH_TODOS_SUCCESS` вместо `RECEIVE_TODOS`.

Мы также обработаем `FETCH_TODOS_FAILURE`, вернув `false`, чтобы наш индикатор загрузки не зависал.

Последний редюсер, который нам нужно изменить - это `byId`, где мы заменяем `RECEIVE_TODOS` на `FETCH_TODOS_SUCCESS`.

#### Обновленный `isFetching` редюсер
```javascript
const isFetching = (state = false, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'FETCH_TODOS_REQUEST':
        return true;
      case 'FETCH_TODOS_SUCCESS':
      case 'FETCH_TODOS_FAILURE':
        return false;
      default:
        return state;
    }
  };
```

С этими изменениями индикатор загрузки не будет зависать, потому что срабатывает соответствующий экшен сбоя, переводя `isFetching` обратно в значение `false`.

### Отображение Error

Мы создадим новый файл `FetchError.js` в директории `components` .

После `import`'а `React` мы создадим новый функциональный компонент `FetchError` без сохранения состояния, который будет принимать два пропа: строку `message` и функцию `onRetry`. Этот компонент будет экспортом по умолчанию для этого файла.

Отрендеренный `<div>` будет содержать ошибку, говорящую о том, что произошло что-то плохое (включая сообщение, переданное в пропсах), и кнопку, которая при нажатии будет вызывать коллбэк проп `onRetry`, чтобы пользователь мог повторить попытку получения данных.


##### `FetchError` компонент
```javascript
const FetchError = ({ message, onRetry }) => (
  <div>
    <p>Could not fetch todos. {message}</p>
    <button onClick={onRetry}>Retry</button>
  </div>
);
```

### Добавление `FetchError` в `VisibleTodoList`

Нам нужно `import FetchError` внутри `VisibleTodoList`, а затем обновить метод `render`.

Чтобы получить error message, нам нужно деструктурировать его из `props` компонента `VisibleTodoList`.

```javascript
// Inside VisibleTodoList
render() {
  const { isFetching, errorMessage, toggleTodo, todos } = this.props;
  ...
```

Мы также добавим еще одно условие внутри `render`, говорящее, что _если_ у нас есть сообщение об ошибке в наших пропсах и у нас нет todos для отображения, _то_ вернуть `FetchError` компонент.

Компонент `FetchError` сам требует проп `message`, которому будет передан проп `errorMessage`, который я только что деструктурировал. Мы также предоставим коллбэк проп `onRetry`, в котором мы передадим функцию ошибки, которая вызывает `this.fetchData` для перезапуска процесса выборки данных.

```javascript
// Inside VisibleTodoList's `render()`
if (errorMessage && !todos.length) {
     return (
       <FetchError
         message={errorMessage}
         onRetry={() => this.fetchData()}
       />
     );
   }
```

Нам нужно добавить `errorMessage` в `mapStateToProps` из `VisibleTodoList`, чтобы сделать его доступным. Следуя тому же шаблону, который используется с `isFetching`, мы получим проп `errorMessage` путем вызова селектора с именем `getErrorMessage` и передачи `state` приложения и `filter`.

```javascript
// Inside VisibleTodoList
const mapStateToProps = (state, { params }) => {
  const filter = params.filter || 'all';
  return {
    isFetching: getIsFetching(state, filter),
    errorMessage: getErrorMessage(state, filter),
    todos: getVisibleTodos(state, filter),
    filter,
  };
};
```

### Реализация `getErrorMessage`

Нам нужно добавить `getErrorMessage` к импорту редюсера `VisibleTodoList` в верхней части файла:

`import { getVisibleTodos, getErrorMessage, getIsFetching } from '../reducers';`

Теперь внутри нашего корневого файла редюсеров (/reducers/index.js) мы создаем селектор `getErrorMessage` с помощью копирования-вставки  и рефакторинга нашего `getIsFetching` селектора.

#### Создание селектора
```javascript
export const getErrorMessage = (state, filter) =>
  fromList.getErrorMessage(state.listByFilter[filter]);
```

#### Обновление `createList`

Внутри `createList.js` мы добавим новый экспортированный селектор `getErrorMessage`, который принимает состояние (state) списка и возвращает свойство, называемое error message.

```javascript
export const getErrorMessage = (state) => state.errorMessage;
```

Теперь объявим новый редюсер с именем `errorMessage` с начальным `state` -  `null`. Мы делаем это, потому что редюсер не может иметь начальное состояние undefined, поэтому мы должны сделать его отсутствие явным.

Как и в других редюсерах в этом файле, мы хотим пропустить любые действия с фильтром, которые не соответствуют `filter`, указанному в качестве аргумента `createList`. Когда фильтр _совпадает_, мы хотим обработать несколько экшенов:

  * Отображение сообщения об ошибке в случае сбоя
  * Удаление сообщения об ошибке, если пользователь повторяет свой запрос
  * Вернуть текущее состояние для любого другого экшена

Редюсер `errorMessage` также должен быть добавлен в `combineReducers`.

##### Завершенный `errorMessage` редюсер
```javascript
const errorMessage = (state = null, action) => {
    if (filter !== action.filter) {
      return state;
    }
    switch (action.type) {
      case 'FETCH_TODOS_FAILURE':
        return action.message;
      case 'FETCH_TODOS_REQUEST':
      case 'FETCH_TODOS_SUCCESS':
        return null;
      default:
        return state;
    }
  };

  return combineReducers({
    ids,
    isFetching,
    errorMessage,
  });
```

### Обновление API
Вместо того, чтобы наш API каждый раз выдавал ошибку, мы будем выбрасывать ее случайным образом, чтобы мы могли опробовать нашу кнопку «повторить попытку» ("retry" button).

[Демонстрация и резюме на 6:43 в видео](https://egghead.io/lessons/javascript-redux-displaying-error-messages)


<p align="center">
<a href="./23-Avoiding_Race_Conditions_with_Thunks.md"><- Предыдущая</a>
<a href="./25-Creating_Data_on_the_Server.md">Следующая -></a>
</p>