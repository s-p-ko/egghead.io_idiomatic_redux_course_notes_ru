## Перевод данного [конспекта](https://github.com/tayiorbeii/egghead.io_idiomatic_redux_course_notes) на русский язык с некоторыми дополнениями

# Building React Applications with Idiomatic Redux (Egghead.io)

![](https://s3.amazonaws.com/f.cl.ly/items/212E0u153X2A18131808/Image%202016-07-10%20at%2012.00.28%20PM.png?v=feaddbc8)

Это репо содержит конспект [Dan Abramov's](https://github.com/gaearon) second [Redux course](https://egghead.io/courses/building-react-applications-with-idiomatic-redux) on Egghead.io.

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

## 11. Нормализация состояния формы (Normalizing the State shape)

Узнаем, как нормализовать состояния форму для обеспечения согласованности данных, что важно в реальных приложениях.
[Видео](https://egghead.io/lessons/javascript-redux-normalizing-the-state-shape)

## 12. Обёртываем `dispatch()` в логи экшенов

Узнаем, как централизованные обновления в Redux позволяют нам логировать каждое изменение состояния в консоли вместе с экшеном, его вызвавшим.
[Видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-log-actions)

## 13. Добавление фэйкового бэкэнда в проект

Узнаем о фейковом бэкэнд-модуле, который будем использовать на следующих уроках для имитации получения данных.
[Видео](https://egghead.io/lessons/javascript-redux-adding-a-fake-backend-to-the-project)

## 14. Получение данных при изменении роута (маршрута)

Узнаем, как запускать асинхронный запрос при изменении роута (маршрута).
[Видео](https://egghead.io/lessons/javascript-redux-fetching-data-on-route-change)

## 15. Диспатчинг экшенов с получеными данными

Узнаем, как диспатчить Redux экшены после получения данных, и вспомним, как это сделать при изменении роута (маршрута).
[Видео](https://egghead.io/lessons/javascript-redux-dispatching-actions-with-the-fetched-data?series=building-react-applications-with-idiomatic-redux)

## 16. Обертывание `dispatch()` для распонавания проимсов

Узнаем, как научить `dispatch()` распознавать промисы, чтобы можно было переместить асинхронную логику из компонентов в асинхронные экшен криэйторы.
[Видео](https://egghead.io/lessons/javascript-redux-wrapping-dispatch-to-recognize-promises)

## 17. Цепочка мидлвар

Узнаем, как можно обобщить обертку `dispatch()` для различных целей в концепцию, под именем "мидлвар", которая широко доступна в экосистеме Redux.
[Видео](https://egghead.io/lessons/javascript-redux-the-middleware-chain)

## 18. Применение Redux мидлвара

Узнаем, как заменить написанные нами мидлвары и пользовательский код, который мы использовали, чтобы склеить его с существующими основными и сторонними утилитами.
[Видео](https://egghead.io/lessons/javascript-redux-applying-redux-middleware)

## 19. Обновление состояния с помощью полученных данных

Узнаем, как перемещение на сервер источника истинности для данных  изменяет форму состояния и редюсеры в нашем приложении.
[Видео](https://egghead.io/lessons/javascript-redux-updating-the-state-with-the-fetched-data)

## 20. Рефакторинг редюсеров

Узнаем, как удалить дублирование в файлах редюсера и как сохранить информацию о форме состояния вместе с недавно извлеченными редюсерами.
[Видео](https://egghead.io/lessons/javascript-redux-refactoring-the-reducers)

## 21. Отображение индикаторов загрузки

Узнаем, как отображать индикаторы загрузки во время загрузки данных.
[Видео](https://egghead.io/lessons/javascript-redux-displaying-loading-indicators)

## 22. Асинхронный диспатчинг экшенов с помощью Thunks

Узнаем о “thunks” - наиболее распространенном способе написания асинхронных экшен криэйтеров в Redux.
[Видео](https://egghead.io/lessons/javascript-redux-dispatching-actions-asynchronously-with-thunks)

## 23. Избегайте состояний гонки с помощью Thunks

Узнаем, как мидлвар Redux Thunk позволяет нам условно диспатчить  экшены, чтобы избежать ненужных сетевых запросов и потенциальных состояний гонки.
[Видео](https://egghead.io/lessons/javascript-redux-avoiding-race-conditions-with-thunks)

## 24. Отображение Error Messages

Узнаем, как обрабатывать ошибку внутри асинхронного экшена, отображать ее сообщение в пользовательском интерфейсе и предлагать пользователю возможность повторить действие.
[Видео](https://egghead.io/lessons/javascript-redux-displaying-error-messages)

## 25. Создание данных на сервере

Узнаем, как дождаться создания элемента на сервере и обновить соответствующее локальное состояние.
[Видео](https://egghead.io/lessons/javascript-redux-creating-data-on-the-server)

## 26. Нормализация API ответов с помощью `normalizr`

Узнаем, как использовать normalizr для преобразования всех API ответов в нормализованный формат, чтобы мы могли упростить редюсеры.
[Видео](https://egghead.io/lessons/javascript-redux-normalizing-api-responses-with-normalizr)

## 27. Обновление данных на сервере

Узнаем, как дождаться обновления элемента на сервере, а затем обновить соответствующее локальное состояние.
[Видео](https://egghead.io/lessons/javascript-redux-updating-data-on-the-server)
