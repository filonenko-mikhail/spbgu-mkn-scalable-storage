ТЗ фронтенд и бекенд

- Напомню что умеет бекенд:
- Поднимает http server с роутами
  - /replace
    - принимает json patch и создаёт транзакцию
  - /ws
    - отправляет транзакции по websocket подключению

- Задача сделать фронтенд, который сможет в бекенд
- Установить node.js
- Установить npm
- Структура файлов
```
.
└── lesson8
    ├── package.json
    ├── webpack.config.js
    ├── tsconfig.json
    ├── src
    │   └── index.tsx
    └── index.html
```
- Файлы проекта
  - `package.json`
    - используется утилитой npm
    - отвечает на два вопроса:
      - какие зависимости установить
      - как собрать проект
  - `webpack.config.js`
    - используется утилитой webpack
    - отвечает на вопрос, как собрать проект
  - `tsconfig.json`
    - используется траспилятором typescript
    - отвечает на вопрос преобразования typescript в js
- Файлы исходников
  - `index.html`
    - является входящий точкой сайта
    - расположен базовый блок, в котором будет жить react приложение
  - `src/index.tsx`
    - является входящей точкой именно react приложения 
    - рендерится в `index.html`

- `package.json`
```json
{
  "name": "typescript-react-project",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@types/react": "",
    "@types/react-dom": "",
    "effector": "",
    "effector-react": "",
    "fast-json-patch": "",
    "react": "",
    "react-dom": "",
    "ts-loader": "",
    "websocket-ts": ""
  },
  "scripts": {
    "watch": "webpack watch",
    "build": "webpack"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "devDependencies": {
    "typescript": "",
    "webpack": "",
    "webpack-cli": ""
  }
}
```
- `webpack.config.js`
```js
const path = require('path');
module.exports = {
  entry: './src/index.tsx',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      }
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js', '.jsx']
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
  devtool: "source-map",
  optimization: {
    minimize: false
  },
};
```
- `tsconfig.json`
```json
{
  "compilerOptions": {
    "outDir": "./dist/",
    "noImplicitAny": true,
    "module": "es6",
    "target": "es5",
    "jsx": "react",
    "allowJs": true,
    "moduleResolution": "node",
    "inlineSourceMap": true
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules"
  ]
}
```
- index.html
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="icon" href="data:,">
    <title>The Game</title>
  </head>
  <body>
    <!-- this is where react renders into -->
    <div id="root"></div>
    <script src="dist/bundle.js"></script>
  </body>
</html>
```

- Все сниппеты можно копировать и вставлять друг за другом в файл index.tsx
- Указываем все импорты которые нам пригодятся
  - Работа с управлением состоянием effector и проброс состояние в react компоненты
    - `import { createEvent, createStore } from "effector"`
    - `import { useEvent, useStore } from "effector-react"`
  - Работа с websocket
    - `import { ConstantBackoff, Websocket, WebsocketBuilder, WebsocketEvent } from "websocket-ts"`
  - Работа с json patch
    - `import { applyPatch } from "fast-json-patch"`
  
- `src/index.tsx`
```tsx
import * as React from "react"
import * as ReactDOM from "react-dom"
import { createEvent, createStore } from "effector"
import { useEvent, useStore } from "effector-react"

import { ConstantBackoff, Websocket, WebsocketBuilder, WebsocketEvent } from "websocket-ts"

import { applyPatch } from "fast-json-patch"

// Вот тут всякая логика

ReactDOM.render(
  <React.StrictMode>
  </React.StrictMode>,
  document.getElementById("root")
)
```
- Всякая логика
- Создаем состояние приложения (или снапшот с бекенда) и его патч
  - Для этого потребуется effector конструкции:
    - store (createStore)
    - event (createEvent)
  - store.on подключается к event и выполняет колбек с мутацией состояния
  - в коллбеке приходит json patch
  - мы применяем json patch к состоянию с помощью функции applyPatch
  - функцию возвращает новое состояние
```tsx
const patchSnapEv = createEvent()
const $snap = createStore<any>({})
$snap.on(patchSnapEv, (state, patch:any) => {
  const doc = applyPatch(state, patch, false, false).newDocument
  return doc
})
```
- Подключаем websocket поток приходящих json patch-ей
  - Берем приходящее из вебсокета сообщение
  - Отправляем его в event для наложения патча на состояние
```tsx
const ws = new WebsocketBuilder("ws://localhost:8080/ws")
  .withBackoff(new ConstantBackoff(10*1000))
  .build()

const receiveMessage = (i: Websocket, ev: MessageEvent) => {
  const transaction = JSON.parse(ev.data)
  const patch = JSON.parse(transaction.Payload)
  patchSnapEv(patch)
}

ws.addEventListener(WebsocketEvent.message, receiveMessage)
```
- Создаём состояние текущего игрока
  - event
  - store
  - колбек который заменяем имя игрока на событии event
```tsx
const setPlayerNameEv = createEvent<string>()
const $playerName = createStore<string>("")
$playerName.on(setPlayerNameEv, (_, payload) => {
  return payload
})
```
- Создаём react компонент с контролами для управления игроком
  - имя игрока
  - кнопка создать
  - кнопка вверх
  - кнопка вниз
  - кнопка влево
  - кнопка вправо
  - В контроле подключаем использование store $snap
  - В контроле подключаем использование store $playerName
```tsx
function PlayerInput() {
  const setPlayerName = useEvent(setPlayerNameEv)
  const playerName = useStore($playerName)
  const handleChange = function(event: React.ChangeEvent<HTMLInputElement>) {
    setPlayerName(event.target.value)
  }

  const snap = useStore($snap)

  const handleClick = function() {
    fetch("http://127.0.0.1:8080/replace", {
      method: "POST",
      body:`[{"op":"add", "path": "/${playerName}", "value": {"x":20, "y":20}}]`})
  }

  const updatePlayer = function(player:any) {
    fetch("http://127.0.0.1:8080/replace", {
      method: "POST",
      body:`[{"op":"add", "path": "/${playerName}", "value": {"x":${player.x}, "y":${player.y}}}]`})
  }
  const handleUp = function() {
    let player = snap[playerName]
    player.y--
    updatePlayer(player)
  }

  const handleDown = function() {
    let player = snap[playerName]
    player.y++
    updatePlayer(player)
  }

  const handleLeft = function() {
    let player = snap[playerName]
    player.x--
    updatePlayer(player)
  }

  const handleRight = function() {
    let player = snap[playerName]
    player.x++
    updatePlayer(player)
  }

  return (<>
    <input type="text"
      onChange={handleChange}
      value={playerName}/>
    <input type="button"
      value={"Установить"}
      onClick={handleClick}/>
    <input type="button"
      value={"Вверх"}
      onClick={handleUp}/>
    <input type="button"
      value={"Вниз"}
      onClick={handleDown}/>
    <input type="button"
      value={"Влево"}
      onClick={handleLeft}/>
    <input type="button"
      value={"Вправо"}
      onClick={handleRight}/>
  </>)
}
```
- Создаём react компонент для рендеринга поля и игрока
  - В контроле подключаем использование store $snap
```tsx
function Players() {
  const snap = useStore($snap)
  let players = []
  for (const k in snap) {
    let player = snap[k]
    players.push(<circle cx={player.x} cy={player.y} r="20" stroke="white" fill="transparent" />)
  }
  return <>{players}</>
}
```

- И рендерим в итоговом рут компоненте react
```tsx
ReactDOM.render(
  <React.StrictMode>
    <PlayerInput></PlayerInput>
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 600">
      <rect x='0' y='0' width='100%' height='100%' fill='tomato' opacity='0.75' />
      <Players></Players>
    </svg>
  </React.StrictMode>,
  document.getElementById("root")
)
```

- Запустить установку зависимостей
  - npm install
- Запустить сборку проекта
  - npm run build
- Открыть index.html в броузере
- Запустить бекенд `go run .`
- Нажать на кнопку посмотреть всё ли сработало
