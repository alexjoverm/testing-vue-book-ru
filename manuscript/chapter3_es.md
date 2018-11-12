# Testing de Estilos y Estructura de Componentes Vue.js en Jest

[vue-test-utils](https://github.com/vuejs/vue-test-utils) nos da un conjunto de herramientas útiles para testear componentes.

Hasta ahora, en los test unitarios hemos utilizado [Snapshots de Jest](https://facebook.github.io/jest/docs/snapshot-testing.html). Muy bien, pero a veces queremos ir un poco más allá.

Aunque se puede acceder a la instancia de Vue() vía [`cmp.vm`](https://github.com/alexjoverm/vue-testing-series/blob/master/test/ListaMensajes.test.js#L17), tenemos una serie de mecanismos a nuestra disposición que nos facilitarán la vida.

## El Objeto _Wrapper_

`Wrapper` es el objecto principal de `vue-test-utils`. Es el tipo que devuelven las funciones `mount`, `shallowMount`, `find` y `findAll`. Se puede [ver aquí](https://github.com/vuejs/vue-test-utils/blob/master/types/index.d.ts#L34) toda la API y tipados (_typings_).

### Los métodos `find` y `findAll`

Aceptan un [Selector](https://github.com/vuejs/vue-test-utils/blob/master/types/index.d.ts#L11) como argumento, el cual puede ser un Selector de CSS o un Componente de Vue.

Así que podemos, por ejemplo, hacer:

```javascript
let cmp = mount(ListaMensajes)
expect(cmp.find('.mensaje').element).toBeInstanceOf(HTMLElement)

// O incluso usarlo varias veces
let el = cmp.find('.mensaje').find('span').element

// Aunque, para este ejemplo, podemos hacerlo así directamente:
let el = cmp.find('.mensaje span').element
```

<!-- TODO: Asserting? -->

### Asserting Structure and Style

Vamos a añadir más tests a `ListaMensajes.test.js`:

```javascript
it('es un componente ListaMensajes', () => {
  expect(cmp.is(ListaMensajes)).toBe(true)

  // O como Selector CSS del _markup_ generado:
  expect(cmp.is('ul')).toBe(true)
})

it('contiene un cmoponente Mensaje', () => {
  expect(cmp.contains(Mensaje)).toBe(true)

  // O como Selector CSS del _markup_ generado:
  expect(cmp.contains('.mensaje')).toBe(true)
})
```

Usamos `is` para __afirmar__ sobre el componente raíz y `contains` para _checkear_ si existen sub-componentes. Así como, `find`, todos pueden recibir un Selector: ya sea un Selector de CSS o un Componente

We have some utils to assert the **Vue instance**:

```javascript
it('Ambos ListaMensajes y Mensaje son instancias de Vue', () => {
  expect(cmp.isVueInstance()).toBe(true)
  expect(cmp.find(Mensaje).isVueInstance()).toBe(true)
})
```

Ahora, testearemos la **Estructurs** en mayor detalle:

```javascript
it('elemento Mensaje existe', () => {
  expect(cmp.find('.mensaje').exists()).toBe(true)
})

it('Mensaje no es vacío', () => {
  expect(cmp.find(Mensaje).isEmpty()).toBe(false)
})

it('Mensaje tiene un atributo de clase de valor "mensaje"', () => {
  // para asertar con "class", el método `hasClass`es muy útil
  expect(cmp.find(Mensaje).hasAttribute('class', 'mensaje')).toBe(true)
})
```

Los métodos `exists`, `isEmpty` and `hasAttribute` son muy útiles para estas comprobaciones.

Asimismo, tenemos `hasClass` y `hasStyle` para testear **Estilos**. Así que, actualicemos el componente `Mensaje.vue` con unos estilos, dado que `hasStyle` aserta sólo en estilos inline:

```html
<li style="margin-top: 10px" class="mensaje">{{mensaje}}</li>
```

El test sería:

```javascript
it('Mensaje tiene la clase .mensaje', () => {
  expect(cmp.find(Mensaje).classes()).toContain('mensaje')
})

it('Mensaje tiene el estilo padding-top: 10', () => {
  expect(cmp.find(Mensaje).attributes().style).toBe('padding-top: 10px;')
})
```

## Resumiendo

Tenemos un montón de métodos para hacer el testing de componentes Vue mucho más sencillo. Podemos encontrarlos todos en [el archivo de _typings_](https://github.com/vuejs/vue-test-utils/blob/master/types/index.d.ts).

Podréis encontrar el código en [este repo](https://github.com/alexjoverm/vue-testing-series/blob/Test-Styles-and-Structure-in-Vue-js-and-Jest/test/ListaMensajes.test.js).