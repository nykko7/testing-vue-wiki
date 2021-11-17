La aplicación de Talaria utiliza como framework a Vue, además de usar también a Vuetify y la librería Vuex para manejar el estado de la app. 

Para poder utilizar Jest con todas estas herramientas, es necesario hacer un setup inicial antes de empezar a hacer tests. A continuación se muestra como es el setup general y un ejemplo, propio de esta aplicación.

<h3> Instalación de vue-test-utils y vue-jest </h3>

Para poder hacer un test a uno de nuestros componentes, necesitamos utilizar la librería de “vue-test-utils”. Además, para poder utilizar jest para testear nuestros componentes, también necesitamos la librería “vue-jest” Para su instalación, utilizamos yarn:

```shell 
 yarn add @vue/test-utils vue-jest
```

<h3> Setup de Vuetify </h3>

Par poder utilizer Vuetify con Jest, debemos de crear un archivo de `setup.js ~/test/setup.js`. En este nuevo archivo, agregaremos las siguientes líneas:

```javascript
import Vue from  'vue'
import Vuetify from  'vuetify'

Vue.use(Vuetify)

```

Una ves que se guarde el archivo, debemos configurar Jest para que identifique estas líneas a la hora de testear. Para eso, en el archivo de `jest.config.js` agregamos lo siguiente:

```javascript
setupFiles: ['<rootDir>/test/setup.js'],
```

Con esto ya quedaría listo el setup para probar Vuetify. Ahora ya que esta configurado, pasemos a las configuraciones que se tienen que realizar en el test para poder testear Vuetify con Jest. Primero que nada, en nuestro archivo de test, importamos Vuetify:

```javascript
import Vuetify from 'vuetify'
```

Posteriormente, una vez se inicie el test, en este se debe de declarar una variable en donde se inicializa Vuetify, esto lo hacemos en el método de nuestro test “beforeEach”:

```javascript
describe('Logout screen',  ()  =>  {
    let vuetify
    let wrapper
    beforeEach(()  =>  {
	vuetify =  new  Vuetify()
    })
}
```

Por ultimo, al momento de montar nuestro wrapper, montamos nuestro componente y le damos como argumento el objeto de Vuetify para poder renderizarlo correctamente.

```javascript
wrapper =  mount(CustomComponent,  { vuetify })
```

<h3> Setup de Vuex</h3>

En el caso de Vuex, solo es necesario importar Vuex en nuestro test e importar los módulos que necesitamos de nuestra store. 

```javascript
 import Vuex from  'vuex'
 import  { state, mutations, getters }  from  '~/store/user'
```

> Nota: En este ejemplo en especifico, la store de “user” esta exportando cada modulo, por lo que se debe de importar específicamente el o los módulos que se van a utilizar.

Después de esto, debemos crear un localVue con Vuex, utilizando el método de createLocalVue. Este método retorna una clase de Vue en donde se añaden componentes y plugins, esto sin contaminar la clase global de Vue:

```javascript
const  localVue  =  createLocalVue()
localVue.use(Vuex)
```

Dentro de nuestro archivo donde estamos haciendo nuestro test, creamos la store a utilizar en el test:

```javascript
const  store  =  new Vuex.Store({
    modules:  {
	    user:  {
		    namespaced:  true,
		    state,
		    mutations,
		    getters,
	    },
    },
})
```

Y en el wrapper donde tenemos nuestro componente montado, le damos como argumentos a localVue y a store:

```javascript
wrapper =  mount(CustomComponent,  {
    localVue,
    store,
    vuetify,
})
```

<h3> Nuxt.JS </h3>

En el caso de este proyecto, se utilizo Nuxt.JS para manejar el routing. Para forzar que este routing se haga a través de esta librería, debemos agregar “nuxt” como atributo del elemento:

```html
<v-btn nuxt to="/login">Acceder</v-btn>
```

Y en nuestro wrapper, tenemos que colocar un stub, que simula el comportamiento del componente:

```javascript
wrapper =  mount(CustomComponent,  {
    localVue,
    store,
    vuetify,
    stubs:  {
	      NuxtLink:  true,
    },
})
```

De esta forma, Nuxt.JS ya puede ser utilizado para tests.


<h3> Conclusión </h3>
Con esto, ya queda listo el setup para poder testear componentes que utilizan Vuetify + Nuxt.JS + Vuex! 

A continuación se muestra un ejemplo de como se vería un test con todas las configuraciones anteriores. Este test revisa el componente de Login de este proyecto. 

```javascript
import Vuex from 'vuex'
import { mount, createLocalVue } from '@vue/test-utils'

import Vuetify from 'vuetify'
import Navbar from '~/components/Navbar.vue'
import { state, mutations, getters } from '~/store/user'

const localVue = createLocalVue()
localVue.use(Vuex)

describe('Logout screen', () => {
    let vuetify
    let wrapper
    beforeEach(() => {
        vuetify = new Vuetify()
    })

    it('logout', () => {
        const store = new Vuex.Store({
        modules: {
            user: {
                namespaced: true,
                state,
                mutations,
                getters,
             },
        	},
        })
        wrapper = mount(Navbar, {
            localVue,
            store,
            vuetify,
            stubs: {
                NuxtLink: true,
            },
        })

        expect(wrapper.find('#login').text()).toBe('Acceder')
    })
    it('login', () => {
        const store = new Vuex.Store({
        modules: {
            user: {
                namespaced: true,
                state,
                mutations,
                getters,
            },
        },
        })
        store.commit('user/ON_AUTH_STATE_CHANGED_MUTATION', {
            authUser: {
                displayName: 'Moha',
                uid: '',
            },
        })
        wrapper = mount(Navbar, {
            localVue,
            store,
            vuetify,
            stubs: {
                NuxtLink: true,
            },
        })

        // console.log(store.getters['user/isAuthenticated'])
        expect(wrapper.find('#login').isVisible()).toBe(false)
    })
})
```

<h3> Referencias </h3>

- [Vue Testing Handbook](https://lmiller1990.github.io/vue-testing-handbook/stubbing-components.html#automatically-stubbing-with-shallowmount)

- [[Vue warn]: Unknown custom element: <nuxt-link> - When running jest unit tests](https://stackoverflow.com/questions/49665571/vue-warn-unknown-custom-element-nuxt-link-when-running-jest-unit-tests)

- [How to Setup Jest Testing In Nuxt Js Project](https://dev.to/bawa_geek/how-to-setup-jest-testing-in-nuxt-js-project-5c84)

- [Vuex: Modules (Namespacing)](https://vuex.vuejs.org/guide/modules.html#namespacing)

- [Testing setup for Nuxt + Vuex + Vuetify with Jest](https://stackoverflow.com/questions/59810929/testing-setup-for-nuxt-vuex-vuetify-with-jest)

- [Unit Testing (Vuetify)](https://vuetifyjs.com/en/getting-started/unit-testing/)