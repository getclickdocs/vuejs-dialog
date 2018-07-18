# Vuejs Dialog Plugin

> A lightweight, promise based alert, prompt and confirm dialog.

[![npm version](https://badge.fury.io/js/vuejs-dialog.svg)](https://badge.fury.io/js/vuejs-dialog)
[![Build Status](https://travis-ci.org/Godofbrowser/vuejs-dialog.svg?branch=master)](https://travis-ci.org/Godofbrowser/vuejs-dialog)
[![Scrutinizer](https://img.shields.io/scrutinizer/g/Godofbrowser/vuejs-dialog.svg?branch=master)](https://scrutinizer-ci.com/g/Godofbrowser/vuejs-dialog/?branch=master)
[![npm](https://img.shields.io/npm/dt/vuejs-dialog.svg)]()

![Vuejs Dialog Plugin](./src/docs/img/html-enabled.png?raw=true "Vuejs Dialog Plugin example")
![Vuejs Dialog Plugin](./src/docs/img/demo.gif?raw=true "Vuejs Dialog Plugin usage demo")


## Demo

[https://godofbrowser.github.io/vuejs-dialog/](https://godofbrowser.github.io/vuejs-dialog/)

## Important updates in version v1.x.x
1. Dialog will always resolve with an object. (i.e callback for proceed always will receive an object)
2. For directives usage, the object returned in (1) above will include a node. The node is the element the directive was bound to (see issue #5)
3. Styles will have to be included explicitly as they have been extracted into a separate file (see issue #28)
4. If loader is enabled globally, and a dialog is triggered via a directive without a callback, the loader is ignored for clicks on proceed
5. Custom class injection on parent node (see issue #25)
6. Ability to register custom views. This allows for custom logic, custom buttons, etc (see issue #13, #14, #33)

## Installation

#### HTML

  ```html
  // Include vuejs
 <script type="text/javascript" src="./path/to/vue.min.js"></script>
 
 // Include vuejs-dialog plugin
 <link href="./path/to/vuejs-dialog.min.css" rel="stylesheet">
 <script type="text/javascript" src="./path/to/vuejs-dialog.min.js"></script>
 
 <script>
// Tell Vue to install the plugin.
window.Vue.use(VuejsDialog.default)
</script>
  ```
#### Package Manager
```javascript
// installation via npm 
npm i -S vuejs-dialog

// or

// installation via yarn
yarn add vuejs-dialog
```

```javascript
// then

// import into project
import Vue from "vue"
import VuejsDialog from "vuejs-dialog"

// include the default style
import 'vuejs-dialog/vuejs-dialog.min.css'

// Tell Vue to install the plugin.
Vue.use(VuejsDialog)
```

## Basic Usage

```javascript
// Anywhere in your Vuejs App.

this.$dialog.confirm('Please confirm to continue')
	.then(function () {
		console.log('Clicked on proceed')
	})
	.catch(function () {
		console.log('Clicked on cancel')
	});
```


## Usage with ajax - Loader enabled
```javascript
// Anywhere in your Vuejs App.

this.$dialog.confirm("If you delete this record, it'll be gone forever.", {
    loader: true // default: false - when set to true, the proceed button shows a loader when clicked.
    			// And a dialog object will be passed to the then() callback
})
	.then((dialog) => {
		// Triggered when proceed button is clicked

		// dialog.loading(false) // stops the proceed button's loader
		// dialog.loading(true) // starts the proceed button's loader again
		// dialog.close() // stops the loader and close the dialog

		// do some stuff like ajax request.
		setTimeout(() => {
			console.log('Delete action completed ');
			dialog.close();
		}, 2500);

	})
    .catch(() => {
        // Triggered when cancel button is clicked

        console.log('Delete aborted');
    });
```

## Usage as a directive

__If you don't pass a message, the global/default message would be used.__
```html
<button type="submit" v-confirm="">submit</button>
```

```html
// Callbacks can be provided
// Note: If "loader" is set to true, the makeAdmin callback will receive a "dialog" object
// Which is useful for closing the dialog when transaction is complete.
<button v-confirm="{ok: makeAdmin, cancel: doNothing, message: 'User will be given admin privileges. Make user an Admin?'}">Make Admin</button>
```
```javascript
methods: {
    makeAdmin: function() {
        // Do stuffs
        
    },
    doNothing: function() {
        // Do nothing or some other stuffs
    }
}
```

__A more practical use of ths `v-confirm` directive inside a loop - Solution 1__

```html
// While looping through users
<button v-for="user in users"
        v-confirm="{
            loader: true,
            ok: dialog => makeAdmin(dialog, user), 
            cancel: doNothing, 
            message: 'User will be given admin privileges. Make user an Admin?'}"
>
Make Admin
</button>
```
```javascript
methods: {
    makeAdmin: function(dialog, user) {
        // Make user admin from the backend
        /* tellServerToMakeAdmin(user) */
        
        // When completed, close the dialog
        /* dialog.close() */
        
    },
    doNothing: function() {
        // Do nothing or some other stuffs
    }
}
```


__( new ) A more practical use of ths `v-confirm` directive inside a loop - Solution 2__

```html
// While looping through users
<button v-for="user in users"
        :data-user="user"
        v-confirm="{
            loader: true,
            ok: makeAdmin, 
            cancel: doNothing, 
            message: 'User will be given admin privileges. Make user an Admin?'}"
>
Make Admin
</button>
```
```javascript
methods: {
    makeAdmin: function(dialog) {
        let button = dialog.node // node is only available if triggered via a directive
        let user = button.dataset.user
        
        // Make user admin from the backend
        /* tellServerToMakeAdmin(user) */
        
        // When completed, close the dialog
        /* dialog.close() */
        
    },
    doNothing: function() {
        // Do nothing or some other stuffs
    }
}
```

__For v-confirm directive, if an "OK" callback is not provided, the default event would be triggered.__

```html
// Default Behaviour when used on links
<a href="http://example.com" v-confirm="'This will take you to http://example.com. Proceed with caution'">Go to example.com</a>

```

## Setting a dialog title (new)

You can now set a dialog title by passing your message as an object instead of a string.
The message object should contain a `title` and `body`

```javascript
let message = {
    title: 'Vuejs Dialog Plugin',
    body: 'A lightweight, promise based alert, prompt and confirm dialog'
}

this.$dialog.confirm(message)
```


### Options
```javascript
// Parameters and options

let message = "Are you sure?";

let options = {
    html: false, // set to true if your message contains HTML tags. eg: "Delete <b>Foo</b> ?"
    loader: false, // set to true if you want the dailog to show a loader after click on "proceed"
    reverse: false, // switch the button positions (left to right, and vise versa)
    okText: 'Continue',
    cancelText: 'Close',
    animation: 'zoom', // Available: "zoom", "bounce", "fade"
    type: 'basic', // coming soon: 'soft', 'hard'
    verification: 'continue', // for hard confirm, user will be prompted to type this to enable the proceed button
    verificationHelp: 'Type "[+:verification]" below to confirm', // Verification help text. [+:verification] will be matched with 'options.verification' (i.e 'Type "continue" below to confirm')
    clicksCount: 3, // for soft confirm, user will be asked to click on "proceed" btn 3 times before actually proceeding
    backdropClose: false // set to true to close the dialog when clicking outside of the dialog window, i.e. click landing on the mask 
    customClass: '' // Custom class to be injected into the parent node for the current dialog instance
};

this.$dialog.confirm(message, options)
	.then(function () {
	    // This will be triggered when user clicks on proceed
	})
	.catch(function () {
	    // This will be triggered when user clicks on cancel
	});
```
### Global Configuration
```javascript
// You can also set all your defaults at the point of installation.
// This will be your global configuration

Vue.use(VuejsDialog, {
    html: true, 
    loader: true,
    okText: 'Proceed',
    cancelText: 'Cancel',
    animation: 'bounce', 
})

// Please note that local configurations will be considered before global configurations.
// This gives you the flexibility of overriding the global config on individual call.
```

### CSS Override

If you have included the plugin's style and wish to make a few overides, you can do so with basic css, ex:
```css
.dg-btn--ok {
     border-color: green;
 }
 
.dg-btn-loader .dg-circle {
    background-color: green;
}
```

### Pro tip
You can use any of the options in your verification help text. Example:

```javascript
this.$dialog.confirm($message, {
    verificationHelp: 'Enter "[+:verification]" below and click on "[+:okText]"',
     type: 'hard'
})
```
## More flexibility with Custom components

```vue
/* File: custom-component.vue */
<template>
    <div class="custom-view-wrapper">
        <template v-if=messageHasTitle>
            <h2 v-if="options.html" class="dg-title" v-html="messageTitle"></h2>
            <h2 v-else class="dg-title">{{ messageTitle }}</h2>
        </template>
        <template v-else>
            <h2>Share with friends</h2>
        </template>

        <div v-if="options.html" class="dg-content" v-html="messageBody"></div>
        <div v-else class="dg-content">{{ messageBody }}</div>
        <br/>

        <ok-btn @click="handleShare('facebook')" :options="options">Facebook</ok-btn>
        <ok-btn @click="handleShare('twitter')" :options="options">Twitter</ok-btn>
        <ok-btn @click="handleShare('googleplus')" :options="options">Google+</ok-btn>
        <ok-btn @click="handleShare('linkedin')" :options="options">LinkedIn</ok-btn>
        <cancel-btn @click="handleDismiss()" :options="options">Dismiss</cancel-btn>
    </div>
</template>

<script>
    import DialogMixin from 'vuejs-dialog/vuejs-dialog-mixin.min.js' // Include mixin
    import OkBtn from 'path/to/components/ok-btn.vue'
    import CancelBtn from 'path/to/components/cancel-btn.vue'

    export default {
        mixins: [ DialogMixin ],
        methods: {
            handleShare(platform) {
                this.proceed(platform) // included in DialogMixin
            },
            handleDismiss() {
                this.cancel() // included in DialogMixin
            }
        },
        components: { CancelBtn, OkBtn }
    }
</script>

<style scoped="">
    button {
        width: 100%;
        margin-bottom: 10px;
        float: none;
    }
</style>

```

```javascript
import CustomView from './path/to/file/custom-component.vue'
const VIEW_NAME = 'my-unique-view-name'

let vm = new Vue({
    created() {
        this.$dialog.registerComponent(VIEW_NAME, CustomView)
    },
    methods: {
        showCustomView(){
            this.$dialog.alert(trans('messages.html'), {
                view: VIEW_NAME, // can be set globally too
                html: true,
                animation: 'fade',
                backdropClose: true
            })
        }
    }
})
```

 ... and you get your custom view


![Vuejs Dialog Plugin](./src/docs/img/custom-view.png?raw=true "Vuejs Dialog Plugin custom view demo")

## What's next?
### Custom components (coming soon!!!)

```vue
/* File: custom-component.vue */
<template>
    <div class="custom-view-wrapper">
        <h2>Share this amazing plugin</h2>

        <div v-if="options.html" class="dg-content" v-html="messageBody"></div>
        <div v-else="" class="dg-content">{{ messageBody }}</div>

        <ok-btn @click="share('share url')" :loading="loading" :options="options" :enabled="true">Share on facebook</ok-btn>
        <ok-btn @click="share('share url')" :loading="loading" :options="options" :enabled="true">Share on twitter</ok-btn>
        <ok-btn @click="share('share url')" :loading="loading" :options="options" :enabled="true">Share on linkedIn</ok-btn>
        <cancel-btn @click="proceed()" :loading="loading" :options="options" :enabled="true">Maybe later!</cancel-btn>
    </div>
</template>

<script>
    import DialogMixin from 'vuejs-dialog/js/mixins/dialog-mixin'

    export default {
        mixins: [DialogMixin], // All dialog methods (proceed, cancel, etc), state variables (options, etc) and computed properties are included
        methods: {
            share(url) {
                // popup share window
            }
        }
    }
</script>

<style scoped="">
    button {
        width: 100%;
        margin-bottom: 10px;
        float: none;
    }
</style>
```

```javascript
import TestView from './path/to/file/custom-component.vue'
const VIEW_NAME = 'my-view'

let vm = new Vue({
    created() {
        this.$dialog.registerComponent(VIEW_NAME, TestView)
    },
    methods: {
        showCustomView(){
            this.$dialog.alert(trans('messages.html'), {
                view: VIEW_NAME, // can be set globally too
                html: true,
                animation: 'fade',
                backdropClose: true
            })
        }
    }
})
```
...and you get your custom view
![Vuejs Dialog Plugin](./src/docs/img/custom-view.png?raw=true "Vuejs Dialog Plugin custom view demo")

# License

[MIT](http://opensource.org/licenses/MIT)

## Contributing

* Fork it!
* Create your feature branch: git checkout -b my-new-feature
* Commit your changes: git commit -am 'Add some feature'
* Push to the branch: git push origin my-new-feature
* Submit a pull request :)
