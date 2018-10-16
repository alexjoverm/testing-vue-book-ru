# Testear Componentes Anidados de Vue.js en Jest

Vamos a tratar de de usar _vue-test-utils_ para testear un árbol de componentes completamente renderizado.

En [el primer capítulo](#chapter-1) hemos visto cómo usar _Shallow Rendering_ para testear un componente aislado, evitando que se renderizaran componentes hijos.

Pero en algunos casos, quizás necesitamos testear componentes que se conjuntan, o [moléculas](http://atomicdesign.bradfrost.com/chapter-2/#molecules) como se describe en los principios del Diseño Atómico. Hay que tener en cuenta que esto es aplicable para [Componentes Tontos](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), dado que no están ligados al estado global de la _app_. En la mayoría de casos, será más recomendable usar la técnica del _Shallow Rendering_ para componentes contenedores.

## Añadir un Componente de Mensaje

En el caso de un componente _MessageList_ y los hijos _Message_, a parte de añadir sus propios test unitarios, podemos testear como se comportan juntos tamién.

Vamos a crear un `components/Message.vue`:

<!-- TODO: propiedad `message` traducida? -->

```html
<template>
    <li class="message">{{message}}</li>
</template>

<script>
  export default {
    props: ['message']
  }
</script>
```

Y actualizamos la `components/MessageList.vue` para que lo use:

```html
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
```

## Testeamos el componente madre _MessageList_ con los hijos _Message_

Para testear _MessageList_ con componentes anidados, necesitamos usar `mount` en vez de `shallow` como vimos en el anterior ejercicio en `test/MessageList.test.js`:

<!-- TODO: mensajes de los tests, traducidos?? -->

```javascript
import { mount } from 'vue-test-utils'
import MessageList from '../src/components/MessageList'

describe('MessageList.test.js', () => {
  let cmp

  beforeEach(() => {
    cmp = mount(MessageList, {
      // Ojo: las `props` se sobreescriben con ``propsData`
      propsData: {
        messages: ['Gato']
      }
    })
  })

  it('has received ["Gato"] as the message property', () => {
    expect(cmp.vm.messages).toEqual(['Gato'])
  })

  it('has the expected html structure', () => {
    expect(cmp.element).toMatchSnapshot()
  })
})
```

 > Por cierto, habrás visto el `beforeEach`. Es una manera elegante de preparar un componente limpio antes de cada test, lo cual es muy importante para el testing unitario. Cada comportamiento que queremos testear tiene que ser inependiente del anterior.

Los métodos `mount` y `shallow` usan la misma API. La diferencia es cómo se renderizan. Iremos viendo la API poco a poco en estas series

Si ejecutas `npm t` (alias de `npm test`) verás que los tests fallan porque el _Snapshot_ no coincide con la estructura de `MessageList.test.js`. Para actualizarlo, ejecutaremos el comando con la opción `-u` (de `update`):

```
npm t -- -u
```

Si abrimos e inspeccionamos `test/__snapshots__/MessageList.test.js.snap`, veremos que la clase `class="message"` está ahí, lo que siggnifica que el ccomponente se ha actualizado.

```javascript
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`MessageList.test.js has the expected html structure 1`] = `
<ul>
  <li
    class="message"
  >
    Gato
  </li>
</ul>
`;
```

Tengamos presente **evitar la renderización anidada cuando puede haber efectos secundarios** ("side effects"), dado que los _life cycle hooks_ de los componentes hijos (`created`, `mounted`) serán disparados y por lo tanto puede haber llamadas HTTP u otros efectos secundarios que no queremos que se ejecuten. Para probar lo que comentamos, puedes probar a añadir en el componente `Message.vue` un `console.log` en el hook `created()`:

```javascript
export default {
  props: ['message'],
  created() {
    console.log('CREADO!')
  }
}
```
Cuando ejecutes los test de nuevo (con `npm t`), verás el texto `CREADO!` en el _output_ de la terminal. "Andarse con ojo!"

Se pueden encontrar [todos los ejemplos en Github](https://github.com/alexjoverm/vue-testing-series/tree/https://github.com/alexjoverm/vue-testing-series/tree/Test-fully-rendered-Vue-js-Components-in-Jest).
