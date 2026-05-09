# Сценарий видео: JSPipe для Unity WebGL без боли

> Формат: живой туториал на 10-14 минут.
>
> Настроение: не лекция и не чтение документации. Разработчик показывает реальную боль Unity WebGL interop, быстро собирает рабочий пример и по пути объясняет, почему JSPipe устроен именно так.

## Название

**Unity WebGL + TypeScript + npm: хватит страдать с `.jslib`**

Другие варианты:

- **C# вызывает TypeScript в Unity WebGL. Без ручного ада**
- **Как я подружил Unity WebGL с npm**
- **JSPipe за 10 минут: мост между Unity и браузером**

## Главная мысль видео

Не надо продавать ассет в лоб.

Видео должно звучать так:

> "Вот типичная боль Unity WebGL. Вот почему обычный `.jslib` быстро становится тесным. А теперь я покажу, как сделать то же самое через нормальный модульный bridge: C# вызывает TypeScript, TypeScript отвечает, npm-пакет подключается, WebGL build все собирает."

## 0:00-0:25 - Хук

### На экране

Быстро показать `.jslib` или псевдокод с глобальной функцией, потом рядом TypeScript/npm import.

### Реплика

Если вы когда-нибудь пытались подключить нормальный npm-пакет к Unity WebGL, вы знаете этот момент.

Сначала у вас один маленький `.jslib`.

Потом второй.

Потом callback id.

Потом JSON строкой.

Потом `Promise`, который где-то умер, а Unity делает вид, что ничего не произошло.

В этом видео я покажу другой подход: C# и TypeScript будут общаться через модули, без свалки глобальных функций.

## 0:25-0:50 - Что получится в конце

### На экране

Показать финальный лог или заготовленный результат:

```text
Name from TypeScript: Player from TypeScript
Hello from TypeScript: Browser runtime
```

### Реплика

За несколько минут мы сделаем две вещи.

Unity вызовет TypeScript и получит ответ.

Потом TypeScript сам отправит сообщение обратно в Unity.

А в конце я покажу, куда подключаются npm-зависимости и почему тут есть отдельная история с WebGL template.

## 0:50-1:30 - Минимальная идея JSPipe

### На экране

Большой простой текст:

```text
Greeting.GetName()
Greeting.SayHello()
```

Потом:

```text
C# module name: Greeting
TS module name: Greeting
```

### Реплика

Главная идея JSPipe простая: мы не делаем сто глобальных JS-функций.

Мы заводим модуль.

Например, `Greeting`.

На C# стороне есть `Greeting`.

На TypeScript стороне тоже есть `Greeting`.

Если имена совпали, они могут обмениваться сообщениями.

Нужно просто отправить событие - используем `Notify`.

Нужен ответ - используем `Call`.

Все. Это уже сильно приятнее, чем археология в `.jslib`.

## 1:30-2:10 - Включаем JSPipe в Unity

### На экране

Unity, файл `Bootstrap.cs`.

### Код

```csharp
using TiltShift.JsPipe;
using UnityEngine;

public class Bootstrap : MonoBehaviour
{
    private void Awake()
    {
#if UNITY_WEBGL && !UNITY_EDITOR
        JsPipeHost.Init();
#endif
    }
}
```

### Реплика

Начинаем с Unity.

Нам нужно один раз поднять host на C# стороне.

Да, тут стоит `UNITY_WEBGL && !UNITY_EDITOR`. Это нормально: JSPipe нужен именно в WebGL build. В Editor обычно делают mock, но сегодня мы не будем утаскивать демо в сторону.

Создаем объект `Bootstrap`, вешаем скрипт, забываем про него.

## 2:10-3:40 - Пишем C# модуль

### На экране

Файл `GreetingModule.cs`.

### Код

```csharp
using Cysharp.Threading.Tasks;
using TiltShift.JsPipe;
using UnityEngine;

public class GreetingModule : JSPipeModule
{
    public override string Name => "Greeting";

    public GreetingModule()
    {
        RegisterHandler<NameRequest>("SayHello", OnSayHello);
    }

    private void OnSayHello(NameRequest request)
    {
        Debug.Log($"Hello from TypeScript: {request.Name}");
    }

    public async UniTask<string> GetUserName()
    {
        var response = await Call<NameResponse>("GetName");
        return response.Name;
    }

    private class NameRequest
    {
        public string Name { get; set; }
    }

    private class NameResponse
    {
        public string Name { get; set; }
    }
}
```

### Реплика

Теперь сам модуль.

Самая важная строка здесь:

```csharp
public override string Name => "Greeting";
```

Это имя - контракт. TypeScript-модуль должен называться так же.

Дальше мы говорим: если с web-стороны придет `SayHello`, вызови `OnSayHello`.

А метод `GetUserName` делает обратное: он вызывает TypeScript-метод `GetName` и ждет ответ.

То есть в одном классе у нас сразу две стороны общения:

входящий handler и исходящий call.

## 3:40-4:20 - Запускаем вызов из Unity

### На экране

Файл `GreetingDemo.cs`.

### Код

```csharp
using Cysharp.Threading.Tasks;
using UnityEngine;

public class GreetingDemo : MonoBehaviour
{
    private GreetingModule _greeting;

    private void Awake()
    {
        _greeting = new GreetingModule();
    }

    private async void Start()
    {
#if UNITY_WEBGL && !UNITY_EDITOR
        await UniTask.Delay(1000);

        var name = await _greeting.GetUserName();
        Debug.Log($"Name from TypeScript: {name}");
#endif
    }
}
```

### Реплика

Теперь используем модуль.

Создаем его в `Awake`. JSPipe зарегистрирует модуль автоматически.

В `Start` вызываем `GetUserName`.

Для демо я ставлю небольшую задержку, чтобы не усложнять разговор lifecycle-событиями. В боевом проекте лучше завязаться на готовность bridge.

## 4:20-5:10 - Поднимаем TypeScript окружение

### На экране

Unity menu:

```text
Tools -> JSPipe -> Install ENV
```

Потом показать файлы:

```text
Assets/upackage.json
Assets/Index.ts
Assets/tsconfig.json
```

### Реплика

Теперь web-сторона.

И вот здесь JSPipe делает приятную вещь: он не заставляет нас писать весь JavaScript в `.jslib`.

Идем в `Tools -> JSPipe -> Install ENV`.

После этого в проекте появляется TypeScript/npm окружение: `upackage.json`, `tsconfig`, entry point и локальные зависимости.

Нас сейчас интересует `Index.ts`. Это место, где мы напишем TypeScript-модуль.

## 5:10-6:30 - Пишем TypeScript модуль

### На экране

Файл `Assets/Index.ts`.

### Код

```ts
import { JSPipeModule } from "./jspipe/JSPipeModule";

class GreetingModule extends JSPipeModule {
  constructor() {
    super("Greeting");

    this.registerHandler("GetName", () => {
      return {
        Name: "Player from TypeScript"
      };
    });
  }

  public sayHello(name: string): void {
    this.notify("SayHello", {
      Name: name
    });
  }
}

const greeting = new GreetingModule();

setTimeout(() => {
  greeting.sayHello("Browser runtime");
}, 2000);
```

### Реплика

Теперь пишем зеркальную часть в TypeScript.

И снова главная строка:

```ts
super("Greeting");
```

Имя совпадает с C# модулем.

Регистрируем `GetName`. Это тот метод, который C# вызовет через `Call`.

А через `sayHello` отправляем `Notify("SayHello")` обратно в Unity.

То есть сейчас у нас уже есть двусторонняя связь:

C# спрашивает имя.

TypeScript отвечает.

TypeScript отправляет привет.

C# принимает.

## 6:30-7:30 - Собираем WebGL и смотрим результат

### На экране

Unity Build Settings -> WebGL -> Build.

Потом браузер и console/logs.

### Реплика

Собираем WebGL.

Во время сборки JSPipe соберет TypeScript, подготовит JS Pre, соберет WebGL template и подключит нужные скрипты.

Нам не нужно руками копировать bundle, вставлять script tag и помнить, какой файл куда положить.

Открываем билд.

И вот результат:

```text
Name from TypeScript: Player from TypeScript
Hello from TypeScript: Browser runtime
```

Это маленькое демо, но в нем уже есть главное: C# и TypeScript общаются не через случайные глобальные функции, а через модульный bridge.

## 7:30-8:40 - Зачем тут npm

### На экране

Открыть `upackage.json`.

### Код

```json
{
  "main": "./Index.ts",
  "template": "./Template.ts",
  "dependencies": {
    "zod": "^3.23.8"
  }
}
```

### Реплика

Теперь вопрос: а зачем вообще весь этот TypeScript/npm слой?

Потому что в реальном WebGL-проекте вы почти наверняка захотите подключить готовую web-библиотеку.

Analytics SDK.

Wallet connector.

SDK платежей.

Валидацию.

Crypto.

QR generator.

Что угодно из npm.

В `upackage.json` можно добавить зависимости. После изменения запускаем `Install ENV`, и дальше импортируем пакет прямо в TypeScript.

Например:

```ts
import { z } from "zod";
```

И это уже нормальный web-код, а не минифицированный кусок, случайно вставленный в `.jslib`.

## 8:40-9:50 - `main` и `template`: две разные жизни JavaScript

### На экране

Простая схема:

```text
template -> runs in HTML page before Unity
main     -> runs as JSPre near Unity/WebAssembly
```

### Реплика

Еще одна важная штука: в Unity WebGL есть не один "JavaScript".

Есть код страницы. Он должен выполниться рано: до запуска Unity, до WebAssembly, пока грузится HTML.

А есть код bridge-слоя. Он должен жить рядом с Unity runtime и регистрировать модули.

Поэтому в JSPipe есть два entry point.

`template` - для HTML-страницы.

`main` - для JS Pre и общения с Unity.

Если не разделять эти две зоны, очень быстро начинается классика: "а почему этот объект undefined, если вчера работало?"

## 9:50-10:40 - Ошибки и диагностика

### На экране

Показать короткий override.

### Код

```csharp
public override void OnRxPacket(JsPipePacket packet)
{
    Debug.Log($"[{Name}] Packet: {packet.Id}");

    base.OnRxPacket(packet);

    if (packet.Response?.Error != null)
    {
        Debug.LogError(packet.Response.Error.Message);
    }
}
```

### Реплика

Когда bridge становится частью проекта, хочется понимать, что по нему ходит.

Для этого у модуля можно переопределить `OnRxPacket`.

Туда удобно добавить логирование, метрики, трассировку или обработку ошибок.

Главное - не забыть `base.OnRxPacket(packet)`, иначе вы сами же перехватите пакет и не дадите JSPipe его обработать.

## 10:40-11:40 - Самые частые ошибки

### На экране

Список крупным текстом.

```text
1. Module names do not match
2. Handler name typo
3. Forgot Install ENV
4. Trying to run browser bridge in Editor
5. Template file missing
```

### Реплика

Пять вещей, на которых проще всего споткнуться.

Первое: имя модуля. Если в C# `Greeting`, а в TypeScript `Greting` без второй `e`, ничего хорошего не будет.

Второе: имя handler. `Call("GetName")` должен совпадать с `registerHandler("GetName")`.

Третье: добавили npm dependency, но забыли снова выполнить `Install ENV`.

Четвертое: пытаетесь проверить browser bridge в Editor. Для этого лучше делать mock.

Пятое: проблемы с WebGL template. Для нормального flow должен быть `JSPipeTemplate/index.html`.

## 11:40-12:30 - Финал

### На экране

Вернуться к коду C# и TypeScript рядом.

### Реплика

Итого.

Если вам нужно один раз вызвать JavaScript из Unity, `.jslib` вполне нормальный вариант.

Но если Unity WebGL становится частью web-приложения, появляются async, npm, browser SDK, template-код и несколько интеграций - лучше думать не отдельными JS-функциями, а bridge-слоем.

JSPipe как раз про это:

модули вместо глобальной свалки,

`Notify` для событий,

`Call` для request/response,

TypeScript и npm как нормальная часть проекта,

и сборка, встроенная в WebGL build flow.

На этом все. В следующем видео можно взять уже реальный сценарий: например, авторизацию через browser SDK или wallet connection из Unity WebGL.

## Ритм монтажа

Держать видео бодрым:

- не читать все строки кода;
- проговаривать только ключевые места;
- после каждого куска кода сразу объяснять, зачем он нужен;
- чаще возвращаться к результату в console;
- не углубляться в build internals;
- documentation details оставить для описания под видео.

## Что подготовить до записи

- Unity проект с установленным JSPipe.
- WebGL platform уже выбрана.
- `Cysharp.Threading.Tasks` доступен, если используется `UniTask`.
- `Install ENV` заранее проверен.
- Минимальный `JSPipeTemplate/index.html` существует.
- Первый WebGL build уже один раз успешно собран.
- Открыт редактор кода с крупным шрифтом.
- Browser console готова.
- Unity Console очищена перед записью.

## Описание под видео

В этом видео показываю JSPipe - bridge для Unity WebGL, который позволяет связать C# внутри Unity с TypeScript/npm-кодом в браузере.

Что внутри:

- первый `JSPipeModule` на C#;
- зеркальный модуль на TypeScript;
- `Call` для request/response;
- `Notify` для событий;
- подключение npm-зависимостей через `upackage.json`;
- разница между `main` и `template`;
- базовая диагностика через `OnRxPacket`.

## Короткая версия для Shorts

Unity WebGL уже работает в браузере.

Но если вы хотите подключить npm-пакет, SDK аналитики или wallet provider, обычный `.jslib` быстро превращается в кашу.

В JSPipe C# и TypeScript общаются через модули.

C#:

```csharp
public override string Name => "Greeting";
var response = await Call<NameResponse>("GetName");
```

TypeScript:

```ts
super("Greeting");
this.registerHandler("GetName", () => ({ Name: "Player" }));
```

Имена модулей совпали - bridge работает.

Нужно событие - `Notify`.

Нужен ответ - `Call`.

А npm-зависимости подключаются как нормальный TypeScript-код.

