# Testear Componentes Anidados de Vue.js en Jest

Vamos a tratar de de usar _vue-test-utils_ para testear un árbol de componentes completamente renderizado.

<!-- TODO: ver enlace del libro -->

En [el primer capítulo](#chapter-1) hemos visto cómo usar _Shallow Rendering_ para testear un componente aislado, evitando que se renderizaran componentes hijos.

Pero en algunos casos, quizás necesitamos testear componentes que se conjuntan, o [moléculas](http://atomicdesign.bradfrost.com/chapter-2/#molecules) como se describe en los principios del Diseño Atómico. Hay que tener en cuenta que esto es aplicable para [Componentes Tontos](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), dado que no están ligados al estado global de la _app_. En la mayoría de casos, será más recomendable usar la técnica del _Shallow Rendering_ para componentes contenedores.

## Añadir un Componente de Mensaje

En el caso de un componente _ListaMensajes_ y los hijos _Mensaje_, a parte de añadir sus propios test unitarios, podemos testear como se comportan juntos tamién.

Vamos a crear un `components/Mensaje.vue`:

<!-- TODO: propiedad `mensaje` traducida? -->

```html
<template>
    <li class="mensaje">{{mensaje}}</li>
</template>

<script>
  export default {
    props: ['mensaje']
  }
</script>
```

Y actualizamos la `components/ListaMensajes.vue` para que lo use:

```html
<template>
    <ul>
        <Mensaje :mensaje="mensaje" v-for="mensaje in mensajes"/>
    </ul>
</template>

<script>
import Mensaje from './Mensaje'

export default {
  props: ['mensajes'],
  components: {
   Mensaje
  }
}
</script>
```

## Testeamos el componente madre _ListaMensajes_ con los hijos _Mensaje_

Para testear _ListaMensajes_ con componentes anidados, necesitamos usar `mount` en vez de `shallowMount` como vimos en el anterior ejercicio en `test/ListaMensajes.test.js`:

<!-- TODO: mensajes de los tests, traducidos?? -->

```javascript
import { mount } from 'vue-test-utils'
import ListaMensajes from '../src/components/ListaMensajes'

describe('ListaMensajes.test.js', () => {
  let cmp

  beforeEach(() => {
    cmp = mount(ListaMensajes, {
      // Ojo: las `props` se sobreescriben con ``propsData`
      propsData: {
        mensajes: ['Gato']
      }
    })
  })

  it('ha ercibido ["Gato"] como la propiedad mensajes', () => {
    expect(cmp.vm.mensajes).toEqual(['Gato'])
  })

  it('tiene la estructura html esperada', () => {
    expect(cmp.element).toMatchSnapshot()
  })
})
```

 > Por cierto, habrás visto el `beforeEach`. Es una manera elegante de preparar un componente limpio antes de cada test, lo cual es muy importante para el testing unitario. Cada comportamiento que queremos testear tiene que ser inependiente del anterior.

Los métodos `mount` y `shallowMount` usan la misma API. La diferencia es cómo se renderizan. Iremos viendo la API poco a poco en estas series

Si ejecutas `npm t` (alias de `npm test`) verás que los tests fallan porque el _Snapshot_ no coincide con la estructura de `ListaMensajes.test.js`. Para actualizarlo, ejecutaremos el comando con la opción `-u` (de `update`):

```
npm t -- -u
```

Si abrimos e inspeccionamos `test/__snapshots__/ListaMensajes.test.js.snap`, veremos que la clase `class="mensaje"` está ahí, lo que siggnifica que el ccomponente se ha actualizado.

```javascript
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`ListaMensajes.test.js tiene la estructura html esperada 1`] = `
<ul>
  <li
    class="mensaje"
  >
    Gato
  </li>
</ul>
`;
```

Tengamos presente **evitar la renderización anidada cuando puede haber efectos secundarios** ("side effects"), dado que los _life cycle hooks_ de los componentes hijos (`created`, `mounted`) serán disparados y por lo tanto puede haber llamadas HTTP u otros efectos secundarios que no queremos que se ejecuten. Para probar lo que comentamos, puedes probar a añadir en el componente `Mensaje.vue` un `console.log` en el hook `created()`:

```javascript
export default {
  props: ['mensaje'],
  created() {
    console.log('CREADO!')
  }
}
```
Cuando ejecutes los test de nuevo (con `npm t`), verás el texto `CREADO!` en el _output_ de la terminal. "Andarse con ojo!"

Se pueden encontrar [todos los ejemplos en Github](https://github.com/alexjoverm/vue-testing-series/tree/https://github.com/alexjoverm/vue-testing-series/tree/Test-fully-rendered-Vue-js-Components-in-Jest).
