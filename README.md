# Building React Applications with Idiomatic Redux (Egghead.io)

## Перевод [конспекта](https://github.com/tayiorbeii/egghead.io_idiomatic_redux_course_notes) на русский  с некоторыми дополнениями

![](https://s3.amazonaws.com/f.cl.ly/items/212E0u153X2A18131808/Image%202016-07-10%20at%2012.00.28%20PM.png?v=feaddbc8)

Это репо - конспект [Dan Abramov's](https://github.com/gaearon) second [Redux course](https://egghead.io/courses/building-react-applications-with-idiomatic-redux) on Egghead.io.

### Gitbook Setup

```

npm install -g gitbook-cli
npm install
gitbook install
gitbook serve
```

## 01\. Упрощение стрелочных функций

Смотрим, как можно использовать возможности ES6 для еще большей очистки стрелочных функций. [Видео](https://egghead.io/lessons/javascript-redux-simplifying-the-arrow-functions)

## 02. Обеспечение начального состояния

Узнаем, как запустить приложение Redux с ранее сохраненным состоянием и как оно сливается с начальным состоянием, обусловленным редюсерами. [Видео](https://egghead.io/lessons/javascript-redux-supplying-the-initial-state)

## 03. Сохранение состояния в Local Storage

Узнаем, как использовать store.subscribe() для эффективного сохранения некоторых состояний приложения в localStorage и восстановления их после обновления. [Видео](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage#/tab-transcript)

## 04. Рефакторинг точки входа

Узнаем, как лучше разделить код в точке входа, чтобы подготовить его к добавлению маршрутизатора.
[Видео](https://egghead.io/lessons/javascript-redux-refactoring-the-entry-point?series=building-react-applications-with-idiomatic-redux#/tab-transcript)

## 05. Добавление React Router в проект

Узнаем, как добавить React Router в проект Redux и заставить его рендерить наш корневой (root) компонент. [Видео](https://egghead.io/lessons/javascript-redux-adding-react-router-to-the-project?series=building-react-applications-with-idiomatic-redux#/tab-transcript)

## 06. Навигация с React Router `<Link>`

Узнаем, как изменить адресную строку с помощью компонента из React Router.
[Видео](https://egghead.io/lessons/javascript-redux-navigating-with-react-router-link?series=building-react-applications-with-idiomatic-redux)

## 07. Фильтрация Redux State c React Router Params

Узнаем, как добавление React Router меняет баланс обязанностей и как компоненты могут использовать и то, и другое одновременно.
[Видео](https://egghead.io/lessons/javascript-redux-filtering-redux-state-with-react-router-params)

## 08. Использование `withRouter()` для вставки параметров в подключенные компоненты

Мы узнаем, как использовать `withRouter()` для внедрения параметров, предоставленных React Router, в подключенные компоненты глубоко в дереве, не передавая их полностью вниз как пропсы.
[Видео](https://egghead.io/lessons/javascript-redux-using-withrouter-to-inject-the-params-into-connected-components)

## 09. Использование сокращенной записи `mapDispatchToProps()`

Мы узнаем, как избежать шаблонного кода в `mapDispatchToProps()` для общего случая, когда аргументы экшн криэйтера совпадают с коллбэк проп аргументами.
[Видео](https://egghead.io/lessons/javascript-redux-using-mapdispatchtoprops-shorthand-notation)

## 10. Размещение (colocating) селекторов с редюсерами

Узнаем, как инкапсулировать знания о форме состояния в файлах редюсера, чтобы компоненты не полагались на них.
[Видео](https://egghead.io/lessons/javascript-redux-colocating-selectors-with-reducers?series=building-react-applications-with-idiomatic-redux#/tab-transcript)

## 11. Normalizing the State Shape

We will learn how to normalize the state shape to ensure data consistency that is important in real-world applications.
[Видео](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)

## 12. Wrapping `dispatch()` to Log Actions

We will learn how centralized updates in Redux let us log every state change to the console along with the action that caused it.
[Видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)

## 13. Adding a Fake Backend to the Project

We will learn about fake backend module that we will be using throughout the next lessons to simulate data fetching.
[Видео](https://egghead.io/lessons/javascript-redux-adding-a-fake-backend-to-the-project)

## 14. Fetching Data on Route Change

We will learn how to fire up an async request when the route changes.
[Видео](https://egghead.io/lessons/javascript-redux-fetching-data-on-route-change)

## 15. Dispatching Actions with the Fetched Data

We will learn how to dispatch a Redux action after the data has been fetched and recap how to do it when the route changes.
[ВидеоВидео](https://egghead.io/lessons/javascript-redux-dispatching-actions-with-the-fetched-data?series=building-react-applications-with-idiomatic-redux)

## 16. Wrapping `dispatch()` to Recognize Promises

We will learn how to teach `dispatch()` to recognize Promises so that we can move the async logic out of the components into asynchronous action creators.
[Видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-recognize-promises)

## 17. The Middleware chain

We will learn how we can generalize wrapping `dispatch()` for different purposes into a concept called “middleware” that is widely available in the Redux ecosystem.
[Видео](https://egghead.io/lessons/javascript-redux-the-middleware-chain)

## 18. Applying Redux Middleware

We will learn how to replace the middleware we wrote and the custom code we used to glue it together with the existing core and third party utilities.
[Видео](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)

## 19. Updating the State with the Fetched Data

We will learn how moving the source of truth for the data to the server changes the state shape and the reducers in our app.
[Видео](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data)

## 20. Refactoring the reducers

We will learn how to remove the duplication in our reducer files and how to keep the knowledge about the state shape colocated with the newly extracted reducers.
[Видео](https://egghead.io/lessons/javascript-redux-refactoring-the-reducers)

## 21. Displaying Loading Indicators

We will learn how to display the loading indicators while the data is being fetched.
[Видео](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)

## 22. Dispatching Actions Asynchronously with Thunks

We will learn about “thunks”—the most common way to write async action creators in Redux.
[Видео](https://egghead.io/lessons/javascript-redux-dispatching-actions-asynchronously-with-thunks)

## 23. Avoiding Race Conditions with Thunks

We will learn how Redux Thunk middleware lets us conditionally dispatch actions to avoid unnecessary network requests and potential race conditions.
[Видео](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)

## 24. Displaying Error Messages

We will learn how to handle an error inside an async action, display its message in the user interface, and offer user an opportunity to retry the action.
[Видео](https://egghead.io/lessons/javascript-redux-displaying-error-messages)

## 25. Creating Data on the Server

We will learn how to wait until the item is created on the server, and update the corresponding local state.
[Видео](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)

## 26. Normalizing API Responses with `normalizr`

We will learn how to use normalizr to convert all API responses to a normalized format so that we can simplify the reducers.
[Видео](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)

## 27. Updating Data on the Server

We will learn how to wait until the item is updated on the server, and then update the corresponding local state.
[Видео](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)
