# 06. Навигация с React Router `<Link>`

[Ссылка на видео](https://egghead.io/lessons/javascript-redux-navigating-with-react-router-link?series=building-react-applications-with-idiomatic-redux)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/06-navigating-with-react-router-link)

В этом уроке мы обновим «ссылки», управляющие фильтром видимости, чтобы они больше походили на _реальные_ ссылки. Мы собираемся изменить его так, чтобы кнопка «Назад» в нашем браузере работала, а URL-адрес изменялся при переключении фильтров.

Мы начнем с добавления в Route параметра `path`, именуемый `filter`. Мы заключим его в скобки, чтобы сообщить React Router, что он необязательный (мы хотим, чтобы все Todos отображались по умолчанию).

#### Обновить `Root.js`

```javascript
const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/(:filter)" component={App} />
    </Router>
  </Provider>
);
.
.
.
```

_Примечание: пояснения в видео требуют версии `react-router` до 4.0.0. Начиная с этой версии были внесены некоторые изменения, требующие немного другого синтаксиса:_

#### Обновить `Root.js` (react-router v4.0.0 или выше)

```javascript
const Root = ({ store }) => (
  <Provider store={store}>
    <Router history={browserHistory}>
      <Route path="/:filter?" component={App} />
    </Router>
  </Provider>
);
.
.
.
```

### Обновить ссылки в `Footer.js`

Нам также необходимо обновить ссылки фильтров видимости внутри футера.

#### `Footer.js` до

```javascript
...
const Footer = () => (
  <p>
    Show:
    {" "}
    <FilterLink filter="SHOW_ALL">
      All
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_ACTIVE">
      Active
    </FilterLink>
    {", "}
    <FilterLink filter="SHOW_COMPLETED">
      Completed
    </FilterLink>
  </p>
);

export default Footer;
```

Предыдущая реализация использовала соглашение для свойства (prop) `filter`, но мы обновим их, чтобы они соответствовали «активному» ("active") и «завершенному» ("completed") путям, которые мы хотим отображать в адресной строке.

#### `Footer.js` после

```javascript
// Rest as above...
<p>
  Show:
  {" "}
  <FilterLink filter="all">
    All
  </FilterLink>
  {", "}
  <FilterLink filter="active">
    Active
  </FilterLink>
  {", "}
  <FilterLink filter="completed">
    Completed
  </FilterLink>
</p>
```

Мы используем пустую строку для обозначения пути по умолчанию и избегаем передачи пустой строки.

### Обновить реализацию `FilterLink.js`

В нашей текущей реализации компонент `FilterLink` отправляет действие каждый раз, когда по нему щелкают, затем считывает свое активное состояние из хранилища (store) и сравнивает свое свойство (prop) `filter` с `visibilityFilter` в хранилище.

#### `FilterLink.js` до

```javascript
...
const mapStateToProps = (state, ownProps) => ({
  active: ownProps.filter === state.visibilityFilter,
});

const mapDispatchToProps = (dispatch, ownProps) => ({
  onClick() {
    dispatch(setVisibilityFilter(ownProps.filter));
  },
});

const FilterLink = connect(
  mapStateToProps,
  mapDispatchToProps
)(Link);

export default FilterLink;
```

Однако нам больше не нужна эта реализация, поскольку мы хотим, чтобы маршрутизатор (router) контролировал любое состояние, указанное в URL. Мы импортируем `Link` из `react-router` и будем использовать его в нашей новой реализации.

`FilterLink` теперь принимает `filter` как проп и рендерит его через `Link` React Router'а.

Проп `to` соответствует пути, на который мы хотим, чтобы указывала ссылка, поэтому, если `filter` строго равен `'all'`, мы будем использовать корневой путь, иначе будем использовать сам `filter` в качестве пути URL-фдреса.

Мы также будем использовать проп `activeStyle`, чтобы стилизовать ее по-другому, когда ее проп `to` соответствует текущему пути.

Мы передаем `children` в сам `Link` и добавляем `children` в качестве пропа в `FilterLink`, чтобы родительский компонент мог указывать дочерние элементы.

#### `FilterLink.js` после

```javascript
import React, { PropTypes } from 'react';
import { Link } from 'react-router';

const FilterLink = ({ filter, children }) => (
  <Link
    to={filter === 'all' ? '' : filter}
    activeStyle={{
      textDecoration: 'none',
      color: 'black',
    }}
  >
    {children}
  </Link>
);

FilterLink.propTypes = {
  filter: PropTypes.oneOf(['all', 'completed', 'active']).isRequired,
  children: PropTypes.node.isRequired,
};

export default FilterLink;
```

_Примечание: пояснения в видео требуют версии `react-router` до 4.0.0. Начиная с этой версии были внесены некоторые изменения, требующие немного другого синтаксиса:_

#### `FilterLink.js` после (react-router v4.0.0 или выше)

```javascript
import React, { PropTypes } from 'react';
import { NavLink } from 'react-router-dom';

const FilterLink = ({ filter, children }) => (
    <NavLink
        exact
        to={'/' + (filter === 'all' ? '' : filter)}
        activeStyle={{
            textDecoration: 'none',
            color: 'black',
        }}
    >
        {children}
    </NavLink>
);

FilterLink.propTypes = {
  filter: PropTypes.oneOf(['all', 'completed', 'active']).isRequired,
  children: PropTypes.node.isRequired,
};

export default FilterLink;
```

### Ёще больше чистоты ...

Мы больше не используем экшн криэйтер `setVisibilityFilter`, поэтому его можно удалить из `src/actions/index.js`, оставив нам только экшн криэйтеры `addTodo` и `toggleTodo`.

Мы также можем удалить наш собственный компонент `Link` из `src/components`, так как теперь используем компонент `react-router`'а.

[Резюме со 2:20 видео](https://egghead.io/lessons/javascript-redux-navigating-with-react-router-link?series=building-react-applications-with-idiomatic-redux)

<p align="center">
<a href="./05-Adding_React_Router_to_the_Project.md"><- Предыдущая</a>
<a href="./07-Filtering_Redux_State_with_React_Router_Params.md">Следующая -></a>
</p>
