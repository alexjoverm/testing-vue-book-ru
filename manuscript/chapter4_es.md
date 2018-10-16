# Testeo de Propiedades (_Props_) y _Custom Events_ en Componentes Vue.js con Jest

Hay diferentes maneras de testear propeidades (_props_), eventos y eventos _custom_.

Las _propsData_ son atributos particulares que se pasan de componentes padres a hijos. Los _custom events_ sirven para justo lo contrario, mandan lo que nosotr@s queramos al padre directo. Combinando ambos, tenemos la **comunicación** de componentes Vue.js en ambas direcciones del árbol: escalando o descendiendo.

Con el testeo unitario, testear las entradas y salidas (_props_y _custom events_) significa testear cómo se comporta un componente cuando recibe y emite información, de manera aislada. Vamos a ensucarnos las manos!

## Propiedades

Cuando estamos testeando _props_, testeamos cómo se comporta un componente cuando se le pasan ciertas propeidades. Pero antes, una nota importante:

> Para pasarlas, usaremos `propsData`, en vez de `props`. Éste último es para definir propiedades en el componente, no para pasarlas.

Crearemos un archivo `mensaje.test.js` y añadiremos lo siguiente:

```javascript
 describe('mensaje.test.js', () => {
  let cmp

  describe('Propiedades', () => {
    // @TODO
  })
})
```

Podemos agrupar _test cases_ en bloques _describe_ par anidarlos. Así, podremos usar esta estrategia para agrupar los tests por ciertas propiedades y eventos.

Vamos a utilizar un función factoría para crear el componente mensaje, pasándole propiedades:

```javascript
const createCmp = propsData => mount(mensaje, { propsData })
```

### Testeando la existencia de propiedades

Lo primero sería testear si una propiedad existe o no. Recordemos que el componente `mensaje.vue` tiene una propiedad `mensaje`, así que afirmaremos que recibe esa propiedad correctamente. _Vue-test-utils_ viene con una función `hasProp(prop, value)`, que nos será muy útil en este caso:

```javascript
it('Tiene la propiedad `mensaje`', () => {
  cmp = createCmp({ mensaje: 'Hola' })
  expect(cmp.hasProp('mensaje', 'Hola')).toBeTruthy()
})
```

<!-- TODO: ?? -->

Las propiedades se comportan de manera que se recibirán sólo si han sido declaradas en el componente. O sea, que si pasamos **una propiedad que no ha sido definida, no llegará al componente**. Para comprobarlo, usaremos una propiedad inexistente:

```javascript
it('No tiene una propiedad llamada "gato"', () => {
  cmp = createCmp({ gato: 'Hola' })
  expect(cmp.hasProp('gato', 'Hola')).toBeFalsy()
})
```

Sin embargo, este test fallará porque Vue tiene (atributos sin propiedades) [non-props attributes](https://vuejs.org/v2/guide/components.html#Non-Prop-Attributes) que la setea en la raíz del componente `mensaje`, con lo que será recomocida como una `prop` y el test devolverá `true`. Si lo cambiamos a `toBeTruty` haremos que el test pase:

```javascript
it('No tiene una propiedad llamada "gato"', () => {
  cmp = createCmp({ gato: 'Hola' });
  expect(cmp.hasProp('gato', 'Hola')).toBeTruthy()
})
```

Podemos testear un valor por defecto (**default value**) tambi´n. En nuestro `mensaje.vue` cambiaremos sus `props` de la siguiente manera:

```javascript
props: {
  mensaje: String,
  autor: {
    type: String,
    default: 'Paco'
  }
},
```

El test podría ser:

```javascript
it('Paco es el autor por defecto', () => {
  cmp = createCmp({ mensaje: 'hey' })
  expect(cmp.hasProp('autor', 'Paco')).toBeTruthy()
})
```

### Testeando validaciones de propiedades

Las Propiedades también pueden tener reglas de validación, asegurar que son obligatorias o de que tienen un o unos determinados tipo de valor. Cambiamos la propiedad `mensaje` de la siguiente manera:

```javascript
props: {
  mensaje: {
    type: String,
    required: true,
    validator: m => m.length > 1
  }
}
```

Por ir un poco más allá, podemos usar constructores personalizados de tipos o reglas de validación propias, cómo podemos ver en [la documentación](https://vuejs.org/v2/guide/components.html#Prop-Validation). De momento, solo lo mostraremos como ejemplo:

```javascript
class Mensaje {}
...
props: {
  mensaje: {
    type: Mensaje, // Como si usaramos la comprobación de una instancia (`instance of`)
    ...
    }
  }
}
```

Cuando unavalidación no se cumple, Vue mostrará un `console.error`. Por ejemplo, para `createCmp({ mensaje: 1 })`, mostrará es siguiente error:

```
[Vue warn]: Invalid prop: type check failed for prop "mensaje". Expected String, got Number.
(found in <Root>)
```

<!-- TODO update: -->

En el momento de escribir este artículo, `vue-test-utils` no viene con una utilidad para testea esto. Pero podemos usar `jest.spyOn`:

```javascript
it('mensaje de tipo cadena', () => {
  let spy = jest.spyOn(console, 'error')
  cmp = createCmp({ mensaje: 1 })
  expect(spy).toBeCalledWith(expect.stringContaining('[Vue warn]: Invalid prop'))

  spy.mockReset() // ó mockRestore() para limpiar el _mock_
})
```

Así podemos espiar la función `console.error`, y _checkear_ que se muestra un mensaje con una cadena en concreto. No es que sea lo más adecuado, ya que estamos espiando objetos globales y dependiendo de _side effects_.

Afortunadamente, hay una manera más sencilla de comprobarlo, y es usar `vm.$options`. Aquí es donde Vue guarda las opciones de un componente de manera extendida. Y por "extendida" nos referimos a que se puede definiir las propiedades de maneras diferentes:

```javascript
props: ['mensaje']

// ó

props: {
  mensaje: String
}

// ó

props: {
  mensaje: {
    type: String
  }
}
```

Al finál, aparecerán de la forma más extendida (como el último ejemplo). Así que si intentáramos obtener `cmp.vm.$option.props.mensaje` usando el primer caso, aparecerían con formato `{ type: X }` (aunque en el primer ejemplo fuera `{ type: null}`)

Con ello en mente, podemos escribir una _test suite_ para testear que la propiedad `mensaje` tiene las reglas de validación esperadas:

```javascript
describe('mensaje.test.js', () => {
  ...
  describe('Propiedades', () => {
    ...
    describe('Validación', () => {
      const mensaje = createCmp().vm.$options.props.mensaje

      it('"mensaje" es de tipo cadena', () => {
        expect(mensaje.type).toBe(String)
      })

      it('"mensaje" es "required"', () => {
        expect(mensaje.required).toBeTruthy()
      })

      it('"mensaje" tiene al menos 2 caracteres', () => {
        expect(mensaje.validator && mensaje.validator('a')).toBeFalsy()
        expect(mensaje.validator && mensaje.validator('aa')).toBeTruthy()
      })
    })
```

## Eventos _Custom_

Podemos testear dos cosas en los Eventos Custom:

 - Afirmar que tras una acción se dispara un evento.
 - Checkear que un "Event Listener" se ejecuta cuando se emite el evento.

En el caso de los ejemplos `mensajeList.vue` y `mensaje.vue`, se traduce en:

 - Afirmar que el componente `mensaje` dispara un `mensaje-clickado` cuando el mensaje se clicka.
 - Checkear en `mensajeList` que `mensaje-clickado` se produce y el _callback_ `handleMessageClick` es llamado

Vamos a ello, en `mensaje.vue` usaremos `$emit`para disparar nuestro evento:

```html
<!-- mensaje.vue -->
<template>
    <li
      style="margin-top: 10px"
      class="mensaje"
      @click="enClick">
        {{ mensaje }}
    </li>
</template>

<script>
  export default {
    name: 'mensaje',
    props: ['mensaje'],
    methods: {
      enClick() {
        this.$emit('mensaje-clickado', this.mensaje)
      }
    }
  }
</script>
```

Y en `mensajeList.vue`, recibiremos el evento en `@mensaje-clickado`:

```html
<template>
    <ul>
        <mensaje-item
          @mensaje-clickado="handleMessageClick"
          :mensaje="mensaje"
          v-for="(mensaje, index) in mensajes"
          :key="index"></mensaje-item>
    </ul>
</template>

<script>
import MensajeItem from './mensaje'

export default {
  name: 'mensajeList',
  props: ['mensajes'],
  methods: {
    handleMessageClick(mensaje) {
      console.log(mensaje)
    }
  },
  components: {
    MensajeItem
  }
}
</script>
```

Ahora vamos a escribir nuestro test unitario. Creamos un bloque `describe` anidado en `test/Message.spec.js` y preparamos el esqueleto de el _test case_ _"Afirmar que los componentes`mensaje` disparan un `mensaje-clickado` cuando un `mensaje-item` es clickado"_ que habíamos mencionado:

```javascript
...
describe('mensaje.test.js', () => {
  ...
  describe('Eventos', () => {
    beforeEach(() => {
      cmp = createCmp({ mensaje: 'Gato' })
    })

    it('llama a "enClick" cuando se clicka en un mensaje-item', () => {
      // @TODO
    })
  })
})
```

<!-- TODO: translate `handler` -->

### Testeando que el Evento Click a disparado una función

Lo primero que podemos testear es que al clickar en un mensaje, la función `enClick` se llama. Para ello podemos usar el método `trigger` de el objeto _wrapper_ y un _spy_ (espía) de Jest usando la función global `spyOn`:

```javascript
it('llama a enClick cuando se clicka un mensaje', () => {
  const spy = spyOn(cmp.vm, 'enClick')
  cmp.update() // fuerza la re-renderización, aplicando cambios en la plantilla

  const el = cmp.find('.mensaje').trigger('click')
  expect(cmp.vm.enClick).toBeCalled()
})
```

>Daros cuenta del `cmp.update()`. cuando cambiamos algo en la plantilla como `enClick` en este casp, y queremos que la plantilla pille los cambios, necesitamos usar la función `update`.

Otra cosa a tener en cuenta es que hemos utilziado la misma función `enClick`. Lo aconsejable sería _mockearla_ para sólo testear que el asociado al _click_ se ha llamado. Para ello usaremos la función Mock de Jest (`jest.fn()`, que es una función que nosotr@s controlamos):

```javascript
it('llama a enClick cuando se clicka un mensaje', () => {
  cmp.vm.enClick = jest.fn()
  cmp.update()

  const el = cmp.find('.mensaje').trigger('click')
  expect(cmp.vm.enClick).toBeCalled()
})
```

De dicha manera, sustituimos el método `enClick` completamente, al cual hemos accedido desde el _wrapper component_ que devuelve la funcion _mount()_.

Podemos simplificarlo aún más con el _helper_ `setMethods` que las herramientas oficiales nos preveen:

```javascript
it('llama a enClick cuando se clicka un mensaje', () => {
  const stub = jest.fn()
  cmp.setMethods({ enClick: stub })

  const el = cmp.find('.mensaje').trigger('click')
  expect(stub).toBeCalled()
})
```

Usar **`setMethods` es la manera recomendado**, ya que es una abstracción que nos brindan las herramientas oficiales útil en caso de que el mecanismo interno de Vue cambie.

### Testeando que el Evento _Custom_ `mensaje-clickado` se emite

Hemos testeado que el evento de click tiene un método asociado, pero no hemos comprobado que el _handler_ emite un evento `mensaje-clickado`. Podemos llamar directamente al método `enClick` y usar una función _Jest Mock_ en combinación con el método `$on` del _vm_:

```javascript
it('emite un evento "mensaje-clickado" cuando el método enClick se invoca', () => {
  const stub = jest.fn()
  cmp.vm.$on('mensaje-clickado', stub)
  cmp.vm.enClick()

  expect(stub).toBeCalledWith('Gato')
})
```

Estamos usando el `toBeCalledWith` para afirmar los parámetros que se le están pasando, haciendo el test todavía más robusto. Esta vez no necesitamos `cmp.update()`, ya que no hay cambios en la plantilla que necesitaran ser propagados.

### Testeando que el @mensaje-clickado dispara un evento

Para eventos _custom_, no usamos el método `trigger`, ya que es sólo para eventos propios del DOM. Pero podemos emitirlo nosotr@s mism@s, utilizando el `vm.$emit` del componente Mensaje. Añadiremos lo siguiente en `mensajeList.test.js`:

```javascript
it('Llama a handleMessageClick cuando @mensaje-click se produce', () => {
  const stub = jest.fn()
  cmp.setMethods({ handleMessageClick: stub })
  cmp.update()

  const el = cmp.find(mensaje).vm.$emit('mensaje-clickado', 'gato')
  expect(stub).toBeCalledWith('gato')
})
```

<!-- TODO: update `handleMessageClicked` ?? -->

## Resumiendo

Hemos visto múltiples casos para testear propiedades y eventos. `vue-test-utils`, la herramienta oficial de Vue, nos facilita la vida con ello.

Se puede encontrar la versión de código funcional usada en este capítulo en [este repo](https://github.com/alexjoverm/vue-testing-series/tree/Test-Properties-and-Custom-Events-in-Vue-js-Components-with-Jest).
