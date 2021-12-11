# 26. Нормализация API ответов с помощью `normalizr`
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/26-normalizing-json-responses-with-normalizr)

Редюсер `byId` в настоящее время должен по-разному обрабатывать действия сервера, потому что они имеют разные формы ответа.

Например, ответ экшена  `'FETCH_TODOS_SUCCESS'` - это массив todos. Этот массив нужно перебирать и объединять по одному в следующее состояние.

Ответ для `'ADD_TODO_SUCCESS'` - это единственная todo, которая была только что добавлена, и эту единственную todo нужно объединить другим способом.

Вместо добавления новых случаев для каждого нового вызова API я хочу нормализовать ответы, чтобы форма ответа всегда была одинаковой.

### Установка `normalizr`

`normalizr` - это служебная библиотека, которая поможет нам нормализовать ответы API, чтобы они имели одинаковую форму.

`$ npm install --save normalizr`

###  Создание `schema.js`

Мы создадим новый файл `schema.js` внутри нашего каталога `actions`.

Мы начнем с импорта конструктора `Schema` и функции с именем `arrayOf` из `normalizr`.

Наша первая экспортированная схема будет для объектов `todo`, и мы укажем `todos` как имя словаря в нормализованном ответе.

Наша следующая схема под названием `arrayOfTodos` соответствует ответам, которые содержат массивы объектов `todo`.

```javascript
import { schema } from 'normalizr'

export const todo = new schema.Entity('todos');
export const arrayOfTodos = new schema.Array(todo);
```

### Обновление наших экшен криэйтеров

Внутри файла `actions/index.js` мы добавим именованный импорт для функции с именем `normalize`, которую мы импортируем из `normalizr`. Мы также добавляем импорт пространства имен для всех схем, которые мы определили в файле схемы.

Внутри колбэка `FETCH_TODOS_SUCCESS`  мы добавим log "normalized response", чтобы я мог видеть, как выглядит нормализованный ответ. Мы вызываем функцию `normalize` с исходным `response` в качестве первого аргумента и соответствующей схемой (в данном случае `arrayOfTodos`) в качестве второго аргумента.

```javascript
return api.fetchTodos(filter).then(
  response => {
    console.log(
      'normalized response',
      normalize(response, schema.arrayOfTodos)
    )
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
```

Мы обновим `addTodo` аналогичным образом:
```javascript
export const addTodo = (text) => (dispatch) =>
  api.addTodo(text).then(response => {
    console.log(
      'normalized response',
      normalize(response, schema.todo)
    )
    dispatch({
      type: 'ADD_TODO_SUCCESS',
      responsed,
    });
  });
```

### Сравнение ответов

На этом этапе ответ в экшене представляет собой массив todo объектов, но наш нормализованный ответ для `'FETCH_TODOS_SUCCESS'` - это объект, содержащий два поля:  `entities` и `result`.

`entity` содержит нормализованный словарь под названием` todos`, который содержит каждое `todo` в ответе по его id. `normalizr` нашел эти `todo` объекты в ответе, следуя схеме `arrayOfTodos`. Удобно, что они индексируются по IDs'ам, поэтому их будет легко объединить в таблицу поиска.

Второе поле - это `result`, который представляет собой массив IDs'ов `todo`. Они находятся в том же порядке, что и задачи в исходном массиве ответов. Однако `normalizr` заменяет каждое `todo` своим ID и перемещает каждое todo в словарь `todos`.

### Завершение обновлений наших экшен криэйтеров

Теперь мы изменим экшен криэйтеры, чтобы они передавали нормализованный ответ в поле ответа вместо исходного ответа.

##### До:
```javascript
return api.fetchTodos(filter).then(
  response => {
    console.log(
      'normalized response',
      normalize(response, schema.arrayOfTodos)
    )
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response,
    });
  },
```

##### После:
```javascript
return api.fetchTodos(filter).then(
    dispatch({
      type: 'FETCH_TODOS_SUCCESS',
      filter,
      response: normalize(response, schema.arrayOfTodos),
    });
  },
```

### Обновление редюсеров

Мы можем удалить особые случаи в нашем `byId` редюсере, потому что форма ответа нормализована. Вместо переключения по типу экшена мы проверим, есть ли у экшена объект ответа..

##### `byId` редюсер до:
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'FETCH_TODOS_SUCCESS': // eslint-disable-line no-case-declarations
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    case 'ADD_TODO_SUCCESS':
      return {
        ...state,
        [action.response.id]: action.response,
      };
    default:
      return state;
  }
};
```
Мы вернем новую версию таблицы поиска, которая содержит все существующие записи, а также любые записи внутри `entity.todos` в нормализованном ответе. Для других экшенов я верну таблицу поиска как есть.

##### `byId` редюсер после:
```javascript
const byId = (state = {}, action) => {
  if (action.response) {
    return {
      ...state,
      ...action.response.entities.todos,
    };
  }
  return state;
};
```

Теперь нам нужно обновить `ids` редюсер внутри `createList.js` для нашей новой формы `action.response`.

##### `ids` редюсер до:
```javascript
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

Теперь ответ экшена имеет поле `result`, которое является либо массивом `id`s'ов (в случае с `'FETCH_TODOS_SUCCESS'`), либо одним  `id` выбранной todo (в случае с `'ADD_TODO_SUCCESS'`).

##### `ids` редюсер после
```javascript
const ids = (state = [], action) => {
   switch (action.type) {
     case 'FETCH_TODOS_SUCCESS':
       return filter === action.filter ?
         action.response.result :
         state;
     case 'ADD_TODO_SUCCESS':
       return filter !== 'completed' ?
         [...state, action.response.result] :
         state;
     default:
       return state;
   }
 };
```

[Демонстрация и подведение итогов на 5:33 видео](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)


<p align="center">
<a href="./25-Creating_Data_on_the_Server.md"><- Предыдущая</a>
<a href="./27-Updating_Data_on_the_Server.md">Следующая -></a>
</p>