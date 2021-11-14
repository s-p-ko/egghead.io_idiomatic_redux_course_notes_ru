# 17. Цепочка мидлвар
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-the-middleware-chain)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/17-the-middleware-chain)

В нашем последнем уроке мы написали две функции, которые, чтобы добавить кастомное поведение, оборачивают `dispatch` функцию. Давайте подробнее рассмотрим, как они работают вместе.

```javascript
const configureStore = () => {
  const store = createStore(todoApp);

  if (process.env.NODE_ENV !== 'production') {
    store.dispatch = addLoggingToDispatch(store);
  }

  store.dispatch = addPromiseSupportToDispatch(store);

  return store;
};
```
Последняя версия функции `dispatch` перед возвратом `store` является результатом вызова `addPromiseSupportToDispatch`.

#### `addPromiseSupportToDispatch` до:
```javascript
const addPromiseSupportToDispatch = (store) => {
  const rawDispatch = store.dispatch;
  return (action) => {
    if (typeof action.then === 'function') {
      return action.then(rawDispatch);
    }
    return rawDispatch(action);
  };
};
```

Функция, возвращающая `addPromiseSupportToDispatch`, действует как обычная `dispatch` функция, но если она получает промис, она ожидает его разрешения (resolve), а затем передает результат в `rawDispatch` (где`rawDispatch` - это предыдущее значение `store.dispatch`).

Для не-промисов функция сразу вызывает `rawDispatch`. `rawDispatch` соответствует store.dispatch во время вызова `addPromiseSupportToDispatch`.

### Рефакторинг наших `dispatch`-усиливающих функций

Поскольку `store.dispatch` был переназначен ранее (внутри `configureStore`), не совсем справедливо называть его `rawDispatch` внутри `addPromiseSupportToDispatch`.

Мы переименуем `rawDispatch` в `next`, потому что это следующая `dispatch` функция в цепочке.

#### `addPromiseSupportToDispatch` после:
```javascript
const addPromiseSupportToDispatch = (store) => {
  const next = store.dispatch;
  return (action) => {
    if (typeof action.then === 'function') {
      return action.then(next);
    }
    return next(action);
  };
};
```

Выше, `next` относится к `store.dispatch`, который был возвращен из `addLoggingToDispatch()`.

Напомним, что наша функция `addLoggingToDispatch()` также возвращает функцию с тем же API, что и исходная `dispatch` функция, но логирует экшен `type`, предыдущее `state`, `action` и следующее `state`  по пути.

Это вызывает `rawDispatch`, который соответствует `store.dispatch` во время вызова `addLoggingToDispatch`. В данном случае это `store.dispatch`, предоставляемый `createStore()` внутри `configureStore`.

Однако, вполне возможно, что мы захотим переопределить функцию `dispatch` перед добавлением логирования.

Для единообразия мы также переименуем `rawDispatch` в `next`. В данном конкретном случае `next` указывает на исходный файл `store.dispatch`.

#### `addLoggingToDispatch()`
```javascript
const addLoggingToDispatch = (store) => {
  const next = store.dispatch;
  if (!console.group) {
    return next;
  }

  return (action) => {
    console.group(action.type);
    console.log('%c prev state', 'color: gray', store.getState());
    console.log('%c action', 'color: blue', action);

    const returnValue = next(action);

    console.log('%c next state', 'color: green', store.getState());
    console.groupEnd(action.type);
    return returnValue;
  };
};
```

### Знакомство с мидлвар функциями

Хотя данный метод расширения стор работает - не очень хорошо переопределять общедоступный API и заменять его кастомными функциями.

Чтобы уйти от этого паттерна, объявим массив _middleware functions_, который является просто причудливым названием для функций с дополнительными функциями, которые мы написали.

Этот массив `middlewares` будет содержать функции, которые будут применяться позже как один (единый) шаг.

Мы поместим `addLoggingToDispatch` и `addPromiseSupportToDispatch` в массив мидлваров.

Теперь мы создаем функцию `wrapDispatchWithMiddlewares()`, которая принимает `store` в качестве первого аргумента, а массив мидлваров - в качестве второго.

#### Рефакторинг `configureStore`
```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];

  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(logger);
  }

  wrapDispatchWithMiddlewares(store, middlewares);

  return store;
};
```

Внутри `wrapDispatchWithMiddlewares()` мы собираемся использовать метод `middlewares`'а `forEach` для запуска некоторого кода для каждого мидлвара.

В частности, мы переопределим функцию `store.dispatch`, чтобы она указывала на результат вызова мидлвара со `store` в качестве аргумента.

```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.forEach(middleware =>
    store.dispatch = middleware(store)
  );
```

Напомним, что внутри самих мидлвар функций есть определенный шаблон, который мы должны повторить. Мы берем значение `store.dispatch` и сохраняем его в переменной с именем `next`, которую мы вызываем позже.

Чтобы сделать его частью контракта мидлвара, мы можем сделать `next` внешним аргументом, точно так же, как `store` перед ним и `action` после него.

#### Обновление `addLoggingToDispatch()`
```javascript
const addLoggingToDispatch = (store) => {
  return (next) => {
    if (!console.group) {
      return next;
    }

    return (action) => {
      console.group(action.type);
      console.log('%c prev state', 'color: gray', store.getState());
      console.log('%c action', 'color: blue', action);

      const returnValue = next(action);

      console.log('%c next state', 'color: green', store.getState());
      console.groupEnd(action.type);
      return returnValue;
    };
  }
};
```

С этим изменением мидлвар становится функцией, которая возвращает функцию, которая возвращает функцию.

Этот паттерн называется **каррирование**. Это не очень распространено в JavaScript, но широко распространено в языках функционального программирования.


#### Обновление `addPromiseSupportToDispatch`
```javascript
const addPromiseSupportToDispatch = (store) => {
  return (next) => {
    return (action) => {
      if (typeof action.then === 'function') {
        return action.then(next);
      }
      return next(action);
    };
  }
};
```

Опять же, вместо того, чтобы брать следующий мидлвар из стора, мы сделаем его вводимым в качестве аргумента, чтобы функция, вызывающая мидлвар, могла выбирать, какой мидлвар передать.

Наконец, поскольку `store` - не единственный внедренный (инжектед) аргумент, нам также необходимо внедрить следующий мидлвар, который является предыдущим значением `store.dispatch`.

#### Обновление `wrapDispatchWithMiddlewares`
```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

Теперь, когда мидлвары являются первоклассной концепцией, мы можем переименовать `addLoggingToDispatch` просто в `logger`, а  `addPromiseSupportToDispatch` - в `promise`.


### Оснастим стрелками и if'ами наш `promise` мидлвар

Стиль каррирования объявления функции может быть трудным для чтения. К счастью, мы можем использовать стрелочные функции и полагаться на тот факт, что их тела могут быть выражениями.

```javascript
// Before
const addPromiseSupportToDispatch = (store) => {
  return (next) => {
    return (action) => {
      if (typeof action.then === 'function') {
        return action.then(next);
      }
      return next(action);
    };
  }
};

// After
const promise = (store) => (next) => (action) => {
  if (typeof action.then === 'function') {
    return action.then(next);
  }
  return next(action);
}
```

Это по-прежнему функция, которая возвращает функцию, возвращающую функцию, но ее гораздо легче читать.

Ментальная модель, которую вы можете использовать для этого, - "_это просто функция с несколькими аргументами, которые применяются по мере их появления_".

### Завершаем с мидлварами

Наши мидлвары в настоящее время указаны в том порядке, в котором функция `dispatch` переопределяется, но было бы более естественным указать порядок, в котором экшен распространяется через мидлвары.

Мы изменим наше объявление мидлвара, чтобы указать их в порядке, в котором экшен проходит через них:

```javascript
const configureStore = () => {
  const store = createStore(todoApp);
  const middlewares = [promise];
  .
  .
  .
```
Мы также `wrapDispatchWithMiddlewares` справа налево, клонируя прошлый массив, а затем обращая его.


```javascript
const wrapDispatchWithMiddlewares = (store, middlewares) =>
  middlewares.slice().reverse().forEach(middleware => {
    store.dispatch = middleware(store)(store.dispatch);
  });
```

[Резюме с 5:42 видео](https://egghead.io/lessons/javascript-redux-the-middleware-chain)


<p align="center">
<a href="./16-Wrapping_dispatch_to_Recognize_Promises.md"><- Предыдущая</a>
<a href="./18-Applying_Redux_Middleware.md">Следующая -></a>
</p>
