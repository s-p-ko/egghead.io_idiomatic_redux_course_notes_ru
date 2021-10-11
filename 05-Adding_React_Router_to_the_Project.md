# 05. Добавление React Router в проект

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/05-adding-react-router)

В этом уроке мы добавим React Router.

#### `Root.js` до

```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <App />
  </Provider>
);
.
.
.
```

Чтобы добавить в проект React Router, запустите:

`$ npm install --save react-router`

Внутри `Root.js` мы импортируем компоненты `Router` и `Route`.

Мы также заменяем `<App />` на `<Router />`. И важно, чтобы он все еще находился внутри `<Provider />`, чтоб любые, отображаемые (rendered) маршрутизатором компоненты, по-прежнему имели доступ к хранилищу (store).

Внутри `<Router />` мы поместим единственный элемент `<Route />`, который сообщает React Router'у, что мы хотим отобразить наш компонент `<App />` по корневому пути (`'/'`) в адресной строке браузера.

#### `Root.js` после

```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import { Router, Route, browserHistory } from 'react-router';
import App from './App';

const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/" component={App} />
    </Router>
  </Provider>
);
.
.
.
```

_Примечание: видео содержит исправление для странных символов адресной строки, происходящих из старого релиза `react-router`_

_Примечание: объяснения в видео требуют версии `react-router` до 4.0.0. Начиная с этой версии были внесены некоторые изменения, требующие немного другого синтаксиса:_

#### `Root.js` после (react-router v4.0.0 или выше)

```javascript
import React, { PropTypes } from 'react';
import { Provider } from 'react-redux';
import App from './App';
import { BrowserRouter, Route } from 'react-router-dom'

const Root = ({ store }) => (
    <Provider store={store}>
        <BrowserRouter>
            <Route path="/" component={App} />
        </BrowserRouter>
    </Provider>
);
.
.
.
```

[Резюме с 1:09 видео](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux#/tab-transcript)

<p align="center">
<a href="./04-Refactoring_the_Entry_Point.md"><- Предыдущая</a>
<a href="./06-Navigating_with_React_Router_Link.md">Следующая -></a>
</p>
