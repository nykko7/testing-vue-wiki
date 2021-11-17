Si tienes configurado el archivo de `test coverage` en tu aplicación, lo mas probable es que has notado que hay varios archivos con *una* sola linea sin cubrir. Estos archivos usualmente se ven de la siguiente forma:

```html
<template>
  <v-container fill-height>
    <v-row>
      <v-col>
        <Login />
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import Login from '~/components/Login.vue'

export default {
  layout: 'unlogged',
  componentns: {
    Login,
  },
}
</script>
```

La linea que se indicaria que no esta testeada, es la linea que tiene la declaración de importar: 

```javascript
import Login from '~/components/Login.vue'
```

Esto solamente importa el componente para mostrarlo en pantalla a la hora de renderizar nuestro componente. Como podemos nosotros testear que esta importación se este realizando correctamente? Para esto, utilizaremos primeramente el método `toBeTruthy` de Jest, para verificar que este si exista a la hora de ser importado: 

```javascript
  it('Check that Login is called correctly', () => {
    expect(Login).toBeTruthy()
  })
```

Por ultimo, podemos utilizar el metodo `Object.keys()`, para verificar que nuestro componente tiene propiedades (como se supone que debe de tener):

```javascript
  it('Check that Login is called correctly', () => {
    expect(Login).toBeTruthy()
    expect(Object.keys(Login).length > 0).toBe(true)
  })
```

Este tipo de tests normalmente se les refiere como `sanity tests`, y normalmente se utilizan para comprobar mas bien si Jest esta funcionando de la forma correcta. En papel, estos tests no son de mucha ayuda, y generalmente solamente se utilizan si se quiere tener un cobertura de código del 100%. 

<h3> Referencias </h3>

- [How to test imports/exports with Jest?](https://stackoverflow.com/questions/54207884/how-to-test-imports-exports-with-jest/54207975)

- [Object.keys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)