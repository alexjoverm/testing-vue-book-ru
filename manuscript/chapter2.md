# Тестирование компонентов с глубокой вложенностью

Давайте посмотрим, как использовать Vue Test Utils для тестирования полностью отрисованного дерева компонента.

В [первой главе](#chapter-1) мы узнали, как использовать поверхностную отрисовку для тестирования компонента изолированно, предотвращая отрисовку поддерева компонента, т.е. его дочерних элементов.

Но в некоторых случаях мы могли бы протестировать компоненты, которые ведут себя как группа, или [молекулы](http://atomicdesign.bradfrost.com/chapter-2/#molecules), как указано в книге Atomic Design. Имейте в виду, что это относится к [презентационным компонентам](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), поскольку они не знают о состоянии и логике приложения. В большинстве случаев вы хотите использовать поверхностную отрисовку для компонентов-контейнеров.

## Добавление компонента Message

В случае компонентов Message и MessageList, помимо написания для них модульных тестов, мы могли бы также протестировать их как единое целое.

Начнём с создания файла `components/Message.vue`, в котором определим компонент Message:

{lang=html}
    <template>
      <li class="message">{{message}}</li>
    </template>

    <script>
      export default {
        props: ['message']
      }
    </script>

И обновим `components/MessageList.vue` для его использования:

{lang=html}
    <template>
      <ul>
        <Message :message="message" v-for="message in messages"/>
      </ul>
    </template>

    <script>
    import Message from './Message'

    export default {
      props: ['messages'],
      components: {
        Message
      }
    }
    </script>

## Тестирование `MessageList` с компонентом `Message`

Для тестирования MessageList с полной (глубокой) отрисовкой, нам нужно просто использовать `mount` вместо `shallow` в ранее созданном тесте в файле `test/MessageList.test.js`:

{lang=javascript}
    import { mount } from '@vue/test-utils'
    import MessageList from '../src/components/MessageList'

    describe('MessageList.test.js', () => {
      let cmp;

      beforeEach(() => {
        cmp = mount(MessageList, {
          // Помните, что входные параметры переопределяется с помощью `propsData`
          propsData: {
            messages: ['Кот']
          }
        });
      });

      it('получен массив ["Кот"] в качестве входного параметра message', () => {
        expect(cmp.vm.messages).toEqual(['Кот'])
      });

      it('имеет ожидаемую структуру HTML', () => {
        expect(cmp.element).toMatchSnapshot()
      });
    });

I> Кстати говоря, вы поняли, что такое `beforeEach`? Это очень элегантный способ создания нового компонента перед каждым выполнением теста, что очень важно при модульном тестировании, поскольку он определяет, что тест не должен зависеть друг от друга.

Как `mount`, так и `shallow` используют точно такой же API, разница только в отрисовке. Я покажу вам постепенно API далее в этой серии.

Если вы запустите `npm t`, то увидите, что тест завершился неудачей, потому что снимок не соответствует `MessageList.test.js`. Чтобы пересоздать его, запустите выполнение тестов с помощью опции `-u`:

{lang=bash, linenos=off}
    npm t -- -u

Затем, если вы откроете и посмотрите содержимое `test/__snapshots__/MessageList.test.js.snap`, то увидите присутствие `class="message"`, что означает, что компонент отрисован в соотвествии с последними изменениями.

{lang=javascript}
    // Jest Snapshot v1, https://goo.gl/fbAQLP

    exports[`MessageList.test.js имеет ожидаемую структуру HTML 1`] = `
    <ul>
      <li
        class="message"
      >
        Кот
      </li>
    </ul>
    `;

Помните о том, чтобы **избегать использование глубокой отрисовки в случаях, когда могут быть побочные эффекты**, поскольку хуки компонентов дочерних элементов, такие как `created` и `mount`, будут запускаться, и там могут быть HTTP-вызовы или другие операции, которые таким образом будут выполнены, когда как мы при тестировании мы этого не хотим. Если вы хотите попробовать в действии то, о чем я только что написал, добавьте в компонент `Message.vue` вызов `console.log` в хуке `created`:

{lang=javascript}
    export default {
      props: ['message'],
      created() {
        console.log('СОЗДАН!');
      }
    };

Теперь, если вы снова запустите тесты с помощью `npm t`, увидите текст `"СОЗДАН!"` в выводе терминала. Поэтому будьте осторожны.

Вы можете найти [полный пример на GitHub](https://github.com/alexjoverm/vue-testing-series/tree/Test-fully-rendered-Vue-js-Components-in-Jest).
