Al tratar de testear el componente "ClinicalConfiguration.vue", tuve algunos problemas creando la configuración correcta debido al método `$route`, que se utilizaba en ese componente.

Un objeto tipo route representa el estado de la ruta actualmente activa. Contiene la información de la URL y la información que concuerda a esa URL. Este objeto puede ser visto en distintos lugares, pero en el caso de este proyecto, se encuentra en la siguiente línea (entre otras):

```javascript
this.$store.getters['projects/getById'](this.$route.params.id)
```

Esta línea obtiene la URL para la configuración del proyecto en base al ID generado por la aplicación para identificar al proyecto. Entonces, al no darle un valor valido de ID el test no funcionaba.

Entonces, para poder testear correctamente este componente, era necesario mockear esta instancia de Vue. Para esto, primero que nada, con Vuex se llenaron los datos necesarios para inicializar el proyecto, esto mediante el uso del método store.commit():

```javascript
store.commit('projects/SET_PROJECT', {
  id: '1',
  title: 'Proyecto A',
  publish: undefined,
  configuration: undefined,
})
```

Una vez hecho esto, se definió una constante donde se almacena la ruta en la cual se renderiza el componente para la configuración del estudio clínico. 

```javascript
const $route = {
  params: {
    id: '1',
  },
}
```

Este código se colocó antes de inicializar todos los tests, ya que todos los tests realizados para este componente necesitan saber de esta ruta para poder continuar con el proceso de configuración.

Por último, le pasamos como argumentos a nuesto wrapper el mock de `$route`:

```javascript
    mocks: {
       $route,
    },
```

De esta forma, ya se puede acceder al componente sin problemas.

A continuación se presenta el test completo que se utilizó para verificar que el dropdowon de "Nivel de Ciego" solo aparezca para el caso en que el tipo de estudio sea "Intervencional": 

```javascript
  it('Observational study', () => {
    wrapper = mount(ClinicalConfiguration, {
      localVue,
      store,
      vuetify,

      mocks: {
        $route,
      },

      data() {
        return {
          studyType: 'interventional',
        }
      },
    })

    expect(wrapper.find('#blindLevel').isVisible()).toBe(true)
  })
```

<h3> Referencias </h3>

- [Using with Vue Router](https://vue-test-utils.vuejs.org/guides/using-with-vue-router.html)

- [Dynamic Route Matching](https://router.vuejs.org/guide/essentials/dynamic-matching.html#reacting-to-params-changes)