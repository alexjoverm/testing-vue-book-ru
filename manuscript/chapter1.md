# Написание первого модульного теста компонента Vue.js {#chapter-1}

Узнайте, как писать модульные тесты с помощью официальных инструментов Vue.js и фреймворка Jest.

[vue-test-utils](https://github.com/vuejs/vue-test-utils), официальная библиотека тестирования VueJS, основанная на [avoriaz](https://github.com/eddyerburgh/avoriaz), уже не за горами. [@EddYerburgh](https://twitter.com/EddYerburgh) действительно делает очень хорошую работу, создавая его. Она предоставляет все необходимые инструменты для легкого написания модульного теста в приложении, сделанном на VueJS.

[Jest](https://facebook.github.io/jest), с другой стороны, представляет собой фреймворк для тестирования, разработанный в Facebook, позволяющий очень быстро тестировать с потрясающими такими возможностями как:

- Почти нет настроек по умолчанию
- Очень классный интерактивный режим
- Возможность выполнения тестов параллельно
- Шпионы (spies), заглушки (stubs) и подставные объекты (mocks) из коробки
- Встроенное покрытие кода
- Тестирование снимками
- Утилиты имитации модулей

Возможно, вы уже написали тест без этих инструментов, и просто используя karma + mocha + chai + sinon + ..., но вы увидите, насколько проще это может быть 😉.

## Настройка примера проекта vue-test

Начнем с создания нового проекта с использованием [`vue-cli`](https://github.com/vuejs/vue-cli), отвечая NO на все вопросы с вариантом yes или no:

```bash
npm install -g vue-cli
vue init webpack vue-test
cd vue-test
```

Затем нам нужно будет установить кое-какие зависимости:

```bash
# Установка зависимостей
npm i -D jest jest-vue-preprocessor babel-jest
```

[`jest-vue-preprocessor`](https://github.com/vire/jest-vue-preprocessor) необходим, чтобы Jest понимал файлы с расширением `.vue`, а [`babel-jest`](https://github.com/babel/babel-jest) нужен для интеграции с Babel.

Согласно `vue-test-utils`, он ~~еще не выпущен, но теперь вы можете добавить его в свой `package.json` из источника~~:

**Обновление (10.10.2017)**: он может быть установлен уже из npm, так как версия `beta.1` была опубликована.

```bash
npm i -D vue-test-utils
```

Давайте добавим следующую конфигурацию для Jest в `package.json`:

```json
...
"jest": {
  "moduleNameMapper": {
    "^vue$": "vue/dist/vue.common.js"
  },
  "moduleFileExtensions": [
    "js",
    "vue"
  ],
  "transform": {
    "^.+\\.js$": "<rootDir>/node_modules/babel-jest",
    ".*\\.(vue)$": "<rootDir>/node_modules/jest-vue-preprocessor"
  }
}
...
```

`moduleFileExtensions` указывает Jest, какие расширения искать, а `transform` — какой препроцессор использовать для расширения файла.

Наконец, добавьте скрипт `test` в `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    ...
  },
  ...
}
```

## Тестирование компонента

Далее я буду использовать однофайловые компоненты, и я не проверял, будет ли они работать, если компонент разделить на собственные файлы `html`, `css` или `js`, поэтому давайте предположим, что вы точно также делаете.

Сначала создайте компонент `MessageList.vue` в каталоге `src/components`:

```html
<template>
    <ul>
        <li v-for="message in messages">
            {{ message }}
        </li>
    </ul>
</template>

<script>
export default {
  name: 'list',
  props: ['messages']
}
</script>
```

И обновите `App.vue`, чтобы использовать его, следующим образом:

```html
<template>
  <div id="app">
    <MessageList :messages="messages"/>
  </div>
</template>

<script>
import MessageList from './components/MessageList'

export default {
  name: 'app',
  data: () => ({ messages: ['Привет, Джон', 'Как дела, Пако?'] }),
  components: {
    MessageList
  }
}
</script>
```

У нас уже есть пара компонентов, которые мы можем протестировать. Давайте создадим каталог `test` в корне проекта и файл `App.test.js`:

```javascript
import Vue from 'vue'
import App from '../src/App'

describe('App.test.js', () => {
  let cmp, vm

  beforeEach(() => {
    cmp = Vue.extend(App) // Создать копию исходного компонента
    vm = new cmp({
      data: { // Заменить значение данных на эти поддельные данные
        messages: ['Cat']
      }
    }).$mount() // Создать экземпляр и примонтировать компонент
  })

  it('сообщения идентичны ["Cat"]', () => {
    expect(vm.messages).toEqual(['Cat'])
  })
})
```

I> ## Примечание
I> Обычно сообщения в функции `it` пишутся на английском языке, но сейчас и далее они будут переведены на русский язык.

Right now, if we run `npm test` (or `npm t` as a shorthand version), the test should run and pass. Since we're modifying the tests, let's better run it in **watch mode**:

В данный момент, если мы выполним `npm test` (или `npm t` как сокращенная версия этой команды), тест должен запуститься и успешно пройти. Поскольку мы изменяем тесты, давайте лучше запустим их **в режиме просмотра (watch mode)**:

```shell
npm t -- --watch
```

### Проблема с вложенными компонентами

Этот тест слишком прост. Давайте также проверим, что вывод соответствует ожидаемому. Для этого мы можем использовать удивительную возможность снимков (snapshots) Jest, которая будет генерировать снимок вывода и проводить сравнение с ним в предстоящих запусках. добавьте после предыдущего `it` в `App.test.js`:

```javascript
it('имеет ожидаемую структуру HTML', () => {
  expect(vm.$el).toMatchSnapshot()
})
```

Это создаст файл `test/__snapshots__/App.test.js.snap`. Давайте откроем и изучим его:

```javascript
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`App.test.js имеет ожидаемую структуру HTML 1`] = `
<div
  id="app"
>
  <ul>
    <li>
      Cat
    </li>
  </ul>
</div>
`;
```

Если вы не заметили, здесь есть большая проблема: компонент `MessageList` был также отрисован. **Модульные тесты должны быть протестированы как независимые единицы**, а это значит, что в `App.test.js` мы хотим протестировать компонент `App` и больше ни какой другой.

Это может стать причиной нескольких проблем. Представьте себе, например, что дочерние компоненты (`MessageList` в этом случае) выполняют операции с побочными эффектами на хуке `created`, такие как вызов `fetch`, действие Vuex или изменения состояния? Это то, чего мы определенно не хотим.

К счастью, **поверхностная или неглубокая отрисовка (Shallow Rendering)** хорошо справляется с этим.

### Что такое поверхностная отрисовка?

[Поверхностная отрисовка](http://airbnb.io/enzyme/docs/api/shallow.html) — это метод, который гарантирует, что ваш компонент выполняет отрисовку без дочерних элементов. Это полезно для:

- Тестирование только компонента, который вы хотите протестировать (это как раз то, что означает модульный тест)
- Избегание побочных эффектов, которые могу быть у дочерних компонентов, например, создание HTTP-вызовов, вызовы действий хранилища...

## Тестирование компонента с помощью vue-test-utils

`vue-test-utils` предоставим нам поверхностную отрисовку среди прочего функционала. Мы могли бы переписать предыдущий тест следующим образом:

```javascript
import { shallow } from 'vue-test-utils'
import App from '../src/App'

describe('App.test.js', () => {
  let cmp

  beforeEach(() => {
    cmp = shallow(App, { // Создать поверхностный экземпляр компонента
      data: {
        messages: ['Cat']
      }
    })
  })

  it('сообщения идентичны ["Cat"]', () => {
    // Внутри cmp.vm, мы имеем доступ ко всем методам экземпляра Vue
    expect(cmp.vm.messages).toEqual(['Cat'])
  })

  it('имеет ожидаемую структуру HTML', () => {
    expect(cmp.element).toMatchSnapshot()
  })
})
```

И теперь, если вы все еще используете Jest в режиме просмотра, вы увидите, что тест все еще проходит, но снимок не соответствует. Нажмите `u`, чтобы пересоздать его. Откройте и проверьте его снова:

```javascript
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`App.test.js имеет ожидаемую структуру HTML 1`] = `
<div
  id="app"
>
  <!--  -->
</div>
`;
```

Вы видите? Теперь дочерние элементы не отрисовались, и мы протестировали компонент `App` **полностью изолированным** от дерева компонентов. Кроме того, если у вас есть какие-либо хуки, например `created`, в дочерних компонентах, ни один из них не был вызван 😉.

Если вам интересно, **как реализуется поверхностная отрисовка**, посмотрите [исходный код](https://github.com/vuejs/vue-test-utils/blob/dev/packages/shared/stub-components.js), и вы увидите, что в основном создаются заглушки для ключа `components`, метода `render` и хуки жизненного цикла.

В том же духе вы можете реализовать тест `MessageList.test.js`, как представлено ниже:

```javascript
import { shallow } from 'vue-test-utils'
import MessageList from '../src/components/MessageList'

describe('MessageList.test.js', () => {
  let cmp

  beforeEach(() => {
    cmp = shallow(MessageList, {
      // Помните, что входные параметры переопределяются с помощью `propsData`
      propsData: {
        messages: ['Cat']
      }
    })
  })

  it('получен массив ["Cat"] как свойство сообщения', () => {
    expect(cmp.vm.messages).toEqual(['Cat'])
  })

  it('имеет ожидаемую структуру HTML', () => {
    expect(cmp.element).toMatchSnapshot()
  })
})
```

Полную версию примера вы можете найти в [репозитории на GitHub](https://github.com/alexjoverm/vue-testing-series/tree/lesson-1).