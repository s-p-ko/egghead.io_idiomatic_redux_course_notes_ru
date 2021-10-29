# 01. Упрощение стрелочных функций

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions?course=building-react-applications-with-idiomatic-redux)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/01-simplifying-the-arrow-functions)

Поскольку экшен криэйторы (action creators) - это обычные функции JavaScript, определять их можно как угодно.

Например, если вам не нравится стрелочная нотация, можно заменить её традиционным синтаксисом объявления функции:

##### Синтаксис стрелочной функции

``` javascript
export const addTodo = (text) => {
  return {
    type: 'ADD_TODO',
    id: (nextTodoId++).toString(),
    text,
  };
};
```

##### Традиционный синтаксис функций

``` javascript
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    id: (nextTodoId++).toString(),
    text,
  };
}
```

Если вы предпочитаете использовать стрелочные функции, их можно сделать еще более краткими.

Рассматривая нашу стрелочную функцию выше, мы видим, что она начинается и заканчивается фигурной скобкой и содержит внутри `return` оператор. Поскольку оператор return - это все, что находится внутри нашей функции, то можно использовать его как тело стрелочной функции.

Мы можем удалить блок в пользу объектного выражения:

```javascript
export const addTodo = (text) => ({
  type:'ADD_TODO',
  id: (nextTodoId++).toString(),
  text,
})
```

_Примечание:_ Важно заключить выражение в круглые скобки, чтобы синтаксический анализатор понимал это как выражение, а не как блок.

Эти действия можно повторить для любой функции, которая просто возвращает объект; для этого удалите оператор `return` и превратите тело в выражение.

Этот процесс можно использовать и вне экшен криэйтеров. Например, в `mapStateToProps` и `mapDispatchToProps`, чтобы просто вернуть объекты:

##### до

```javascript
const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
  }
}
```

##### после

```javascript
const mapStateToProps = (state, ownProps) => ({
    active: ownProps.filter === state.visibilityFilter
})

const mapDispatchToProps = (dispatch, ownProps) => ({
    onClick: () => {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
})
```

Мы можем сделать `mapDispatchToProps` более компактным, заменив стрелочную функцию краткой нотацией метода, которая является частью ES6 и доступна, когда функция определена внутри объекта.

```javascript
const mapDispatchToProps = (dispatch, ownProps) => ({
    onClick() {
      dispatch(setVisibilityFilter(ownProps.filter))
    }
})
```

#### [Резюме со 2:05 видео](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions?course=building-react-applications-with-idiomatic-redux)

<p align="center">
<a href="./02-Supplying_the_Initial_State.md">Следующая -></a>
</p>
