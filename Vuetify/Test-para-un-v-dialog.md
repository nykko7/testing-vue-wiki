Al hacer nuestro `wrapper`, este no podrá encontrar los elementos dentro de un `v-dialog` a menos que se active. Es decir, para nosotros poder testear funcionalidades dentro de estos componentes es necesario activar este componente.

> UPDATE (04/06/21): Después de más investigación, se encontró una propiedad para `<v-dialog>` que permite que este este renderizado en el DOM desde que se monta el componente. Para ver esto, revisa el apéndice.  

<h3> Como abrir un v-alert </h3>

En el caso de este proyecto, para abrir esta alerta, se necesita presionar el botón de `"Nuevo Grupo"`, en donde se abre una nueva ventana para poder crear el nuevo grupo: 

![image](uploads/d6d2c83161a9a9dffc89bf5fe7fe1d77/image.png)

Su código siendo:

```html
       <v-btn class="addGroup" color="primary" @click="dialogAddGroup = true">
         Agregar
      </v-btn>

```

Entonces, lo primero que tenemos que hacer es buscar en nuestro `wrapper `el botón de `"Nuevo Grupo"`. Esto lo hacemos mediante el método `find()` y buscando el nombre de la clase del botón:

```javascript
     const addGroupBtn = wrapper.find('.addGroup')
```

Una vez hecho esto, vamos a activar este botón con el metodo de `trigger()`. Este método hace que se dispare el evento que queremos, en este caso, el click de nuestro botón. Después de esto, como es un evento asíncrono, esperamos a que nuestro DOM se actualice para ahora si realizar nuestras pruebas:

```javascript
     const addGroupBtn = wrapper.find('.addGroup')
     addGroupBtn.trigger('click')
     await wrapper.vm.$nextTick()
```

Para comprobar que si estamos dentro de este componente, podemos buscar alguno de los elementos con find() y empezar a nuestras pruebas con ellos. Por ejemplo, si queremos probar el input de texto:

```javascript
     const nombreGrupo = wrapper.find('#nombreGrupo')
     nombreGrupo.setValue("Grupo C")
     
     expect(nombreGrupo.element.value).toBe("Grupo C")
```

Con esto ya sabemos que si nos encontramos dentro del componente a probar y podemos realizar test mas complicados, como de flujo o espias.

Una cosa que es importante notar, es que al correr los tests aparece el siguiente warning:

![image](uploads/5fa1c54a5e6e722b4cfd0c8b2d7665a1/image.png)

Esto se debe a que todos los elementos de Vuetify deben estar dentro de las etiquetas <v-app>. Para simular esto en nuestro test y que el `<v-dialog>` se renderice correctamente, colocamos la siguiente linea al inicio de nuestro archivo donde tenemos nuestros tests:

```javascript
document.body.setAttribute('data-app', true)
```

Con esto el warning desaparece. 

<h3> Apéndice </h3>

Otra forma de poder utilizar y encontrar los elementos dentro de un `<v-dialog>` es mediante la propiedad `eager`. Esta propiedad lo que hace es renderizar el componente en el DOM cuando se monte, sin la necesidad de interactuar con el para que aparezca (como se hacia anteriormente). En nuestro componente, esto se ve de la siguiente forma:

```javascript
<v-dialog v-model="dialogAddGroup" max-width="500px" eager>
...
<v-dialog>
```

Asi en nuestro test podemos pasar a buscar directamente lo que queremos testear dentro de nuestro componente.

> NOTA: A pesar de que este método es distinto al anterior, la linea `document.body.setAttribute('data-app', true)` aun es necesaria para que no aparezca el warning señalado mas arriba. 

<h3> Referencias </h3>

- [How to test UX of v-dialog (Vuetify) components which is not appears in wrapper?](https://forum.vuejs.org/t/how-to-test-ux-of-v-dialog-vuetify-components-which-is-not-appears-in-wrapper/78114)

- [trigger](https://vue-test-utils.vuejs.org/api/wrapper/trigger.html)

- [v-dialog API](https://vuetifyjs.com/en/api/v-dialog/#props)

- [Vuetify, data-app=“true” and problems rendering <v-dialog> in unit tests](https://forum.vuejs.org/t/vuetify-data-app-true-and-problems-rendering-v-dialog-in-unit-tests/27495)