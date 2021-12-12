# 27. Обновление данных на сервере

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/27-updating-data-on-the-server)

Мы начнем с изменения экшен криэйтора `toggleTodo` на thunk экшен криэйтор, поэтому мы добавим `dispatch` в качестве аргумента каррирования. Затем мы вызовем эндпоинт API `toggleTodo` и дождемся ответа.

Когда ответ будет доступен, мы отправим (отдиспатчим) экшен с типом `'TOGGLE_TODO_SUCCESS'` и ответом. Мы снова будем использовать `normalize`, передав исходный `response` в качестве первого аргумента и схему todo в качестве второго аргумента.


##### Обновленный `toggleTodo` экшен криэйтор
```javascript
export const toggleTodo = (id) => (dispatch) =>
  api.toggleTodo(id).then(response => {
    dispatch({
      type: 'TOGGLE_TODO_SUCCESS',
      response: normalize(response, schema.todo),
    });
  });
```

### Обновление `ids` оредюсера

Внутри `createList.js` мы добавим новый случай для экшена `'TOGGLE_TODO_SUCCESS'`.

Мы извлечем код для этого случая в отдельную функцию под названием `handleToggle` и передадим ей `state` и `action`.

```javascript
// Inside the `ids` reducer
  case 'TOGGLE_TODO_SUCCESS':
    return handleToggle(state, action);
```

Мы поместим нашу функцию `handleToggle` над редюсером `ids`. Он принимает `state` (массив ids'ов) и экшен `'TOGGLE_TODO_SUCCESS'`.

Мы деструктурируем `result` как `toggledId` и `entities` из нормализованного ответа. Затем мы прочитаем значение `completed` из `todo`, которое я получаю, ссылаясь на `entities.todos` по `toggledId`.


Есть два случая, когда мы хотим удалить todo из списка:
  * Поле `completed` является `true`, но `filter` - `active`.
  * `completed` является `false`, но `filter` - `completed`

Когда `shouldRemove` является `true`, мы хотим вернуть копию списка, которая не содержит  id todo, которая была только что переключена. Мы достигаем этого путем фильтрации списка по id и оставляем только те, которые имеют другой id. В противном случае мы возвращаем исходный массив.

##### Завершенная функция `handleToggle`
```javascript
const handleToggle = (state, action) => {
  const { result: toggledId, entities } = action.response;
  const { completed } = entities.todos[toggledId];
  const shouldRemove = (
    (completed && filter === 'active') ||
    (!completed && filter === 'completed')
  );
  return shouldRemove ?
    state.filter(id => id !== toggledId) :
    state;
};
```

[Демонстрация и подведение итогов на 3:20 видео](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)

<p align="center">
  <a href="./26-Normalizing_API_Responses_with_normalizr.md"><- Предыдущая</a>
</p>
