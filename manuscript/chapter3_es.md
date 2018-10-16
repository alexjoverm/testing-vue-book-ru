# Testing de Estilos y Estructura de Componentes Vue.js en Jest

[vue-test-utils](https://github.com/vuejs/vue-test-utils) nos da un conjunto de herramientas útiles para testear componentes.

Hasta ahora, en los test unitarios hemos utilizado [Snapshots de Jest](https://facebook.github.io/jest/docs/snapshot-testing.html). Muy bien, pero a veces queremos ir un poco más allá.

Aunque se puede acceder a la instancia de Vue() vía [`cmp.vm`](https://github.com/alexjoverm/vue-testing-series/blob/master/test/MessageList.test.js#L17), tenemos una serie de mecanismos a nuestra disposición que nos facilitarán la vida.

## El Objeto _Wrapper_

`Wrapper` es el objecto principal de `vue-test-utils`. Es el tipo que devuelven las funciones `mount`, `shallow`, `find` y `findAll`. Se puede [ver aquí](https://github.com/vuejs/vue-test-utils/blob/master/types/index.d.ts#L34) toda la API y tipados (_typings_).

### Los métodos `find` y `findAll`

Aceptan un [Selector](https://github.com/vuejs/vue-test-utils/blob/master/types/index.d.ts#L11) como argumento, el cual puede ser un Selector de CSS o un Componente de Vue.

Así que podemos, por ejemplo, hacer:

```javascript
  let cmp = mount(MessageList)
  expect(cmp.find('.message').element).toBeInstanceOf(HTMLElement)

  // O incluso usarlo varias veces
  let el = cmp.find('.message').find('span').element

  // Aunque, para este ejemplo, podemos hacerlo así directamente:
  let el = cmp.find('.message span').element
```

<!-- TODO: Asserting? -->

### Asserting Structure and Style

Vamos a añadir más tests a `MessageList.test.js`:

```javascript
it('is a MessageList component', () => {
  expect(cmp.is(MessageList)).toBe(true)

  // O como Selector CSS del _markup_ generado:
  expect(cmp.is('ul')).toBe(true)
})

it('contains a Message component', () => {
  expect(cmp.contains(Message)).toBe(true)

  // O como Selector CSS del _markup_ generado:
  expect(cmp.contains('.message')).toBe(true)
})
```

Usamos `is` para __afirmar__ sobre el componente raíz y `contains` para _checkear_ si existen sub-componentes. Así como, `find`, todos pueden recibir un Selector: ya sea un Selector de CSS o un Componente

We have some utils to assert the **Vue instance**:

```javascript
it('Both MessageList and Message are vue instances', () => {
  expect(cmp.isVueInstance()).toBe(true)
  expect(cmp.find(Message).isVueInstance()).toBe(true)
})
```

Ahora, testearemos la **Estructurs** en mayor detalle:

```javascript
it('Message element exists', () => {
  expect(cmp.find('.message').exists()).toBe(true)
})

it('Message is not empty', () => {
  expect(cmp.find(Message).isEmpty()).toBe(false)
})

it('Message has a class attribute set to "message"', () => {
  // para asertar con "class", el método `hasClass`es muy útil
  expect(cmp.find(Message).hasAttribute('class', 'message')).toBe(true)
})
```

Los métodos `exists`, `isEmpty` and `hasAttribute` son muy útiles para estas comprobaciones.

Asimismo, tenemos `hasClass` y `hasStyle` para testear **Estilos**. Así que, actualicemos el componente `Message.vue` con unos estilos, dado que `hasStyle` aserta sólo en estilos inline:

```html
<li style="margin-top: 10px" class="message">{{message}}</li>
```

El test sería:

```javascript
it('Message component has the .message class', () => {
  expect(cmp.find(Message).hasClass('message')).toBe(true)
})

it('Message component has style padding-top: 10', () => {
  expect(cmp.find(Message).hasStyle('padding-top', '10')).toBe(true)
})
```

### Los métodos `get`

Como hemos visto, tenemos unos _utils_ bastante cómodos para testear componentes de Vue. La mayoría usan la nomenclatura `hasX`, lo cual está muy bien, pero tener un `getX` nos brinda una mejor experiencia de testeo, en términos de flexibildiad y _debuggeo_. Podríamos re-escribir el siguiente ejemplo:

```javascript
// usando `has`
expect(cmp.find(Message).hasAttribute('class', 'message')).toBe(true)

// por el método `get`
expect(cmp.find(Message).getAttribute('class')).toBe('message')
```

<!-- TODO: revisar issue -->

This is [under discussion](https://github.com/vuejs/vue-test-utils/issues/27) and seems like it will be added to the library at some point.

## Resumiendo

Tenemos un montón de métodos para hacer el testing de componentes Vue mucho más sencillo. Podemos encontrarlos todos en [el archivo de _typings_](https://github.com/vuejs/vue-test-utils/blob/master/types/index.d.ts).

Podréis encontrar el código en [este repo](https://github.com/alexjoverm/vue-testing-series/blob/Test-Styles-and-Structure-in-Vue-js-and-Jest/test/MessageList.test.js).