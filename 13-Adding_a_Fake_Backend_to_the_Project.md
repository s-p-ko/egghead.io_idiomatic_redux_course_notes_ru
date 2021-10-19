# 13. Добавление фэйкового бэкэнда в проект
[Ссылка на видео](https://egghead.io/lessons/javascript-redux-adding-a-fake-backend-to-the-project)

[Код урока на GitHub](https://github.com/gaearon/todos/tree/13-adding-a-fake-backend)

В следующих уроках мы больше не будем иметь дело с персистентностью. Вместо этого мы собираемся добавить в приложение асинхронное получение [данных].

Теперь мы удаляем весь код, относящийся к `localStorage` сохранению (персистентности), а также удаляем файл `localStorage.js`, потому что был добавлен новый модуль, реализующий удаленный фейковый API.

Все todos хранятся в памяти, и была добавлена искусственная задержка. У нас также есть методы, которые возвращают промисы, как у настоящей реализации API.

#### `src/api/index.js`
```javascript
import { v4 } from 'node-uuid';

// This is a fake in-memory implementation of something
// that would be implemented by calling a REST server.

const fakeDatabase = {
  todos: [{
    id: v4(),
    text: 'hey',
    completed: true,
  }, {
    id: v4(),
    text: 'ho',
    completed: true,
  }, {
    id: v4(),
    text: 'let’s go',
    completed: false,
  }],
};

const delay = (ms) =>
  new Promise(resolve => setTimeout(resolve, ms));

export const fetchTodos = (filter) =>
  delay(500).then(() => {
    .
    .
    .
```

Этот подход позволяет нам изучить, как Redux работает с асинхронной выборкой данных без написания реальной серверной части для приложения.

Теперь мы импортируем `fetchTodos` в разные модули нашего приложения.

`import { fetchTodos } from './api'`

Мы узнаем, как поместить эти задачи в хранилище Redux, позже, а пока мы можем вызвать `fetchTodos` с аргументом фильтра, что вернет промис, который разрешается через массив задач, точно так же, как бэкэнд REST возвращает массив.

#### `index.js`
```javascript
fetchTodos('all').then(todos =>
  console.log(todos)
);
```

Фэйковый API ожидает полсекунды, чтобы смоделировать сетевое соединение, а затем выполняет промис для массива todos, которые мы будем обрабатывать так, как если бы они были получены с удаленного сервера.


<p align="center">
<a href="./12-Wrapping_dispatch_to_Log_Actions.md"><- Предыдущая</a>
<a href="./14-Fetching_Data_on_Route_Change.md">Следующая -></a>
</p>