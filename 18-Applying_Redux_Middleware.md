# 18. Применение Redux мидлвара
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/18-applying-redux-middleware)

В предыдущем уроке мы выяснили контракт, который хотим использовать для наших мидлвар функций.

Тем не менее мидлвар не был бы очень полезным, если бы каждому приходилось реализовывать `wrapDispatchWithMiddlewares` самостоятельно.

Теперь мы удалим его и вместо этого импортируем утилиту `applyMiddleware` из Redux.

#### `configureStore` до:
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

Глядя на нашу функцию `configureStore`, обратите внимание, что нам не нужен `store` сразу. Мы можем переместить создание `store` вниз, где мы указываем мидлвар.

Мы также можем удалить нашу пользовательскую функцию `wrapDispatchWithMiddlewares`, и вместо этого мы сразу же создадим `createStore` с мидлваром.

Второй аргумент для создания стора будет результатом вызова `applyMiddleware` с моими мидлвар функциями в качестве позиционных аргументов.

Этот последний аргумент `createStore` называется усилителем и является необязательным. Если вы хотите указать `persistedState`, вам нужно сделать это перед усилителем (вы также можете пропустить `persistedState`, если у вас его нет).

#### `configureStore` посде:
```javascript
// We also deleted `wrapDispatchWithMiddlewares()`

const configureStore = () => {
  const middlewares = [promise];
  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(createLogger());
  }

  return createStore(
    todoApp,
    applyMiddleware(...middlewares)
  );
};
```

### Замена нашего кастомного мидлвара

Многие мидлвары доступны в виде пакетов npm. И написанные нами `promise` и `logger` мидлвары не являются исключением.

Запуск `npm install --save redux-promise` в терминале установит мидлвар, который реализует поддержку Promise.

Точно так же можно установить пакет `redux-logger`, который похож на мидлвар логгера, написанный нами ранее, но `redux-logger` более настраиваемый.

Теперь внутри `configureStore.js` мы можем импортировать и использовать наши новые мидлвары. Поскольку нам больше не нужно ссылаться на `store`, мы можем просто вернуть его прямо из `configureStore`.

#### Обновленный `configureStore.js`
```javascript
import { createStore, applyMiddleware } from 'redux';
import promise from 'redux-promise';
import createLogger from 'redux-logger';
import todoApp from './reducers';

const configureStore = () => {
  const middlewares = [promise];
  if (process.env.NODE_ENV !== 'production') {
    middlewares.push(createLogger());
    // Note: you can supply options to `createLogger()`
  }

  return createStore(
    todoApp,
    applyMiddleware(...middlewares)
  );
};

export default configureStore;
```

[Резюме со 2:06 видео](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)


<p align="center">
<a href="./17-The_Middleware_Chain.md"><- Предыдущая</a>
<a href="./19-Updating_the_State_with_the_Fetched_Data.md">Следующая -></a>
</p>