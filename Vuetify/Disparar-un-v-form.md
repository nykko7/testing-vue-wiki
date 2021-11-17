Anteriormente se ha hablado de como seleccionar una opcion en un `v-select` o como seleccionar una opcion en un `text-field`. En esta entrada veremos como es que podemos disparar una acción o método de un `v-form`. 

Tomemos como ejemplo el siguiente componente. A continuación se presenta el `template` de este:

```html
       <v-form @submit.prevent="register">
              ...
              <v-btn
                id="signBtn"
                :loading="loading"
                block
                type="submit"
                :disabled="
                  !email || !name || !password || !passwordConfirm || !terms
                "
              >
                {{ $t('login.signUp') }}
              </v-btn>
       </v-form>
```

Como podemos notar, nuestro botón no es el que dispara la acción para subir los datos en nuestro registro, si no que es `v-form` el que se encarga de esto. Si buscamos nuestro botón en nuestro wrapper y hacemos click en el:

```javascript
    const signUpButton = wrapper.find('#signBtn')
    signUpButton.trigger('click')
    await wrapper.vm.$nextTick()
```

El método no se dispara, ya que este no depende solamente del botón, si no de `v-form` también. Como podemos entonces hacer que este se dispare? Para esto necesitamos que nuestro `v-form` este renderizado en nuestro DOM. Este es un problema parecido al visto con [v-dialog](https://gitlab.com/oscity/vpress/talaria-front-end/-/wikis/Vuetify/Test-para-un-v-dialog). En este caso, para poder realizar esto, utilizamos la opcion de montar de vue-test-utils llamada `attachTo`:

```javascript
    wrapper = mount(Signup, {
    ...
      attachTo: document.body,
    })
```

Esto permite que nuestro `v-form` sea accesible en nuestro wrapper y asi poder disparar la acción con nuestro botón, como normalmente lo haríamos. 

### Referencias
- [attachTo](https://vue-test-utils.vuejs.org/api/options.html#attachto)
- [Fail to trigger Vuetify form submit using submit button in Jest](https://stackoverflow.com/questions/60685233/fail-to-trigger-vuetify-form-submit-using-submit-button-in-jest)