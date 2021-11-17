En el ciclo de vida de Vue, tenemos varios pasos por los cuales pasa nuestra app hasta que esta es renderizada. Uno de ellos es el paso de `created()` en donde el DOM aun no ha sido montado, pero tienes acceso a los datos dinamicos de tu app. En el siguiente componente, cambiamos los valores de los datos en base a que tipo de acción es llamada en nuestra aplicación:

```javascript
  created() {
    this.unsubscribe = this.$store.subscribe((mutation, state) => {
      if (mutation.type === 'notifications/SHOW_NOTIFICATION') {
        this.text = state.notifications.text
        this.color = state.notifications.color
        this.timeout = state.notifications.timeout
        this.show = state.notifications.show
      } else if (mutation.type === 'notifications/CLOSE_NOTIFICATION') {
        this.show = state.notifications.show
      }
    })
  },
```

En el caso de que la mutación sea tipo `SHOW_NOTIFICATION`, entonces los datos de nuestro componente cambiaran a los que se utilizaron de argumento al momento de llamar a esta acción. Ahora bien, si queremos testear nosotros usando esta acción, y después comprobar que nuestro componente si muestra esta información correctamente, tendremos problemas debido a que el DOM no se actualiza con esta información a la hora de montarse. Nuestro test se ve de la siguiente forma:

```javascript
    wrapper = mount(Notification, {
      localVue,
      store,
      vuetify,
    })

    store.dispatch('notifications/open', {
      color: 'success',
      text: 'Notificacion',
    })
``` 

Como podemos notar, a la hora de montar nuestro componente, este lo hace con los datos por default que tiene el componente, y al hacer esto, no muestra en pantalla la información que le estamos dando en nuestro `store.dispatch`. Para que nuestro componente muestre en pantalla la información que le dimos, es necesario que nuestro DOM se actualice. Para esto, podemos utilizar el método `wrapper.vm.$forceUpdate()`, el cual obliga a nuestro DOM a actualizarse. Con esto, la información que le dimos a nuestra acción ahora si será mostrada correctamente. 

Nuestro test queda entonces de la siguiente forma:

```javascript
  it('Check that the data for the notification is showed correctly', async () => {
    wrapper = mount(Notification, {
      localVue,
      store,
      vuetify,
    })

    store.dispatch('notifications/open', {
      color: 'success',
      text: 'Notificacion',
    })

    await wrapper.vm.$forceUpdate()

    expect(wrapper.vm.text).toEqual('Notificacion')
    expect(wrapper.text()).toEqual(expect.stringContaining('Notificacion'))
  })
```

### Referencias 
- [Vue test utils is updating the component data but not re-rendering the dom](https://stackoverflow.com/questions/53781794/vue-test-utils-is-updating-the-component-data-but-not-re-rendering-the-dom)
- [Difference between the created and mounted events in Vue.js](https://stackoverflow.com/questions/45813347/difference-between-the-created-and-mounted-events-in-vue-js)