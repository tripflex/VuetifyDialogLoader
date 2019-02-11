<template>
  <div>
    <!-- Using hide-overlay below allows for clicking while progress showing-->
    <v-dialog v-model="dialog" persistent :width="options.width" v-bind:style="{ zIndex: options.zIndex }">
      <v-card :color="options.color" dark>
        <v-card-text>
          {{ message }}
          <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-snackbar
      v-model="snackbarVisible"
      :left="snackbar.location === 'left'"
      :right="snackbar.location === 'right'"
      :top="snackbar.location === 'top'"
      :bottom="snackbar.location === 'bottom'"
      :color="snackbar.color"
      :multi-line="snackbar.mode === 'multi-line'"
      :timeout="snackbar.timeout"
      :vertical="snackbar.mode === 'vertical'"
    >
      {{ snackbar.message }}
      <v-btn
        flat
        :color="snackbar.close.color"
        @click="hideSnackbar"
      >
        {{ snackbar.close.text }}
      </v-btn>
    </v-snackbar>
  </div>
</template>

<script>
  /**
   * Vuetify DialogLoader Component (with SnackBar)
   *
   * This component use the Vuetify Dialog Loader to show loading dialog, with optional SnackBar notifications (snackbar disabled by default)
   *
   * Based off eolant's Vuetify Confirm Dialog (https://gist.github.com/eolant/ba0f8a5c9135d1a146e1db575276177d)
   *
   * Insert component where you want to use it:
   * <dialog-loader ref="dialogLoader"></dialog-loader>
   *
   * Alternatively you can place it in main App component and access it globally via this.$root.$confirm
   * <template>
   *   <v-app>
   *     ...
   *     <dialog-loader ref="dialogLoader"></dialog-loader>
   *   </v-app>
   * </template>
   *
   * mounted() {
   *   this.$root.$dialogLoader = this.$refs.dialogLoader
   * }
   *
   *
   * Standard Usage Without Callback:
   * ----
   * this.$root.$dialogLoader.show( 'Doing some action...', { color: 'primary' } )
   * this.$root.$dialogLoader.hide()
   *
   * this.$root.$dialogLoader.showSnackbar( 'Action failed! Oh no!', { color: 'error' } ) --- snackbar will hide after timeout value
   * this.$root.$dialogLoader.hideSnackbar() -- only necessary if timeout is set to 0 or need to hide snackbar for some other reason
   *
   * Advanced Usage (with optional callback that returns a promise)
   * ----
   *  -- Start Action Loader
   * this.$root.$dialogLoader.start(message, options, callback, snackbar)
   *
   * Argument 3 (callback) should be a function that returns a Promise (optional)
   * Argument 4 (snackbar) can be a boolean (true/false) or object (snackbar options) to enable showing snackbar when promise is resolved/rejected (optional)
   *
   * Example:
   * this.$root.$dialogLoader.start( 'Removing Someting...', { color: 'purple' }, promiseBasedFn, true )
   *
   *  -- Stop Action Loader (and show Snackbar)
   * this.$root.$dialogLoader.stop(message, snackbarOptions, callback)
   *
   * this.$root.$dialogLoader.stop('I completed some action! Yay!', { timeout: 5000, location: 'top' }, () => { console.log( 'Callback after snackbar hidden' ) })
   */
  export default {
    data: () => ({
      dialog: false,
      message: null,
      options: {
        color: 'primary',
        width: 290,
        zIndex: 200
      },
      snackbarVisible: false,
      snackbar: {
        enabled: false,
        color: 'success',
        mode: 'multi-line',
        timeout: 4000,
        message: 'Success!',
        callback: null,
        location: 'bottom',
        close: {
          text: 'Close',
          color: ''
        }
      }
    }),
    methods: {
      /**
       * Show loader without any callbacks or timeout (must be manually hidden)
       * @param {string} message
       * @param {object} options
       */
      show(message, options) {
        this.dialog = true
        this.message = message
        this.options = Object.assign(this.options, options)
      },
      /**
       * Hide loader (and show snackbar if enabled)
       */
      hide(){
        this.dialog = false
        if( this.snackbar.enabled ){
          this.showSnackbar()
        }
      },
      /**
       * Start/Show Loader (and maybe Snackbar)
       *
       * @param {string} message
       * @param {object} options
       * @param {Promise.<function>} [callback]
       * @param {boolean|object} [snackbar]
       */
      start(message, options, callback, snackbar) {
        this.show( message, options )

        if( typeof snackbar === 'object' ){
          this.snackbar.enabled = true
          this.snackbar = Object.assign(this.snackbar, snackbar)
        } else if( typeof snackbar !== 'undefined' ){
          this.snackbar.enabled = true
        }

        if( typeof callback === 'function' ){
          callback().then( this.hide ).catch( error => {
            this.snackbar.color = 'error'
            this.snackbar.message = error
            this.hide()
          })
        }
      },
      /**
       * Stop/Hide loader and show snackbar with optional callback
       *
       * @param {string} message
       * @param {object} snackbarOptions
       * @param {function} callback
       */
      stop(message, snackbarOptions, callback){
        this.hide()

        this.snackbar.enabled = true
        this.snackbar = Object.assign(this.snackbar, snackbarOptions)

        if( typeof callback === 'function' ){
          this.snackbar.callback = callback
          // Use our own timeout to call callback
          setTimeout( this.hideSnackbar, this.snackbar.timeout )
          // Set to zero to allow our timeout above to handle hiding
          this.snackbar.timeout = 0
        }
      },
      /**
       * Hide Snackbar (and call callback if previously set)
       */
      hideSnackbar(){
        this.snackbarVisible = false
        if( this.snackbar.callback ){
          this.snackbar.callback()
        }
      },
      /**
       * Show Snackbar
       * @param {string} message
       * @param {object} snackbarOptions
       */
      showSnackbar( message, snackbarOptions ){
        if( message ){
          this.snackbar.message = message
        }
        this.snackbar.enabled = true

        if( typeof snackbar === 'object' ){
          this.snackbar = Object.assign(this.snackbar, snackbarOptions)
        }

        this.snackbarVisible = true
      }
    }
  }
</script>
