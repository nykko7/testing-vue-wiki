En este proyecto, al validar la carga de datos en la configuración del estudio clínico, aparece un botón de continuar para seguir con el flujo de configuración. Si queremos testear esto, necesitamos identificar que condiciones se necesitan para que este botón aparezca, y sobre todo, encontrar estos botones en nuestro wrapper. 

> UPDATE (04/06/21): Después de mas investigación, se pudo encontrar los botones utilizando la etiqueta de html `id`. Para leer como se hizo esto en vez de utilizar el método `findAll`, revisa el apendice. 

<h3> Encontrando los botones </h3>

Primeramente, debemos de ubicar nuestros botones para poder testear si este botón esta visible o no. En este caso, los botones se encuentran dentro del siguiente `v-row` con un id de `id="progBtn"`:

```html
      <v-row id="progBtn" class="px-2">
        <v-spacer></v-spacer>
        <v-btn color="primary" class="mx-1" @click="saveData"> Guardar </v-btn>
        <v-btn
          v-if="validStep"
          color="primary"
          class="mx-1"
          :disabled="!validStep"
          type="submit"
          @click="nextStep"
        >
          Continuar
        </v-btn>
      </v-row>
```

Dentro de aqui tenemos dos botones, uno llamado `Guardar` en donde se guardan los datos al hacer click en el, y otro llamado `Continuar`. El botón de `Continuar` tiene una condicional (`v-if`). Esto significa que este botón no se renderizará a menos que el método `validStep`, retorne verdadero. 

Para seleccionar nuestro botón podemos utilizar el metódo de `find()` de Vue Test Utils (VTU), en donde le daremos como argumento la clase que tiene el botón, siendo en este caso `mx-1`. Sin embargo, esto genera un problema, ya que ambos botones tienen esta clase, asi que esto solo funciona para **botones con distintas clases**.

Ahora bien, sabemos que nuestro `<v-row>` contiene a estos dos botones, entonces podemos utilizar el método `findAll()` lo cual nos devuelve un _WrapperArray_, que contiene a todos los wrappers que contenga el método `findAll`. En este caso, queremos todos los wrappers que contengan un botón, entonces en nuestro test, tendríamos lo siguiente:

```javascript
    const continueButton = wrapper.find('#progBtn').findAll('.v-btn')
```

<h3> Comprobando que nuestro botón no esta </h3>

Ahora ya que tenemos nuestros botones almacenados podemos acceder a ellos con el método `at()` de WrapperArray. Sin embargo, como el botón que queremos verificar está bajo una condición para renderizarse, si esta condición no se cumple, el WrapperArray solo contendrá los botones que si se renderizaron (siendo en este caso solo 1). ¿Entonces como comprobamos que ese botón no esta ahí? WrapperArray tiene un atributo llamado `length`, el cual nos da la cantidad de Wrappers que hay en el WrapperArray. Entonces, si nuestro botón no debe de estar renderizado, el tamaño de nuestro arreglo debe de ser uno. Nuestro test queda entonces:

```javascript
  it('Continue button is not present if a variable is missing', () => {
    const wrapper = mount(ClinicalConfiguration, {
      localVue,
      store,
      vuetify,

      mocks: {
        $route,
      },
    })

    // Como ninguna variable ha sido llenada, esperamos que el botón de continuar
    // no aparezca

    // Dentro del v-row donde se encuentran el botón de "Continuar" y de "Guardar"
    // buscamos todos los botones presentes
    const continueButton = wrapper.find('#progBtn').findAll('.v-btn')

    // Si el botón de continuar no esta presente, entonces la constante de 'continueButton'
    // solo debería de tener un botón
    expect(continueButton.length).toBe(1)
  })

```

Con esto ya comprobamos que si no se cumple la condición de `v-if`, no se puede seguir con el progreso de la configuración del estudio.

<h3> Comprobando que nuestro botón si esta </h3>

Ahora queremos comprobar que si se cumple la condición de nuestro `v-if`, el botón si se aparecerá y nos permitirá seguir con el flujo de configuración. La condición es un método que se llama en la sección de `computed` del componente:

```javascript
    validStep() {
      const { configuration } = this.savedProject
      if (
        configuration &&
        configuration.variables.length > 0 &&
        configuration.description &&
        configuration.conditionDesease &&
        configuration.phase &&
        configuration.interventionTreatment &&
        configuration.phase &&
        configuration.studyType &&
        configuration.numberOfParticipants > 0
      ) {
        if (configuration.studyType === 'interventional') {
          return configuration.blindLevelSelected
        } else {
          return true
        }
      } else {
        return false
      }
    },
```

Solamente si todos esos datos están llenos, el botón aparecerá. Una forma de hacer que este método regrese un `true` es dandole como argumento, a la hora de montar nuestro componente, este método y que regrese `true`:

```javascript
    const wrapper = mount(ClinicalConfiguration, {
      localVue,
      store,
      vuetify,

      mocks: {
        $route,
      },

      computed: {
        validStep() {
          return true
        }
      }
    })

```

Sin embargo, esto no comprueba que los datos que estan en la store de Vuex si fueron llenados correctamente, solo se fuerza al método a que regrese true. Entonces, para poder hacer el test de una forma mas exacto, en nuestra store de Vuex haremos un mock de una configuración completa:

```javascript
    store.commit('projects/SET_PROJECT', {
      id: '1',
      title: 'Proyecto A',
      publish: undefined,
      configuration: {
        blindLevelSelected: null,
        conditionDesease: 'Enfermedad X',
        description: 'Descripción',
        interventionTreatment: 'Intervention X',
        numberOfParticipants: 10,
        phase: 'phase 1',
        studyType: 'observational',
        variables: [{ title: 'Variable A', type: 'Métrica primaria' }],
      },
    })
```

Con esto, la condición se cumple y el botón de `Continuar` debería de aparecer. Para comprobar esto, haremos la misma prueba que el test anterior, solo que en vez de esperar un solo botón, esperamos dos. Nuestro test queda entonces:

```javascript
    const store = new Vuex.Store({
      modules: {
        projects: {
          namespaced: true,
          state,
          mutations,
          getters,
        },
      },
    })

    store.commit('projects/SET_PROJECT', {
      id: '1',
      title: 'Proyecto A',
      publish: undefined,
      configuration: {
        blindLevelSelected: null,
        conditionDesease: 'Enfermedad X',
        description: 'Descripción',
        interventionTreatment: 'Intervention X',
        numberOfParticipants: 10,
        phase: 'phase 1',
        studyType: 'observational',
        variables: [{ title: 'Variable A', type: 'Métrica primaria' }],
      },
    })

    const wrapper = mount(ClinicalConfiguration, {
      localVue,
      store,
      vuetify,

      mocks: {
        $route,
      },
    })

    // Como todas las variables han sido llenadas, esperamos que el botón de continuar
    // aparezca

    // Dentro del v-row donde se encuentran el botón de "Continuar" y de "Guardar"
    // buscamos todos los botones presentes
    const continueButton = wrapper.find('#progBtn').findAll('.v-btn')

    // Si el botón de continuar esta presente, entonces la constante de 'continueButton'
    // solo debería de tener dos botones
    expect(continueButton.length).toBe(2)
  })
```

<h3> Apéndice </h3>

Una forma más fácil de encontrar elementos es utilizando la etiqueta `id` que nos proporciona HTML. Esta etiqueta no es propiedad de Vuetify en si, si no de HTML, sin embargo, esta puede ser utilizada de todas formas para identificar el componente en nuestro DOM. Para ponerle la etiqueta, lo hacemos de la siguiente forma:

```html
        <v-btn
          v-if="btnContinue || validStep"
          id="continueBtn"
          color="primary"
          class="mx-1"
          :disabled="!validStep"
          type="submit"
          @click="nextStep"
        >
          Continuar
        </v-btn>
```

Ahora lo unico que tenemos que hacer es cerciorarnos de que este botón este visible. Esto lo hacemos con el método de VTU llamado `isVisible()`:

```javascript
const continueButton = wrapper.find('#continueBtn')
expect(continueButton.isVisible()).toBe(true)

```

Y si lo que queremos hacer es revisar que el botón no esta presente, podemos utilizar el método de `exists()`:

```javascript
    const continueButton = wrapper.find('#continueBtn')
    expect(continueButton.exists()).toBe(false)

```

<h3> Referencias </h3>

- [WrapperArray](https://vue-test-utils.vuejs.org/api/wrapper-array/)

- [v-btn API](https://vuetifyjs.com/en/api/v-btn/)

- [findAll](https://vue-test-utils.vuejs.org/api/wrapper/#findall)

- [isVisible](https://vue-test-utils.vuejs.org/api/wrapper/#isvisible)

- [exists](https://vue-test-utils.vuejs.org/api/wrapper/#exists)