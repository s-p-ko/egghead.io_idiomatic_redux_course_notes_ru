# 11. Нормализация состояния формы (Normalizing the State shape)
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/11-normalizing-the-state-shape)

В настоящее время мы представляем `todos` в дереве состояний как массив  `todo` объектов. Однако в реальном приложении у нас, вероятно, будет более одного массива, и `todos` с одинаковыми `id` в разных массивах могут не синхронизироваться.

#### `todos.js` до
```javascript
const todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        todo(undefined, action),
      ];
    case 'TOGGLE_TODO':
      return state.map(t =>
        todo(t, action)
      );
    default:
      return state;
  }
};
```

### Рефакторинг `todos.js`

Мы должны рассматривать наше состояние как базу данных, поэтому мы собираемся сохранить `todos` в объекте, индексируемом по `id`.

Мы начнем с переименования редюсера в `byId`. Теперь, вместо того, чтобы добавлять новый элемент в конец или отображать каждый элемент, мы изменим значение в таблице поиска.

Теперь и `TOGGLE_TODO`, и `ADD_TODO` имеют одинаковую логику. Мы хотим вернуть новую таблицу поиска, в которой значение `action.id` будет результатом вызова редюсера для предыдущего значения `action.id` и `action`.

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

Это все еще композиция редюсера, но с объектом вместо массива.

---
**ПРИМЕЧАНИЕ**:
Мы используем оператор Object Spread (`...state`). Это не часть ES6, поэтому нам нужно установить плагин Babel `transform-object-rest-spread` и добавить его в наш файл `.babelrc`, чтобы это работало.
---

Каждый раз, когда редюсер `byId` получает экшен, он возвращает копию своего сопоставления между идентификаторами (`id`s) и актуальным задачами (todos) с обновленным `todo` для текущего экшена.

Теперь мы добавим еще один редюсер, который отслеживает все добавленные идентификаторы (`id`s).

### Добавление `allIds` редюсера

Теперь, когда мы сохраняем сами задачи в `byId` карте (map), у нас будет состояние этого редюсера как массив идентификаторов (`id`s).

Этот редюсер включит тип экшена, и единственный экшен, который меня заботит, - это `'ADD_TODO'`, потому что, если добавляется новый todo, мы хотим вернуть новый массив идентификаторов (`id`s) с новым `id` в качестве последнего элемента.

Для любых других экшенов нам просто нужно вернуть текущее состояние (которое является текущим массивом идентификаторов (`id`s)).

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

Нам по-прежнему нужно экспортировать единственный редюсер из файла `todos.js`, поэтому мы снова используем `combineReducers()`, чтобы объединить `byId` и `allIds` редюсеры.

```javascript
const todos = combineReducers({
  byId,
  allIds,
});
```

---
_Примечание_: вы можете использовать объединенный (combined) редюсеры сколько угодно раз. Вам не обязательно использовать его только на редюсере верхнего уровня. Фактически, очень часто по мере роста вашего приложения вы будете использовать `combineReducers` в нескольких местах.

---

### Обновление `getVisibleTodos` селектора

Теперь, когда мы изменили форму состояния (state shape) в наших редюсерах, нам также нужно обновить селекторы, которые на нее полагаются.

`state` объект в  `getVisibleTodos` теперь будет содержать поля `byId` и `allIds`, потому что он соответствует `state` объединенного (combined) редюсера.

Поскольку мы больше не используем массив todos, мы напишем селектор `getAllTodos`, чтобы создать массив для нас.

`getAllTodos` возьмет текущее `state` и вернет все `todos`, сопоставив `allIds` с таблицей поиска `byId` в `state` (the `state`'s `byId` lookup table).

Мы не будем экспортировать `getAllTodos`, потому что он будет использоваться только в текущем файле.

```javascript
const getAllTodos = (state) =>
  state.allIds.map(id => state.byId[id]);
```

Мы будем использовать этот новый селектор внутри нашего `getVisibleTodo` селектора, чтобы получить массив todos, которые можно фильтровать.

`allTodos` - это массив todos, который ожидают компоненты, поэтому мы можем вернуть его из селектора и не беспокоиться об изменении кода компонента.

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

### Извлечение `todo` редюсера


Файл `todos.js` немного вырос, так что сейчас самое время извлечь todo редюсер, что управляет одним todo, в отдельный файл.

Мы создадим файл с именем `todo.js` в той же папке `src/reducers` и вставим его в реализацию. Теперь мы можем импортировать его в `todos.js` файл.

#### `todo.js`
```javascript
const todo = (state, action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        id: action.id,
        text: action.text,
        completed: false,
      };
    case 'TOGGLE_TODO':
      if (state.id !== action.id) {
        return state;
      }
      return {
        ...state,
        completed: !state.completed,
      };
    default:
      return state;
  }
};

export default todo;
```

#### `todos.js`
```javascript
import { combineReducers } from 'redux';
import todo from './todo';

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

const allIds = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.id];
    default:
      return state;
  }
};

const todos = combineReducers({
  byId,
  allIds,
});

export default todos;

const getAllTodos = (state) =>
  state.allIds.map(id => state.byId[id]);

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

[Резюме с 3:54 видео](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)


<p align="center">
<a href="./10-Colocating_Selectors_with_Reducers.md"><- Предыдущая</a>
<a href="./12-Wrapping_dispatch_to_Log_Actions.md">Следующая -></a>
</p>
