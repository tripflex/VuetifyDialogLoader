# VuetifyDialogLoader
Vuetify Dialog Loader Component (with SnackBar) component that can be used locally or globally

This component use the Vuetify Dialog Loader to show loading dialog, with optional SnackBar notifications (snackbar disabled by default)

Based off eolant's Vuetify Confirm Dialog (https://gist.github.com/eolant/ba0f8a5c9135d1a146e1db575276177d)

### Demo/Sandbox
https://codesandbox.io/s/1v4mo4j1zl?module=%2Fsrc%2Fcomponents%2FDialogLoader.vue

### Usage

Insert component where you want to use it:
```html
<dialog-loader ref="dialogLoader"></dialog-loader>
```
Alternatively you can place it in main App component and access it globally via `this.$root.$confirm`

```vue
<template>
  <v-app>
    ...
    <dialog-loader ref="dialogLoader"></dialog-loader>
  </v-app>
</template>
```
#### Standard Usage Without Callback:
```javascript
this.$root.$dialogLoader.show( 'Doing some action...', { color: 'primary' } )
```
```javascript
this.$root.$dialogLoader.hide()
```
snackbar will hide after timeout value:
```javascript
this.$root.$dialogLoader.showSnackbar( 'Action failed! Oh no!', { color: 'error' } )
```
only necessary if timeout is set to 0 or need to hide snackbar for some other reason:
```javascript
this.$root.$dialogLoader.hideSnackbar()
```

#### Advanced Usage (with optional callback that returns a promise)

**Start Action Loader**
 - Argument 3 (callback) should be a function that returns a Promise (optional)
 - Argument 4 (snackbar) can be a boolean (true/false) or object (snackbar options) to enable showing snackbar when promise is resolved/rejected (optional)
```javascript
this.$root.$dialogLoader.start(message, options, callback, snackbar)
```

_Example:_
```javascript
this.$root.$dialogLoader.start( 'Removing Someting...', { color: 'purple' }, promiseBasedFn, true )
```

**Stop Action Loader (and Show Snackbar)**
```javascript
this.$root.$dialogLoader.stop(message, snackbarOptions, callback)
```

_Example:_
```javascript
this.$root.$dialogLoader.stop('I completed some action! Yay!', { timeout: 5000, location: 'top' }, () => { console.log( 'Callback after snackbar hidden' ) })
```

### Combined with Confirm Dialog
This can be combined with eolant's Vuetify Confirm Dialog (https://gist.github.com/eolant/ba0f8a5c9135d1a146e1db575276177d) component to create a dialog along with loader and snackbar.

Example:
`this.removeClientPromiseFn` _is a function that returns a promise after performing db actions_

```javascript
async toggleDelete( client ){
  if( await this.$root.$confirm('Delete?', 'Are you sure you want to remove this client', { color: 'red' }) ){
    this.$root.$dialogLoader.start( 'Removing Client...', {}, this.removeClientPromiseFn, true )
  }
},
```

Another example just as a demo using `setTimeout`:

```javascript
async toggleDelete( client ){
  if( await this.$root.$confirm('Delete?', 'Are you sure you want to remove this client', { color: 'red' }) ){

    this.$root.$dialogLoader.start( 'Removing Client...', {}, () => {

      return new Promise((resolve, reject) => {
        setTimeout( ()=> {
          resolve()
          // reject( 'Unable to remove client!' )
        }, 3000 )
      })

    }, true )

  }
}
```

### Props
@eolant for his Confirm Dialog component which this is based off of
https://gist.github.com/eolant/ba0f8a5c9135d1a146e1db575276177d

### License
MIT
