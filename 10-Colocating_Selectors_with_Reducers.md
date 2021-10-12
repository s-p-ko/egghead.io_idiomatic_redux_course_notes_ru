# 10. Размещение (colocating) селекторов с редюсерами
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/10-colocating-selectors-with-reducers)

Внутри `VisibleTodoList` функция `mapStateToProps` использует функцию `getVisibleTodos` и передает фрагмент `state`, соответствующий `todos`. Однако, если мы когда-нибудь изменим структуру состояния, мы должны не забыть обновить всю эту сторону.

Чтобы очистить это, мы можем переместить функцию `getVisibleTodos` из слоя представления и поместить ее внутрь файла, содержащего наш `todos` редюсер. Мы делаем это, потому что `todos` редюсер больше всего знает о внутренней структуре `todos` состояния (state).

#### `VisibleTodoList` до
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'all':
      return todos;
    case 'completed':
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};

const mapStateToProps = (state, { params }) => ({
  todos: getVisibleTodos(state.todos, params.filter || 'all'),
});
```

_Примечание: объяснения в видео требуют версии `react-router` до 4.0.0. Начиная с этой версии были внесены некоторые изменения, требующие немного другого синтаксиса:_

#### `VisibleTodoList` до (react-router v4.0.0 или выше)
```javascript
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'all':
      return todos;
    case 'completed':
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};

const mapStateToProps = (state, { match }) => ({
  todos: getVisibleTodos(state.todos, match.params.filter || 'all'),
});
```

### Обновление нашего редюсера

Мы собираемся переместить реализацию нашего `getVisibleTodos` в файл с редюсерами и сделать его именованным экспортом.

Мы придерживаемся простого соглашения. Экспорт по умолчанию всегда является функцией редюсера, но любой именованный экспорт, начинающийся с `'get'`, - это функция, которая подготавливает данные для отображения пользовательским интерфейсом (UI). Обычно мы называем эти функции _selectors_, потому что они выбирают что-то из текущего состояния.

В редюсерах аргумент `state` соответствует состоянию этого конкретного редюсера, поэтому мы будем следовать тому же соглашению для селекторов. Аргумент `state` соответствует состоянию экспортируемого редюсера в этом файле.

#### Внутри `src/reducers/todos.js`
```javascript
export const getVisibleTodos = (state, filter) => {
  switch (filter) {
    case 'all':
      return state;
    case 'completed':
      return state.filter(t => t.completed);
    case 'active':
      return state.filter(t => !t.completed);
    default:
      throw new Error(`Unknown filter: ${filter}.`);
  }
};
```

### Обновление корневого редюсера
Внутри `VisibleTodoList` мы все еще зависим от структуры состояния (state structure), потому что мы читаем `todos` из состояния, но фактический метод чтения `todos` может измениться в будущем.

Имея это в виду, мы собираемся обновить наш корневой редюсер с помощью именованного экспорта селектора. Он также будет называться `getVisibleTodos` и, как и раньше, также принимает `state` и `filter`. Однако в этом случае `state` соответствует состоянию комбинированного редюсера.

Теперь мы хотим иметь возможность вызывать функцию `getVisibleTodos`, определенную в файле` todos` вместе с редюсером, но мы не можем использовать именованный импорт, потому что в области видимости есть функция с точно таким же именем.

Чтобы обойти это, мы будем использовать синтаксис импорта пространства имен, который помещает все экспорты в объект (в данном случае называемый `fromTodos`)..

Теперь мы можем использовать `fromTodos.getVisibleTodos()` для вызова функции, которую мы определили в другом файле, и передать фрагмент `state`, соответствующий `todos`.

#### Обновление корневого редюсера (`src/reducers/index.js`)
```javascript
import { combineReducers } from 'redux';
import todos, * as fromTodos from './todos';

const todoApp = combineReducers({
  todos,
});

export default todoApp;

export const getVisibleTodos = (state, filter) =>
  fromTodos.getVisibleTodos(state.todos, filter);
```

### Обновление `VisibleTodoList`
Теперь мы можем вернуться к нашему компоненту `VisibleTodoList` и импортировать `getVisibleTodos` из корневого файла редюсера.

`import { getVisibleTodos } from '../reducers'`

`getVisibleTodos` инкапсулирует все знания о форме состояния приложения, поэтому мы можем просто передать ему все состояние нашего приложения, и оно выяснит, как выбрать видимые задачи (visible todos) в соответствии с логикой, описанной в нашем селекторе.

Наконец, мы можем вернуться к нашей маппинг функции `mapStateToProps` и вызвать `getVisibleTodos()` со всем состоянием (entire state), а не со `state.todos`, как раньше (эта логика была перенесена в редюсер):

```
const mapStateToProps = (state, { params }) => ({
      todos: getVisibleTodos(state, params.filter || 'all' )
  });
```

_Примечание: объяснения в видео требуют версии `react-router` до 4.0.0. Начиная с этой версии были внесены некоторые изменения, требующие немного другого синтаксиса:_

### react-router v4.0.0 или выше
```
const mapStateToProps = (state, { match }) => ({
      todos: getVisibleTodos(state, match.params.filter || 'all' )
  });
```

[Резюме со 2:51 in video](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux)


<p align="center">
<a href="./09-Using_mapDispatchToProps_Shorthand_Notation.md"><- Предыдущая</a>
<a href="./11-Normalizing_the_State_Shape.md">Следующая -></a>
</p>