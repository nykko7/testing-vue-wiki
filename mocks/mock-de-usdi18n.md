Sabemos que para poder hacer un mock de algún método o función dentro de nuestro componente, es cuestión de declararlo antes y a la hora de montar nuestro componente, darselo como argumento para que este lo utilize a la hora de montarlo. Un ejemplo de esto es el mock de [$route](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Mock-de-$route-y-$router):

```javascript
const $route = {
  params: {
    id: '1',
  },
}
```

Y a la hora de montar nuestro componente:

```javascript
    wrapper = mount(ClinicalRecruitment, {
      ...

      mocks: {
        $route,
      },
    })
```

En esta entrada, veremos como hacer un mock de `$i18n` para una Vuex.Store y para nuestros componentes.

<h3> Mock de $i18n para una Vuex.Store </h3>

Que pasa cuando tenemos que mockear algo dentro de una `store` de `Vuex`? En este proyecto se tiene la siguiente `store` que se utiliza para establecer un proyecto clinico. El problema a hora de testear, es la siguiente mutacion: 

```javascript
  SET_PROJECT(state, payload) {
    state.elements.push({
      id: payload.id,
      action: '',
      items: [
        {
          icon: '',
          title: `${this.$i18n.t('navDrawer.configuration')}`,
          to: `/${payload.id}/clinical-test`,
        },
        {
          icon: '',
          title: this.$i18n.t('navDrawer.forms'),
          to: `/${payload.id}/forms/list`,
        },
        {
          icon: '',
          title: this.$i18n.t('navDrawer.participants'),
          to: `/${payload.id}/participants`,
        },
        {
          icon: '',
          title: this.$i18n.t('navDrawer.collaborators'),
          to: `/${payload.id}/collaborators`,
        },
        {
          icon: '',
          title: this.$i18n.t('navDrawer.changelog'),
          to: `/${payload.id}/changelog`,
        },
        {
          icon: '',
          title: this.$i18n.t('navDrawer.project'),
          to: `/${payload.id}/project`,
        },
      ],
      title: payload.title,
      publish: payload.publish,
      configuration: payload.configuration,
    })
  },
```

Como podemos notar, se utiliza la libreria de i18n para traducir el texto que aparece en pantalla a la hora de utilizar la `store`. Sabemos que para poder utilizar la store dentro de nuestro proyecto es necesario crear una nueva Vuex.Store dentro de nuestro test:

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
```

Pero al montar nosotros nuestra tienda, esta no tiene conocimiento de la libreria i18n. Lo que tenemos que hacer entonces es darle a esta `store` un mock de esta libreria. Para esto, utilizaremos la propiedad `prototype`: 

```javascript
Vuex.Store.prototype.$i18n = jest.fn().mockReturnValue({})
Vuex.Store.prototype.$i18n.t = jest.fn().mockReturnValue({})
```

La propiedad `prototype` de JS nos permite añadir nuevos metodos a constructores. En nuestro caso, añadimos el metodo de `$i18n`, y un metodo encadenado a este llamado `$t`. Como se puede notar, se utiliza una función mock, que en si reemplaza la funcionalidad original de la función y permite continuar con el flujo normal de nuestra tienda. El metodo de `mockReturnValue()` acepta un valor que sera retornado cuando la funcion mock es llamada. En este caso como lo que queriamos era poder utilizar la tienda para configurar proyectos, no nos interesa mucho lo que regresa la funcion, asi que por eso se deja como un objeto vacio. Sin embargo, esto se puede utilizar para observar si una función esta siendo llamada correctamente.

<h3> Mock de $i18n para nuestro componente </h3>

Parecido al mock de `$route`, tenemos ahora que hacer un mock para el metodo `$t` de `$i18n`. Este es mas corto que el anteriormente mencionado, en este caso se puede colocar directamente en la seccion de `mocks` a la hora de montar nuestro componente:

```javascript
      mocks: {
        ...
        $t: (msg) => msg,
      },
```

Con esto, el mensaje que reciba esta función nos devolverá el mismo mensaje, con esto nuestro componente ya puede ser montado sin errores. 

<h3> Referencias </h3>

- [Mock Functions](https://jestjs.io/docs/mock-functions)

- [Mock Return Value](https://jestjs.io/docs/mock-function-api#mockfnmockreturnvaluevalue)

- [How to access this.app and/or a injected plugin in a Vuex store test with Nuxt and Jest
](https://stackoverflow.com/questions/54481326/how-to-unit-test-vue-js-components-that-use-nuxt-i18n)

- [Mocking global objects](https://lmiller1990.github.io/vue-testing-handbook/mocking-global-objects.html#mocking-global-objects)

- [How to unit test Vue.js components that use nuxt-i18n](https://stackoverflow.com/questions/54481326/how-to-unit-test-vue-js-components-that-use-nuxt-i18n)