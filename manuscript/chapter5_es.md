# Testear _Computed Properties_ y _Watchers_ en Componentes Vue.js con Jest {#chapter-5}

Aprende cómo testear la reactividad de _Computed Properties_ y _Watchers_ en Vue.js.

_Computed properties_ y _watchers_ son partes reactivas de la lógica de componentes Vue.js. Ambas sirven propósitos completamente distintos, una es síncrona y la otra asíncrona, lo que las hace un poco diferentes.

En este capítulo, vamos a repasar su capacidad de testeo y los casos que se nos pueden presentar.

## _Computed Properties_

Las _computed properties_ son funciones reactivas simplesque devuelven una transformacón de un dato. Se comportan como los métodos nativos de una clase o un objeto `get/set`:

```javascript
class X {
  ...

  get nombreCompleto() {
    return `${this.nombre} ${this.apellido}`
  }

  set nombreCompleto() {
    // ...
  }
}
```

De hecho, cuando se usan componentes basados en clases de Vue, como explico en mi curso de Egghead ["Use TypeScript to Develop Vue.js Web Applications"](https://egghead.io/courses/use-typescript-to-develop-vue-js-web-applications), se usan de dicha manera. En caso de suar objetos en la definición de un componente, sería:

```javascript
export default {
  ...
  computed: {
    nombreCompleto() { // ES6+ alias de `nombreCompleto: function() {`
      return `${this.nombre} ${this.apellido}`
    }
  }
}
```

Y podemos añadir un `set` de la siguiente manera:

```javascript
computed: {
    nombreCompleto: {
      get() {
        return `${this.nombre} ${this.apellido}`
      },
      set() {
        ...
      }
    }
  }
```

### Testeando _Computed Properties_

Testeando una _computed property_ es tan complicado como testear un método simple. Muchas veces, nisiquiera testearemos una _computed property_ exclusivamente, sino que irá intrínseco con otros tests. Pero lo recomendable sería testear esa transformación, ya sea de combinar datos o limpiar un `input`; para demostrar que todo funciona como esperamos. Así que vamos a verlo!

Primero, vamos a crear un componente `Form.vue`:

```html
<template>
  <div>
    <form action="">
      <input type="text" v-model="inputValue">
      <span class="reversed">{{ reversedInput }}</span>
    </form>
  </div>
</template>

<script>
export default {
  props: ['reversed'],
  data: () => ({
    inputValue: ''
  }),
  computed: {
    reversedInput() {
      return this.reversed
        ? this.inputValue.split("").reverse().join("")
        : this.inputValue
    }
  }
}
</script>
```

Mostrará un `input`, junto con un reflejo de la cadena que introduzcamos. Es un ejemplo simplón pero suficiente para que nosotr@s lo testeemos.

Ahora lo añadimos a nuestro árbol de componentes en `App.vue`, lo pondremos después del componente `MessageList`. Y hay que acordarse de importarlo e incluirlo en  la propiedad `components` de nuestra isntancia. Luego, creamos un `test/Form.test.js` con el esqueleto habitual:

```javascript
import { shallow } from 'vue-test-utils'
import Form from '../src/components/Form'

describe('Form.test.js', () => {
  let cmp

  beforeEach(() => {
    cmp = shallow(Form)
  })
})
```

Y ahora crearemos nuestra _test suite_ con dos _test cases_:

```javascript
describe('Propiedades', () => {
  it('devuelve la cadena que le pasemos si la propiedad `reversed` no es `true`', () => {
    cmp.vm.inputValue = 'Women in tech'
    expect(cmp.vm.reversedInput).toBe('Women in tech')
  })

  it('devuelve la cadena invertida si la propiedad `reversed` es `true`', () => {
    cmp.vm.inputValue = 'Women in tech'
    cmp.setProps({ reversed: true })
    expect(cmp.vm.reversedInput).toBe('hcet ni nemoW')
  })
})s
```

Recordemos que podemos acceder a la instancia del componente con `cmp.vm`, de manera que podemos accdeer a su estado local, _computed properties_ y métodos asociados. Con lo cual, para testearlo, cambiamos el valor de `inputValue` y nos aseguramos de que no se produce transformación si la `reversed` es `false`.

Para el segundo caso, debería ser casi idéntico, con la diferencia de que seteamos la propiedad `reversed` a `true`. Accederíamos a través de `cmp.vm...` para cambiarlo, pero _vue-test-utils_ nos da un _helper_ que lo facilita: `setProps({ property: value, ... })`.

Y ya está! Dependiendo de si la _computed property_ tiene más casos (o "ramas", como llaman librarías de _coverage_ como _Istanbul_), necesitaremos más _test cases_...

## _Watchers_

Sincerándome con vosotr@s, de momento no he necesitado usar _watchers_ para resolver reactivamente algo que no pudiera resolver una _computed property_. También he presenciado su mal uso, que provoca dificulta seguir el código a través de componentes, mezclándo cosas. Con lo que antes de usar uno, pregúntate si realmente lo necesitas

Como se puee ver en los [docs de Vue.js](https://vuejs.org/v2/guide/computed.html#Watchers), los _watchers_ se suelen utilizar para reaccionar a cambios en los datos y realizar operaciones asíncronas, como una petición AJAX.

### Testeando _Watchers_

Vamos a suponer que queremos realizar un cambio cuando el `inputValue` de nuestro estado cambia. Podría ser una petición AJAX, pero dado que sería más complicado y que vamos a verlo en la siguiente lección; vamos a usar un `console.log`. Añadimos una propeidad `watch` a las opciones de nuestro componente `Form.vue`:

```javascript
watch: {
  inputValue(newVal, oldVal) {
    if(newVal.trim().length && newVal !== oldVal) {
      console.log(newVal)
    }
  }
}
```

Importante que la propiedad `inputValue` del watch, que es una función se llame igual que la variable de nuestro estado. Por convención, Vue buscará en `props` y el estado (`data`) usando este nombre, y como `inputValue` está en `data`, se añadirá ahí el `watcher`.

Ved cómo la función del `watch` toma como primer parámetro el nuevo valor y como segundo el antiguo. En este caso, hemos decidido _loggear_ sólo cuando no está vacío y los dos valores son diferentes. Para lo cual, normalmente escribiríamos dos tests, dependiendo del tiempo que dispongamos y de lo crítico que sea esta parte del código.

Qué deberíamos testear en esta función? Bueno, es algo que vamos a discutir también en la siguiente lección cuando hablemos de métodos de testing, pero supongamos que queremos saber que se llama al método `console.log` cuando debería. Vamos a añadir lo básico de nuestro `Form.test.js`:

```javascript
describe('Form.test.js', () => {
  let cmp
  ...

  describe('Watchers - inputValue', () => {
    let spy

    beforeAll(() => {
      spy = jest.spyOn(console, 'log')
    })

    afterEach(() => {
      spy.mockClear()
    })

    it('no se llama si se pasa valor vacío (sin espacios delante)', () => {
    })

    it('no se llama si los valores son igualesn', () => {
    })

    it('se llama con un nuevo valor en el resto de casos', () => {
    })
  })
})
```

Estamos usando un espía (_spy)_ en el método `console.log`. Lo inicializamos al comienzo de cada test y lo limpiamos después de cada uno, de manera que parten de un _spy_ limpio y nos aseguramos de cumplir que los test unitarios deberían ser independientes.

Para testear una función _watch_, tenemos que cambiar el valor de aquello sobre lo que se está aplicando el _watcher_, en este caso es el `inputValue` de nuestro `data()`. Pero ocurre algo curioso..., vamos a empezar por el último test:

```javascript
it('se llama con un nuevo valor en el resto de casos', () => {
  cmp.vm.inputValue = 'vue vixens'
  expect(spy).toBeCalled()
})
```

Al cambiar el `inputValue`, el espía del `console.log` debería llamrse, ¿verdad? ¡Pues no! ¿¿¿_WTF_??? Pero hay una explciación: al contrario de las _computed properties_, los _watchers_ son **aplazados al siguiente ciclo de actualización** que Vue usa para ver los cambios. Así que, básicamente, `console.log` sí se está llamando, pero para el momento en el que nuestros tests ya se han ejecutado.

Para solucionarlo, usaremos la función [`vm.$nextTick`](https://vuejs.org/v2/api/#vm-nextTick) para aplazar la llamada al siguiente ciclo de actualización. Pero si hacemos:

```javascript
it('se llama con un nuevo valor en el resto de casos', () => {
  cmp.vm.inputValue = 'vue vixens'
  cmp.vm.$nextTick(() => {
    expect(spy).toBeCalled()
  })
})
```

<!-- TODO: revisar -->

Fallará, dado que el `expect` no se llama para cuando ha terminado. Esto pasa porque ahora la ejecución es asíncrona: ocurre en el _callback_ de `$nextTick` (que, de hecho, devuelve una Promesa). ¿Cómo podemos esperar al `expect`?

Jest nos da el parámetro **`next`** que podemos usar como _callback_ de los `it`, de manera que si está presente, los _test cases_ no terminan hasta que se llama a `next()` y si no está, se ejecutan de manera síncrona. Así que para que sea correcto:

```javascript
// Fijarse en el parámetro next:
it('se llama con un nuevo valor en el resto de casos', next => {
  cmp.vm.inputValue = 'vue vixens'
  cmp.vm.$nextTick(() => {
    expect(spy).toBeCalled()
    next()
  })
})
```

Podemos usar la misma técnica para los otros dos, con la diferencia de que el _spy_ **no** debería llamarse:

```javascript
it('no se llama si se pasa valor vacío (sin espacios delante)', next => {
  cmp.vm.inputValue = '   '
  cmp.vm.$nextTick(() => {
    expect(spy).not.toBeCalled()
    next()
  })
})

it('no se llama si los valores son iguales', next => {
  cmp.vm.inputValue = 'Sarah Drasner is awesome'

  cmp.vm.$nextTick(() => {
    spy.mockClear()
    cmp.vm.inputValue = 'Sarah Drasner is awesome'

    cmp.vm.$nextTick(() => {
      expect(spy).not.toBeCalled()
      next()
    })
  })
})
```

Aquí, el segundo parece un poco más complejo. El estado interno por defecto está vacío, así que necesitamos cambiarlo, esperar al `nextTick`, limpiar el mock para _restear_ el contador y cambiarlo de nuevo.

Para simplificarlo también podemos recrear el componente desde su estado inicial, sobreescribiendo la propiedad `data`. Recordemos que podemos sobreescribir cualquier opción del componente usando el segundo parámentro de las funciones `mount` ó `shallow`:

```javascript
it('no se llama si los valores son iguales', next => {
  cmp = shallow(Form, { data: ({ inputValue: 'Sarah Drasner is awesome' }) })
  cmp.vm.inputValue = 'Sarah Drasner is awesome'

  cmp.vm.$nextTick(() => {
    expect(spy).not.toBeCalled()
    next()
  })
})
```

## Conclusión

Hemos aprendido en este capítulo cómo testear parte de la lógica de los componentes: _computed properties_ y _watchers_. Hemos visto diferentes escenarios posibles para testearlos. Y quizás hemos aprendido una función interna de Vue que es el `nextTick` para diferir ciclos.

Se puede encontrar el código de estos ejemplos [en el repo](https://github.com/alexjoverm/vue-testing-series/tree/Test-State-Computed-Properties-and-Methods-in-Vue-js-Components-with-Jest).
