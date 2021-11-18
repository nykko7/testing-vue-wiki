A veces en nuestros componentes, utilizamos funciones fuera de nuestro `<script>` y las importamos al inicio de esta. Como podemos testear que esta función esta siendo llamada correctamente en nuestro componente si esta función no esta dentro de nuestro wrapper? 

Tomemos como ejemplo lo siguiente. Tenemos un botón que al hacer clic en el, hace un llamado a un método que utiliza una función importada. El botón:

```html
              <v-btn
                id="signBtn"
                :loading="loading"
                block
                type="submit"
                :disabled="
                  !email || !name || !password || !passwordConfirm || !terms
                "
              >
```

El método al cual llama al hacer click:

```javascript
import { signUp, setUserDoc } from '~/services/index.js'
    async register() {
      try {
        this.loading = true
        const { user } = await signUp(this.$fire, {
          email: this.email,
          password: this.password,
        })
        await updateProfile(this.$fire, {
          displayName: this.name,
        })
        await setUserDoc(this.$fire, user.uid, {
          email: this.email,
          displayName: this.name,
        })
        this.$store.dispatch('user/set_displayname', this.displayName)
        await this.sendEmailVerificationSengrid()
      } catch (error) {
        this.loading = false
        this.$store.dispatch('notifications/open', {
          color: 'error',
          text: error.message,
        })
      }
    },
```

Como podemos notar, signUp es una función que esta siendo importada. Si queremos espiar si esta función es llamada, no podemos hacerlo como lo hemos hecho anteriormente (llamando un espía revisando nuestra instancia de wrapper.vm), ya que esta función, no se encuentra en nuestro componente. 

Para poder espiarla correctamente, es necesario que en nuestro test importemos el archivo donde se tienen estas funciones. De esta forma podemos espiar si la función fue llamada correctamente desde nuestro componente hacía ese archivo donde se tienen las funciones. Nuestro `import` se vería de la siguiente forma:

```javascript
import * as services from '~/services/firebase.js'
```

Con esto, podemos espiar este import para ver si cuando nuestro botón es "clickeado" llama correctamente a esta función o servicio. El espía se vera de la siguiente forma:

```javascript
const serviceSpy = jest.spyOn(services, 'signUp')
```

En la instancia, el primer argumento del espia, le damos el nombre con el que importamos nuestras funciones, y en el segundo la función que se debe de activar o disparar al hacer clic en nuestro botón. Con esto, podemos espiar en funciones externas a nuestro componente, pero que son utilizadas por el. Nuestro test se ve entonces de la siguiente forma:

```javascript
    it('Check that signUp method is called correctly when trying to login', async () => {
        wrapper = mount(Signup, {
            localVue,
            vuetify,
            router,
            store,

            stubs: {
                NuxtLink: true,
            },

            mocks: {
                $t: (msg) => msg,
                $fire,
            },

            data() {
                return {
                    email: 'naboman@yahoo.com',
                    name: 'Nabo',
                    password: 'contra',
                    passwordConfirm: 'contra',
                    terms: true,
                }
            },

            attachTo: document.body,
        })

        const spylogIn = jest.spyOn(wrapper.vm, 'register')
        const serviceSpy = jest.spyOn(services, 'signUp')

        const signUpButton = wrapper.find('#signBtn')
        signUpButton.trigger('click')
        await wrapper.vm.$nextTick()

        expect(spylogIn).toHaveBeenCalledTimes(1)
        expect(serviceSpy).toHaveBeenCalledTimes(1)
    })
```

### Referencias 
- [How to mock a directly-imported function using Jest?](https://stackoverflow.com/questions/54222277/how-to-mock-a-directly-imported-function-using-jest)