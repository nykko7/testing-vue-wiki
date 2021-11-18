En la publicación de [Mock de $route y $router](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Mock-de-$route-y-$router) de esta Wiki, en el ejemplo final el objetivo era verificar que cierto dropdown apareciera dependiendo de una variable. Para complementar este test, queremos verificar que dentro de este dropdown se encuentren 4 opciones y posteriormente elegir una y verificar que este dropdown no este vacio. Para realizar esto tuve varios obstáculos, y en esta entrada en la wiki desenglosaré cada uno de ellos.

<h3> ¿El dropdown tiene las 4 opciones disponibles? </h3>
Primero que nada hay que revisar que el numero de opciones dadas en este dropdown sean las correctas. En este dropdown se tienen 4 opciones, las cuales se pueden observar en el componente de `Clinical Configuration`:

```html
   <v-col
    v-if="studyType === 'interventional'"
    id="blindLevel"
    cols="6"
                >
       <v-select
             id="blindLevelElement"
             v-model="blindLevelSelected"
             label="Nivel de ciego"
             required
             :rules="rules"
             item-text="text"
             item-value="value"
             :items="[
                 { text: 'Abierto', value: 'open' },
                 { text: 'Siemple ciego', value: 'single-blind' },
                 { text: 'Doble ciego', value: 'double-blind' },
                 { text: 'Triple ciego', value: 'triple-blind' },
             ]"
       >
       </v-select>
   </v-col>
```

Como se puede observar, dentro de la etiqueta `v-select` (la cual crea el dropdown) se encuentra la lista de opciones bajo el props de `:items`. Inicialmente traté de acceder a estos datos mediante el uso del método `find()` de Vue-Test-Utils (VTU), sin embargo, al encontrar el id de `"blindLevelElement`, este no me daba acceso a los props del dropdown. 

Para poder obtener estos datos se uso el comando `find()`, pero encadenando dos de estos métodos. Primero usamos uno para encontrar la columna en la que se encuentra nuestro dropdown y posteriormente encadenamos otro find para encontrar nuestro `v-select`. Con esto, ya podemos utilizar el metodo de `props()` para encontrar las opciones marcadas en ese dropdown:

```javascript
    const blindLevelLocation = wrapper.find('#blindLevel')
    const dropdown = blindLevelLocation.find('.v-select')
    expect(dropdown.props('items').length).toBe(4)
```

Con esto, nuestro primer test ya funciona.

<h3> Seleccionar una de las opciones del dropdown </h3>

Ahora queremos elegir una de las opciones y verificar que esta opción concuerde con el valor que se seleccionó en el test. Inicialmente intenté ponerle el valor con el método `setValue()` de VTU, sin embargo, este solo funciona para el caso de input mediante un cuadro de texto.

Para el dropdown es necesario saber como testear un comportamiento asíncrono. Hay dos tipos de este comportamiento, siendo en este caso actualizaciones mediante Vue. Vue procesa en lotes las actualizaciones de DOM pendientes y las aplica de forma asincrónica para evitar re-renderizaciones innecesarias. 

Esto significa que después de mutar una propiedad, para que el cambio se realice, el test debe esperar mientras Vue se actualiza. Una forma de hacer eso es usar `await` y `Vue.nextTick()`, otra forma más fácil y limpia es simplemente usar `await` y esperar el método con el que mutó el estado, como `trigger` o en nuestro caso `selectItem()`. Nuestro test entonces se mira de la siguiente forma:

```javascript
it('Blind level (only for interventional)', async () => {

    dropdown.vm.selectItem({ text: 'Abierto', value: 'open' })
    await wrapper.vm.$nextTick()

```

Ahora, solo nos falta comprobar que el valor del dropdown si sea el que seleccionamos, esto lo hacemos mediante el uso del método `internalValue`, que nos devuelve el valor interno de nuestra selección. Nuestro test completo queda entonces:

```javascript
it('Blind level (only for interventional)', async () => {

    dropdown.vm.selectItem({ text: 'Abierto', value: 'open' })
    await wrapper.vm.$nextTick()

    // Revisar que la opción seleccionada es la correcta
    expect(dropdown.vm.internalValue).toEqual('open')

```

> Nota: Los metodos de `internalValue` y `selectItem` son métodos del componente de v-select, (y al menos hasta donde yo he encontrado) estos no cuentan con documentación oficial, ya que Vuetify ya hace estos tests por su parte. En su documentación, Vuetify y Vue Test Utils "recomiendan escribir tests que afirmen la interfaz publica de tu interfaz, y tratar sus mecanismos internos como una caja negra". 

Con esto, nuestro test pasa y se puede verificar fácilmente que el dropdown no esta vacío, de la siguiente forma:

```javascript
    expect(dropdown.vm.internalValue).not.toEqual(null)
```

Nuestro test queda entonces de la siguiente forma:

```javascript
it('Blind level (only for interventional)', async () => {

    dropdown.vm.selectItem({ text: 'Abierto', value: 'open' })
    await wrapper.vm.$nextTick()

    // Revisar que la opción seleccionada es la correcta
    expect(dropdown.vm.internalValue).toEqual('open')

    // Revisar que el dropdown si haya seleccionado para proseguir con el flujo
    expect(dropdown.vm.internalValue).not.toEqual(null)

```

<h3> Referencias </h3>

- [Testing Asynchronous Behavior](https://vue-test-utils.vuejs.org/guides/testing-async-components.html)

- [find](https://vue-test-utils.vuejs.org/api/wrapper/find.html)

- [Asynchronous Behavior](https://next.vue-test-utils.vuejs.org/guide/advanced/async-suspense.html#a-simple-example-updating-with-trigger)

- [GitHub (Vuetify)](https://github.com/vuetifyjs/vuetify/blob/b2abe9fa274feeb0c5033bf12cc48276d4ac5a78/packages/vuetify/test/unit/components/VSelect/VSelect.spec.js#L28)

- [Guides: Vue Test Utils](https://vue-test-utils.vuejs.org/guides/#getting-started)

- [Mock Functions](https://jestjs.io/docs/mock-functions)