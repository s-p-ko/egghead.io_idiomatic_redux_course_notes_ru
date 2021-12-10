# 25. Создание данных на сервере
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/25-creating-data-on-the-server)

### Обновление фейкового API

Некоторые новые функции были добавлены в наш фейковый API клиент для следующих нескольких уроков.

Первый новый фейковый эндпоинт (конечная точка) API называется `addTodo`. Он имитирует сетевое соединение, а затем создает новый объект `todo`. Он помещает todo с заданным текстом в фейковую базу данных и возвращает объект `todo`, как это обычно делают REST эндпоинты.

#### `addTodo` эндпоинт
```javascript
export const addTodo = (text) =>
  delay(500).then(() => {
    const todo = {
      id: v4(),
      text,
      completed: false,
    };
    fakeDatabase.todos.push(todo);
    return todo;
  });
```

Второй фейковый эндпоинт API я назвал `toggleTodo`. Он также имитирует сетевое соединение, находит соответствующий todo в фейковой базе данных и меняет его поле `completed`, также возвращая  `todo` в качестве ответа.

#### `toggleTodo` эндпоинт
```javascript
export const toggleTodo = (id) =>
  delay(500).then(() => {
    const todo = fakeDatabase.todos.find(t => t.id === id);
    todo.completed = !todo.completed;
    return todo;
  });
```

### Обновление "Add Todo" процесса

В этом уроке мы создадим кнопку добавления todo, которая вызывает фейковый `addTodo` эндпоинт API.

Нам больше не понадобится функция `v4` из `node-uuid` внутри нашего файла экшен криэйтора, потому что генерация идентификатора теперь будет происходить на сервере.

`addTodo` экшен криэйтор будет изменен на thunk экшен криэйтор, поэтому мы добавим `dispatch` в качестве аргумента. Thunk вызовет эндпоинт `addTodo` API с заданным текстом и будет ждать ответа.

Когда ответ доступен, он отправит действие с типом `'ADD_TODO_SUCCESS'` и ответ сервера.

##### Обновленный `addTodo` экшен
```javascript
export const addTodo = (text) => (dispatch) =>
  api.addTodo(text).then(response => {
    dispatch({
      type: 'ADD_TODO_SUCCESS',
      response,
    });
  });
```

#### Обновление `byId` редюсера

Поскольку недавно добавленное todo будет частью ответа сервера, нам нужно изменить редюсер `byId`, чтобы объединить todo с таблицей поиска, которой он управляет.

Я добавляю новый случай для `'ADD_TODO_SUCCESS'` экшена. Мы воспользуемся спрэд оператором для объекта, чтобы создать новую версию таблицы поиска, где под ключом идентификатора ответа на экшен есть новый todo объект, который я прочитал из ответа действия.

```javascript
// Inside reducers/byId.js
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'FETCH_TODOS_SUCCESS':
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    case 'ADD_TODO_SUCCESS': // Our new case
      return {
        ...state,
        [action.response.id]: action.response,
      };
    default:
      return state;
  }
};
```

Однако мы не обновляли список по фильтру, поэтому у всех по-прежнему есть только три IDs'а в списке. Если я перейду на другую вкладку, появится новое todo, потому что его  ID теперь включен в список извлеченных IDs'ов, и аналогично, если я вернусь к предыдущей вкладке, он появится теперь, потому что данные были повторно извлечены.

#### Обновление `createList`

Поскольку список IDs'ов для каждой вкладки управляется редюсером, определенным внутри `createlist.js`, нам нужно обновить наш `ids` редюсер для обработки `'ADD_TODO_SUCCESS'` экшена.

Когда мы получаем подтверждение того, что todo был добавлен на сервер, мы можем вернуть новый список IDs'ов с существующими IDs'ами в начале и вновь добавленным ID в конце.

В отличие от других экшенов, `'ADD_TODO_SUCCESS'` не имеет свойства `filter` для `action` объекта, наша текущая проверка `if (filter !== action.filter)`  внутри `ids` завершится ошибкой. По этой причине мы заменим существующую проверку на разные проверки в разных случаях.

Для `FETCH_TODOS_SUCCESS` мы хотим заменить выбранные IDs'ы, если фильтр в экшене соответствует фильтру этого списка.

Однако для `ADD_TODO_SUCCESS` мы хотим, чтобы вновь добавленная todo отображалась в каждом списке, кроме завершенного фильтра, потому что вновь добавленная todo не завершена.

```javascript
const createList = (filter) => {
  const ids = (state = [], action) => {
    switch (action.type) {
      case 'FETCH_TODOS_SUCCESS':
        return filter === action.filter ?
          action.response.map(todo => todo.id) :
          state;
      case 'ADD_TODO_SUCCESS':
        return filter !== 'completed' ?
          [...state, action.response.id] :
          state;
      default:
        return state;
    }
  };
```

[Демонстрация и подведение итогов с 3:36 на видео](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)


<p align="center">
<a href="./24-Displaying_Error_Messages.md"><- Предыдущая</a>
<a href="./26-Normalizing_API_Responses_with_normalizr.md">Следующая -></a>
</p>