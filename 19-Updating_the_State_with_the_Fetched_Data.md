# 19. Обновление состояния с помощью полученных данных
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data#/tab-transcript)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/19-updating-state-with-fetched-data)

В текущей реализации `getVisibleTodos` внутри `todos.js` все todos хранятся в памяти. У нас есть массив всех идентификаторов (`id`s), и мы получаем массив todos, которые мы можем фильтровать в соответствии с фильтром, переданным из React Router.

#### Текущий `getVisibleTodos`
```javascript
export const getVisibleTodos = (state, filter) => {
  const allTodos = getAllTodos(state);
  switch (filter) {
    case 'all':
      return allTodos;
    case 'completed':
      return allTodos.filter(t => t.completed);
    case 'active':
      return allTodos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

Однако это работает правильно только в том случае, если все данные с сервера уже доступны в клиенте, что обычно не относится к приложениям, которые что-то извлекают. Если у нас есть тысячи todos на сервере, было бы непрактично извлекать их все и фильтровать на клиенте.

### Рефакторинг `getVisibleTodos`

Вместо того, чтобы хранить один большой список идентификаторов (`id`s), мы будем вести список идентификаторов (`id`s) для каждой вкладки фильтра, чтобы их можно было хранить отдельно и заполнять в соответствии с действиями с извлеченными данными.

Мы удалим селектор `getAllTodos`, потому что у нас не будет доступа к `allTodos`. Нам также больше не нужно фильтровать на клиенте, потому что мы будем использовать список  todos, предоставленный сервером. Это означает, что мы можем удалить наш оператор `switch` из текущей реализации.

Вместо чтения из `state.allIds` мы будем читать IDs (идентификаторы) из `state.IdsByFilter[filter]`. Затем мы отмапим идентификаторы (`id`s) с таблицей поиска `state.ById`, чтобы получить актуальные todos.

#### Обновлено `getVisibleTodos`
```javascript
export const getVisibleTodos = (state, filter) => {
  const ids = state.idsByFilter[filter];
  return ids.map(id => state.byId[id]);
};
```

### Рефакторинг `todos`

Селектор теперь ожидает, что `idsByFilter` и `byId` будут частью объединенного `state` (состояния) `todos`редюсера.

#### `todos` редюсер до:
```javascript
const todos = combineReducers({
  byId,
  allIds
});
```

`todos` редюсер использовался для объединения (combine) таблицы поиска и списка `allIds`. Однако теперь мы заменим список `allIds` списком `idsByFilter`, который станет новым комбинированным редюсером.

#### `todos` редюсер после:
```javascript
const todos = combineReducers({
  byId,
  idsByFilter
});
```

### Создание `idsByFilter`

`idsByFilter` объединяет отдельный список идентификаторов (`id`s) для каждого фильтра. Таким образом, `allIds` это для `all` фильтра, `activeIds` - для `active` фильтра и `completedIDs` - для `completed` фильтра.

```javascript
const idsByFilter = combineReducers({
  all: allIds,
  active: activeIds,
  completed: completedIds,
});
```

### Обновление `allIds` редюсера

Первоначальный `allIDs` редюсер управлял массивом идентификаторов (IDs) и `ADD_TODO` экшеном.

Мы собираемся сейчас снять эту ответственность, потому что сейчас мы хотим научить его реагировать на данные, полученные с сервера.


#### `allIds` до:
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default:
      return state;
  }
};
```

Начнем с переименования `ADD_TODO` в `RECEIVE_TODOS`. Чтобы обработать `RECEIVE_TODOS` экшен, мы хотим вернуть новый массив todos, который мы получим из ответа сервера. Мы отмапим этот новый массив todos с функцией, которая просто выбирает `id` из `todo`. Напомним, мы решили хранить все идентификаторы (IDs) отдельно от активных идентификаторов и завершенных идентификаторов, чтобы они извлекались полностью независимо.

#### `allIds` после:
```javascript
const allIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

#### Создание `activeIds` редюсера

Наш `activeIds` редюсер также будет отслеживать массив идентификаторов (`id`s), но только для `todos` на активной вкладке. Нам нужно будет обработать действие `RECEIVE_TODOS` точно так же, как и `allIds` редюсер до него.

```javascript
const activeIds = (state = [], action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

### Обновление правильного массива

И `activeIds`, и `allIds` должны возвращать новый `state`, когда срабатывает `RECEIVE_TODOS` экшен, но нам нужен способ указать, какой массив `id` мы должны обновить.

Если вы вспомните `RECEIVE_TODOS` экшен, вы можете вспомнить, что мы передали `filter` как часть экшена. Это позволяет нам сравнивать `filter` внутри экшена с `filter`, соответствующим редюсеру.

Редюсер `allIds` интересуется только экшинами с `all` фильтром, а `activeIds` интересуется только `active` фильтром.

#### `activeIds` редюсер
```javascript
const activeIds = (state = [], action) => {
  if (action.filter !== 'active') {
    return state;
  }
  // rest of code as above
```
_Повторите для `allIds`, но не забудьте заменить `active` на `all`_

### Создание `completedIds` редюсера

Этот редюсер такой же, как и другие наши редюсеры фильтров, но для `complete` фильтра.

```javascript
const completedIds = (state = [], action) => {
  if (action.filter !== 'completed') {
    return state;
  }
  switch (action.type) {
    case 'RECEIVE_TODOS':
      return action.response.map(todo => todo.id);
    default:
      return state;
  }
};
```

### Обновление `byId` редюсера

Теперь, когда у нас есть редюсеры, управляющие идентификаторами (`id`s), нам нужно обновить редюсер `byId`, чтобы он действительно обрабатывал новые `todos` из ответа.

#### `byId` до:
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'ADD_TODO':
    case 'TOGGLE_TODO':
      return {
        ...state,
        [action.id]: todo(state[action.id], action),
      };
    default:
      return state;
  }
};
```

Мы можем начать с удаления существующих `case`s, потому что данные больше не хранятся локально. Вместо этого мы будем обрабатывать `RECEIVE_TODOS` экшен только в других редюсерах.

Затем мы создадим `nextState`, неглубокую копию объекта `state`, который соответствует таблице поиска. Мы хотим перебрать каждый объект `todo` в `response` и поместить его в наш `nextState`.

Мы заменим все, что находится в записи `nextState` для `todo.id`, новым `todo`, которое мы только что получили.

Наконец, мы вернем следующую версию таблицы поиска из нашего редюсера.

#### `byId` после:
```javascript
const byId = (state = {}, action) => {
  switch (action.type) {
    case 'RECEIVE_TODOS': // eslint-disable-line no-case-declarations
      const nextState = { ...state };
      action.response.forEach(todo => {
        nextState[todo.id] = todo;
      });
      return nextState;
    default:
      return state;
  }
};
```

**Примечание:** Обычно операция присваивания является мутацией. Однако в этом случае это нормально, потому что `nextState` является неглубокой копией, и мы присваиваем только один уровень глубины. Наша функция остается чистой, потому что мы не изменяем ни один из объектов исходного состояния.

### Заканчивая

В качестве последнего шага мы можем удалить импорт `todo.js`, а также сам файл из нашего проекта, потому что логика добавления и переключения todos будет реализована в виде API вызовов к серверу на будущих уроках.

[Резюме на 5:22 в видео](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data)


<p align="center">
<a href="./18-Applying_Redux_Middleware.md"><- Предыдущая</a>
<a href="./20-Refactoring_the_Reducers.md">Следующая -></a>
</p>