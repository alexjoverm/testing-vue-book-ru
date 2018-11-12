# Escribe tu Primer Test Unitario de un Componente de Vue.js con Jest {#chapter-1}

Vamos a aprender cómo escribir tests unitarios con las herramientas oficiales de VueJS y el framework Jest.

[vue-test-utils](https://github.com/vuejs/vue-test-utils), es la librería oficial de testing de VueJS y está basada en su predecesora [avoriaz](https://github.com/eddyerburgh/avoriaz). [@EddYerburgh](https://twitter.com/EddYerburgh) hizo un excepcional trabajo sentando las bases del testing en Vue con dicha librería.

[Jest](https://facebook.github.io/jest), por otro lado es un framework de testing desarrollado en Facebook, que hace que el testing unitario sea súper asequible, con funcionalidades como:

 - Casi _0-config_, por defecto.
 - Un "Modo interactivo".
 - Ejecutar tests en paralelo.
 - _Spies_, _stubs_ y _mocks_.
 - Coverage de código
 - Introduce el concepto de _Snapshot testing_.
 - Y aporta utilidades de _Module mocking_.

Probablemente ya hayas escrito tests sin estas herramientas, usando "sólamente" Karma, Mocha, Chai, Sinon, Istanbul..., pero veréis que con Jest se facilita en gran medida el proceso 😉.

## _Set up_ de un proyecto de ejemplo

Para el _scaffolding_ vamos a utilizar [`vue-cli`](https://github.com/vuejs/vue-cli) y por defecto contestaremos NO a todas las pregutnas que nos aparecen. Llamaremos al proyecto "vue-test":

```bash
npm install -g vue-cli
vue init webpack vue-test
cd vue-test
```

Lo siguiente será instalar algunas dependencias

```bash
# Install dependencies
npm i -D jest vue-jest babel-jest
```

<!-- TODO: jest-vue-preprocessor o vue-jest ?? -->

[`jest-vue-preprocessor`](https://github.com/vire/jest-vue-preprocessor) is needed for making jest understand `.vue` files, and [`babel-jest`](https://github.com/babel/babel-jest) for the integration with Babel.

Y además:

```bash
npm i -D vue-test-utils
```

<!-- TODO: archivo config jest -->

Vamos a añadir la siguiente configuración en nuestro `package.json`: (también puede ir en su propio archivo de configuración)

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
    ".*\\.(vue)$": "<rootDir>/node_modules/vue-jest"
  }
}
...
```

`moduleFileExtensions` informa a Jest de qué extensiones tiene que buscar, y `transform` qué preprocesador usar para cada extensión.

Y por fin, añadimos un script `test` al `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    ...
  },
  ...
}
```

## Testing a Component

Para los ejercicios, usaremos _Single File Components_, ya que no he probado qué tal funciona con archivos `html`, `css` y `js` sueltos, así que asumimos que estarás usando SFC también.

Crearemos un `ListaMensajes.vue` component under `src/components`:

```html
<template>
    <ul>
        <li v-for="mensaje in mensajes">
            {{ mensaje }}
        </li>
    </ul>
</template>

<script>
export default {
  name: 'list',
  props: ['mensajes']
}
</script>
```

Y lo añadimos al `App.vue` para que se utilice, de la siguiente manera:

```html
<template>
  <div id="app">
    <ListaMensajes :mensajes="mensajes"/>
  </div>
</template>

<script>
import ListaMensajes from './components/ListaMensajes'

export default {
  name: 'app',
  data: () => ({ mensajes: ['Hola Don Pepito', 'Hola Don José'] }),
  components: {
    ListaMensajes
  }
}
</script>
```

Ya tenemos, entonces, dos componentes que podríamos testear. Crearemos la carpeta `test` en el directorio raíz del proyecto, y un `App.test.js`:

```javascript
import Vue from 'vue'
import App from '../src/App'

describe('App.test.js', () => {
  let cmp, vm

  beforeEach(() => {
    cmp = Vue.extend(App) // Crea una copia del componente original
    vm = new cmp({
      data: { // Reemplazamos el estado local con datos _fake_
        mensajes: ['Gatito']
      }
    }).$mount() // Lo instanciamos y montamos
  })

  it('mensajes es igual a ["Gatito"]', () => {
    expect(vm.mensajes).toEqual(['Gatito'])
  })
})
```

Ahora, si ejecutamos `npm test` (o `npm t` como atajo), los tests deberían ejecutarse y pasar. Como vamos a estar modificando los tests, será mejor que lo ejecutemos en **watch mode**:

```shell
npm t -- --watch
```

### El problema de los componentes anidados:

El test anterior era demasiado baladí. Vamos a comprobar que el _output_ es también el esperado. Para ello utilizaremos la maravillosa funcionalidad de Jest que son los Snapshots. Generarán una foto o captura ("snapshot") del output y la comparará con ulteriores ejecuciones. Para ello, añadiremos después del `it()` anterior en `App.test.js`:

```javascript
it('tiene la estructura html esperada', () => {
  expect(vm.$el).toMatchSnapshot()
})
```

Ésto creará un archivo `test/__snapshots__/App.test.js.snap`. Si inspeccionamos su contenido:

```javascript
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`App.test.js tiene la estructura html esperada 1`] = `
<div
  id="app"
>
  <ul>
    <li>
      Gatito
    </li>
  </ul>
</div>
`;
```

Por si no te habías percatado, tenemos un problema aquí: el componente `ListaMensajes` se ha renderizado también. Y los **test unitarios tienen que ser unidades independientes**, con lo que en `App.test.js` intentaremos testear el componente `App` y no involucrarnos con nada más.

Si no seguimos esta norma, pronto nos econtraríamos con problemas. Por ejemplo si nuestro componente hijo (`ListaMensajes` en este caso) llevara a cabo _side effects_ en el _hook_ de `created()`, como hacer una petición `fetch()`, o a una acción de Vuex o que cambiara el estado lcoal... Eso es algo que recomendaría evitar.

Por suerte, la funcionalidad del **Shallow Rendering** nos ayuda con este problema.

### Qué es _Shallow Rendering_?

[Shallow Rendering](http://airbnb.io/enzyme/docs/api/shallow.html) ("renderizado superficial") es una técnica que nos asegura que nuestro componente se renderizará sin hijos. Lo cuá es útil para:

 - Testear sólo el componente que queremos testear (que es lo que viene siendo el _testing_ unitario).
 - Evitar _side effects_ que los componentes hijos pudieran causar, como llamadas HTTP, acciones del _store_...

## Testeando con @vue/test-utils

<!-- TODO: original, cambiar @vue/test-utils -->

`@vue/test-utils` nos brinda la funcionalidad del _Shallow Rendering_. Podríamos re-escribir el test anterior de la siguiente manera:

```javascript
import { shallowMount } from '@vue/test-utils'
import App from '../src/App'

describe('App.test.js', () => {
  let cmp

  beforeEach(() => {
    cmp = shallowMount(App, { // Crea una instancia superficial del componente
      data: {
        mensajes: ['Gatito']
      }
    })
  })

  it('mensajes es igual a ["Gatito"]', () => {
    // En cmp.vm, podemos acceder a los métodos de la instancia componente
    expect(cmp.vm.mensajes).toEqual(['Gatito'])
  })

  it('tiene la estructura html esperada', () => {
    expect(cmp.element).toMatchSnapshot()
  })
})
```

A continuación, si todavía estamos en _watchin mode_:

```javascript
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`App.test.js tiene la estructura html esperada 1`] = `
<div
  id="app"
>
  <!--  -->
</div>
`;
```

Ves? AHora ningún hijo se renderiza y hemos testeado el componente `App` de manera **totalmente aislada** del árbol de componentes. De la misma manera, si hubiéramos añadido algún `created()` o algún otro hook en los componentes hijos, no se habrían llamado tampoco 😉.


<!-- TODO: link a nuevo paquete y traducción de Stub?? -->
Si se te despierta la curiosidad sobre **cómo el _shallow render_ se implementa**, puedes ver el [código fuente](https://github.com/vuejs/vue-test-utils/blob/dev/packages/shared/stub-components.js) and you'll see that basically is stubbing the `components` key, the `render` method and the lifecycle hooks.

Asimismo, puedes implementar `ListaMensajes.test.js` como sigue:

```javascript
import { mount } from '@vue/test-utils'
import ListaMensajes from '../src/components/ListaMensajes'

describe('ListaMensajes.test.js', () => {
  let cmp

  beforeEach(() => {
    cmp = mount(ListaMensajes, {
      // Ojo: las `props` se sobreescriben con ``propsData`
      propsData: {
        mensajes: ['Gatito']
      }
    })
  })

  it('ha recibido ["Gatito"] como la propiedad mensajes', () => {
    expect(cmp.vm.mensajes).toEqual(['Gatito'])
  })

  it('tiene la estructura html esperada', () => {
    expect(cmp.element).toMatchSnapshot()
  })
})
```

Find the [full example on Github](https://github.com/alexjoverm/vue-testing-series/tree/lesson-1).