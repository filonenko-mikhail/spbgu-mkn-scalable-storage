ТЗ фронтенд

- Задача сделать самый самый минимум для фронтенда, а именно просто одну кнопку
- Установить node.js
- Установить npm
- Структура файлов
```
.
└── lesson7
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
    "@types/lodash": "",
    "@types/react": "",
    "@types/react-dom": "",
    "@types/sprintf-js": "",
    "axios": "",
    "effector": "",
    "effector-react": "",
    "fast-json-patch": "",
    "lodash": "",
    "react": "",
    "react-dom": "",
    "react-icons": "",
    "sprintf-js": "",
    "ts-loader": "",
    "websocket-ts": ""
  },
  "scripts": {
    "watch": "webpack watch",
    "build": "webpack"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
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
- `src/index.tsx`
```tsx
import * as React from "react"
import * as ReactDOM from "react-dom"
import { useState } from 'react'


function Button() {
  const [title, setTitle] = useState("<placeholder>")
  const handleClick = function (event: React.MouseEvent<HTMLButtonElement, MouseEvent>) {
    console.log(event)
    setTitle(prompt("enter new title", "no data"))
  }
  return (
  <>
  <h1>{title}</h1>
  <button onClick={handleClick} title="Click me">Click Me</button>
  </>)
}

ReactDOM.render(
  <React.StrictMode>
    <Button></Button>
  </React.StrictMode>,
  document.getElementById("root")
)
```
- Запустить установку зависимостей
  - npm install
- Запустить сборку проекта
  - npm run build
- Открыть index.html в броузере
- Нажать на кнопку посмотреть всё ли сработало
