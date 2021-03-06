# Тестирование стилей и структуры компонентов Vue.js {#chapter-3}

Пока что в тестах мы использовали [снимки Jest](https://facebook.github.io/jest/docs/snapshot-testing.html). Это здорово, но иногда мы хотим проверить (или утверждать что-либо) что-то более конкретное.

Хотя вы можете получить доступ к экземпляру Vue через [`cmp.vm`](https://github.com/alexjoverm/vue-testing-series/blob/master/test/MessageList.test.js#L17), в вашем распоряжении есть набор утилит, чтобы получить его проще. Давайте посмотрим, что мы можем сделать.

## Объект Wrapper

`Wrapper` — главный объект Vue Test Utils. Это тип, возвращаемый функциями `mount`,` shallow`, `find` и `findAll`. Вы можете [посмотреть здесь](https://github.com/vuejs/vue-test-utils/blob/v1.0.0-beta.27/packages/test-utils/types/index.d.ts) весь API и типы.

### Методы `find` и `findAll`

Они принимают параметр [Selector](https://github.com/vuejs/vue-test-utils/blob/v1.0.0-beta.27/packages/test-utils/types/index.d.ts#L92) в качестве аргумента, который может быть как CSS-селектором, так и Vue-компонентом.

Поэтому мы можем сделать что-то подобное:

{lang=javascript}
    const messageListCmp = mount(MessageList);

    expect(messageListCmp.find('.message').element).toBeInstanceOf(HTMLElement);

    // Или даже вызывать его несколько раз
    let el = messageListCmp.find('.message').find('span').element;

    // Хотя предыдущий пример мы могли сделать это короче
    let el = messageListCmp.find('.message span').element;

### Утверждение структуры и стиля

Давайте добавим больше тестов для `MessageList.test.js`:

{lang=javascript}
    it('это компонент MessageList', () => {
      expect(messageListCmp.is(MessageList)).toBe(true);

      // Или с помощью CSS-селектора
      expect(messageListCmp.is('ul')).toBe(true);
    });

    it('содержит компонент Message', () => {
      expect(cmp.contains(Message)).toBe(true);

      // Или с помощью CSS-селектора
      expect(cmp.contains('.message')).toBe(true);
    });

Здесь мы используем `is` для утверждения типа корневого компонента и `contains` для проверки существования дочерних компонентов. Так же, как и `find`, они получают объект типа Selector, который может быть CSS-селектором (тип Selector) или компонентом (тип Component).

У нас есть некоторые утилиты для утверждения **экземпляра Vue**:

{lang=javascript}
    it('Компоненты MessageList и Message являются экземплярами Vue', () => {
      expect(cmp.isVueInstance()).toBe(true);
      expect(cmp.find(Message).isVueInstance()).toBe(true);
    });

Теперь мы собираемся проверить **структуру** компонента более подробно:

{lang=javascript}
    it('Существует элемент Message', () => {
      expect(cmp.find('.message').exists()).toBe(true);
    });

    it('Message не пустой', () => {
      expect(cmp.find(Message).isEmpty()).toBe(false);
    });

    it('У Message есть атрибут класса со значением "message"', () => {
      expect(cmp.attributes().class).toBe('message');
    });

Методы `exists`, `isEmpty` очень удобные на практике.

Теперь у нас есть `classes()` и `attributes().style` для проверки **стилей**. Давайте обновим компонент `Message.vue` со стилем, поскольку `attributes().style` проверяет только встроенные стили:

{lang=html}
    <li style="margin-top: 10px" class="message">{{message}}</li>

Вот тесты для этого:

{lang=javascript}
    it('У компонента Message задан класс .message', () => {
      expect(cmp.find(Message).classes()).toContain('message');
    });

    it('У компонента Message определён стиль `padding-top: 10`', () => {
      expect(cmp.find(Message).attributes().style).toBe('margin-top: 10px;');
    });

## Резюме

Для упрощения тестирования компонентов Vue существует куча утилит. Вы можете найти их все в [файле типизации](https://github.com/vuejs/vue-test-utils/blob/v1.0.0-beta.27/packages/test-utils/types/index.d.ts).

Вы можете найти рабочий код этой главы в [этом репозитории](https://github.com/alexjoverm/vue-testing-series/blob/Test-Styles-and-Structure-in-Vue-js-and-Jest/test/MessageList.test.js).
