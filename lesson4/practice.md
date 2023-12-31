ТЗ
- Сделать приложение для запуска нескольких узлов объединённых репликацией
  - Сделать систему транзакций на основе json patch
    - Псевдокод
    - newsnap = patch(snap, transaction)
  - Построить репликацию
  - Поток отправки транзакций без оптимизации
    - Псевдокод
    ```
    for transaction in range wal {
        send transaction
    }
    ```
  - Поток применения
    - Псевдокод
    ```
    for {
        transaction <- stream
        if transaction already applied {
            continue
        }
        newsnap = patch(snap, transaction)
    }
    ```

- JSON patch 
  - Например
  - Было 
  ```json
  { "baz": "qux", "foo": "bar" }
  ```
  - Патч
  ```json
  [
    { "op": "replace", "path": "/baz", "value": "boo" },
    { "op": "add", "path": "/hello", "value": ["world"] },
    { "op": "remove", "path": "/foo" }
  ]
  ```
  - Стало
  ```json
  { "baz": "boo", "hello": ["world"] }
  ```
- websocket это канал постоянной двустороней связи используя http
    - Может работать между браузером и сервером
    - Может работать между двумя серверами
- Потребуются библиотечки!!!
  - go get github.com/evanphx/json-patch/v5
    - для транзакций
  - go get nhooyr.io/websocket 
    - для вебсокетов
- Сделать http сервер
  - /test отдает статический файл index.html (*)
    - Файл для отладки присходящего
    - Файл готовый (см ниже)
    - Файл прикрепить к приложеньке с помощью embed
  - /vclock отдаёт текущий vclock (7)
  - /replace (можно переименовать в /post) принимает data
    - создаёт транзакцию по структуре ниже (1)
    - отправляет в менеджер (2)
  - /get берет у менеджера состояние (3)
    - или напрямую из переменной
  - /ws отправляет репликационный поток (5)
- Менеджер транзакций умеет (2)
  - Применить транзакцию к snap (3)
    - patch := jsonpatch.DecodePatch ...
    - patch.Apply ...
  - Записать в журнал (4)
  - Должен учитывать vclock и если vclock[Source] больше чем Id самой транзакции,
    то транзакцию не применять

- У нас есть снапшот
  - Глобальная переменная
  - snap: string (3)
  - Значение переменной на запуске это пустой json объект
  - "{}"

- У нас есть журнал транзакций 
  - Глобальная переменная с массивом транзакций
  - wal: []string (4)

- Транзакция в журнале должна быть структурой (1)
  - struct 
    - Source: string     - ваша фамилия
    - Id: uint64         - возрастающий счетчик
    - Payload: string    - транзакция (json patch)
- Транзакция это строка внутри, которой JSON patch
- У нас есть логические часы (7)
  - Глобальная переменная
  - Это мапка map[string]uint64
    Например:
    ```
    {
        filonenko:100,
        madzhuga:0
    }
    ```
- У нас есть глобальная переменная, в которой мы хардкодим Source нашего узла
- У нас есть глобальная переменная — локальный счётчик наших транзакций, который мы инкрементим каждую локальную транзакцию
- У нас есть глобальная переменная, в которой перечислены все наши соседи
  - peers []string
- Репликация: 
- Состоит из двух отдельный блоков
  - 1й блок (или downstream канал)
  - websocket handler (5)
    - установлен на http роут /ws
    ```
    c, err := websocket.Accept(w, r, &websocket.AcceptOptions{
			InsecureSkipVerify: true,
			OriginPatterns:     []string{"*"},
		})
    ```
    - `wsjson.Write(r.Context(), c, transaction)`
    - отправляет репликационный поток
    - при новом входящем соединении регистрирует себя в менеджере транзакций
    - этот хендлер только отправляет (!) транзакции
  - 2й блок
  - websocket client (или upstream канал)
    - это несколько горутин по одной на каждый peer из переменной peers 
    ```
    c, _, err := websocket.Dial(ctx, fmt.Sprintf("ws://%s/ws", peer), nil)
    wsjson.Read(r.Context(), c, &transaction)
    ```
    - принимает репликационный поток
    - отгружает принятое в менеджер транзакций
    - этот хендлер только принимает (!) транзакции
  - websocket handler и websocket client — это разные блоки кода
- Внимание! Для менеджера транзакции «из репликации» и «от пользователя» должны быть одинаковыми (неразличимы).
- Чтобы проверить работу приложения запустить в двух экземплярах
  - на 8080
  - на 8081
- И указать соседей в peers
- Например:
```
1ая реплика — go run . 127.0.0.1:8080 127.0.0.1:8081
2ая реплика — go run . 127.0.0.1:8081 127.0.0.1:8080
```
- После перейти на урл /test и ввести транзакции
```
[{"op":"add", "path": "/<фамилия>", "value": []}]
```
- и потом
```
[{"op":"add", "path": "/<фамилия>/-", "value": "Hello"}]
```
- и потом
```json
[{"op":"add", "path": "/<фамилия>/-", "value": "World"}]
```
- В результате /get должен выглядеть
```json
{"<фамилия>":["Hello","World"]}
```
- Например
```json
[{"op":"add", "path": "/filonenko", "value": []}]
```
- 
```json
[{"op":"add", "path": "/filonenko/-", "value": "Hello"}]
```

