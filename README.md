# NuxtJS Application

## Building RESTful UI's in NuxtJS

In this repository we will walk through how to spin up a NuxtJS application and how to approach creating a basic RESTful style layout. Before starting it is recommended that you have followed Nuxt's instructions for getting started and are using the starter page. We will be using `yarn` to install dependencies.

## About the App

We need to add some key dependencies to `package.json' first:

```json
...
"dependencies": {
  "@nuxtjs/auth": "^4.5.1",
  "@nuxtjs/axios": "^5.3.1",
  "bootstrap-vue": "^2.0.0-rc.11",
  "lodash.assign": "^4.2.0",
  "lodash.merge": "^4.6.1",
  "nuxt": "^1.0.0",
  "nuxt-i18n": "^3.3.0",
  "vee-validate": "^2.1.0-beta.6",
  "velocity-animate": "^1.5.1",
  "vue-moment": "^4.0.0",
  "vue-notification": "^1.3.10"
},
"devDependencies": {
  "babel-eslint": "^8.2.1",
  "eslint": "^4.15.0",
  "eslint-friendly-formatter": "^3.0.0",
  "eslint-loader": "^1.7.1",
  "eslint-plugin-vue": "^4.0.0",
  "node-sass": "^4.9.0",
  "postcss-preset-env": "^5.1.0",
  "sass-loader": "^7.0.3"
}
...
```

- `@nuxtjs/auth` for user login
- `@nuxtjs/axios` for using axios http client
- `bootstrap-vue` is our UI component toolkit
- `lodash` helper for handling merging of objects and arrays
- `vee-validate` for handling form validation
- `velocity-animate` and `vue-notification` for excellent toast notifications
- `vue-moment` for integrating the famous datetime formatting library with vue

Now we run `yarn` to install everything. Now go onto our configuration in `nuxt.config.js`:

```javascript
const i18n = require('./config/locales');
  /*
  ** Headers of the page
  */
  head: {
    title: 'NuxtJS App',
    meta: [
      {
        charset: 'utf-8'
      },
      {
        name: 'viewport',
        content: 'width=device-width, initial-scale=1'
      },
      {
        hid: 'description',
        name: 'description',
        content: 'NuxtJS App'
      }
    ],
    link: [
      {
        rel: 'icon',
        type: 'image/x-icon',
        href: '/favicon.ico'
      }
    ],
    css: [
      '@/assets/scss/app.scss'
    ],
    /*
    ** Customize the progress bar color
    */
    loading: {
      color: '#3B8070',
    },
    /*
    ** Build configuration
    */
    build: {
      /*
      ** Run ESLint on save
      */
      extend(config, {isDev, isClient}) {
        const vueLoader = config.module.rules.find((rule) => rule.loader === 'vue-loader')
        vueLoader.options.transformToRequire = {
          'img': 'src',
          'image': 'xlink:href',
          'b-img': 'src'
          'b-img-lazy': ['src', 'blank-src'],
          'b-card': 'img-src',
          'b-card-img': 'img-src',
          'b-carousel-slide': 'img-src',
          'b-embed': 'src'
        };
        if (isDev && isClient) {
          config.module.rules.push({
            enforce: 'pre',
            test: /\.(js|vue)$/,
            loader: 'eslint-loader',
            exclude: /(node_modules)/
          })
        }
      }
    },
    modules: [
      ['bootstrap-vue/nuxt', {css: false}],
      ['nuxt-i18n', i18n],
      ['@nuxtjs/axios'],
      ['@nuxtjs/auth']
    ],
    plugins: [
      {src: '~/plugins/vue-notifications', ssr: false},
      {src: '~/plugins/vee-validate'},
      {src: '~/plugins/vue-moment'}
    ],
    router: {
      middleware: ['auth']
    },
    auth: {
      strategies: {
        local: {
          endpoints: {
            login: {url: '/auth/login', method: 'post', propertyName: 'jwt'},
            logout: {url: '/auth/logout', method: 'post'},
            user: {url: '/auth/user', method: 'get', propertyName: 'user'}
          }
        }
      }
    },
    axios: {
      /*
      ** Set API_URL environment variable to configure access to the API
      */
      baseURL: process.env.API_URL || 'http://localhost:3001/',
      redirectError: {
        401: '/login',
        404: '/notfound'
      }
    }
  }
```

From top to bottom the key additions are:

- Import a `nuxt-i18n` configuration script at the top, and then using it in the module definitions
- `css` property to handle configuring Bootstrap the way we like using SCSS
- `build` section has `vueLoader` configuration for transforming image paths in nuxtjs for related `bootstrap-vue` components
- `modules` includes the modules for `bootstrap-vue`, `nuxt-i18n`, `axios`, and `auth`
- `plugins` configuration scripts for notifications, validate, and moment
- `router` added the `auth` middleware
- Last is the configuration section for `auth` and `axios`

When you add plugins to Nuxt, they're virtually the same every time. If you run into problems where `document` or `window` is undefined, this is because a dependency expects a browser, but Nuxt does Nodejs Server-Side Rendering by default. `document` nor `window` exist here.

## Add some style

Create `~/app/scss/app.scss`:

```scss
@import '~bootstrap/scss/bootstrap';
```

## Make some plugin files

`~/plugins/vue-notifications.js`:

```javascript
import Vue from 'vue';
import Notifications from 'vue-notifications';
import velocity from 'velocity-animate';

Vue.use(Notifications, { velocity });
```

`~/plugins/vue-moment.js`:

```javascript
import Vue from 'vue';
import VueMoment from 'vue-moment';

Vue.use(VueMoment);
```

`~/plugins/vee-validate.js`:

```javascript
import Vue from 'vue';
import VeeValidate from 'vee-validate';

Vue.use(VeeValidate, {
  inject: true,
  fieldsBagName: 'veeFields'
});
```

Next we have to add support for i18n. Create a folder called `config/locales`, and in this folder, create an `index.js` file:

```javascript
module.exports = {
  locales: [
    {
      code: 'en',
      iso: 'en-US',
      name: 'English',
      file: 'en.json'
    }
  ],
  defaultLocale: 'en',
  seo: true,
  lazy: true,
  detectBrowserLanguage: {
    cookieKey: 'redirected',
    useCookie: true
  },
  langDir: 'config/locales/',
  parsePages: false,
  pages: {},
  vueI18n: {
    fallbackLocale: 'en'
  }
};
```

It's easy to add a new locale by just duplicating the object in the `locales` array and adding the values needed. Now add `en.json` as a standard JSON formatted file:

```json
{
  "actions": "Actions",
  "yes": "Yes",
  "no": "No",
  "edit": "Edit",
  "password": "Password",
  "password_confirm": "Confirm Password",
  "login": "Log In",
  "forgot_password": "Forgot Password?",
  "remove": "Remove",
  "destroy_confirm_title": "Please Confirm",
  "destroy_confirm": "Are you sure you want to remove this?",
  "logout": "Log Out",
  "back": "Back",
  "save": "Save",
  "update": "Update",
  "forms": {
    "errors": {
      "required": "Please fill this out.",
      "standard": "A server error occurred."
    }
  },
  "cars": {
    "singular": "car",
    "plural": "cars",
    "new": "New Car",
    "edit": "Edit Car"
  }
}
```

Some people like to keep this file as flat as possible and then alphabetize it by key. Here, we have our entity objects defined and translate our pages from there. It's really whatever style you want to use here, just be consistent.

That is our apps configuration, now let's build stuff!

## Start building stuff

We probably want a more bootstrapped layout first. Open `~/layouts/default.vue` and add:

```js
<template>
  <main>
    <no-ssr>
    <notifications group="alerts"
                   position="bottom right">
      <template slot="body" slot-scope="props">
        <b-alert :show="props.item.duration || 3000"
                 dismissible
                 :variant="props.item.type || 'info'"
                 @dismissed="props.item.timer=0">
          <p>{{props.item.text}}</p>
          <b-progress :variant="props.item.type"
                      striped
                      :animated="true"
                      :max="props.item.duration"
                      :value="props.item.timer"
                      height="4px">
          </b-progress>
        </b-alert>
      </template>
    </notifications>
    </no-ssr>
    <b-row no-gutters class="mb-3">
      <b-col class="bg-dark">
        <b-navbar toggleable="md" type="dark" variant="dark">
          <b-navbar-toggle target="nav_collapse"></b-navbar-toggle>
          <b-navbar-brand href="/">My App</b-navbar-brand>
          <b-collapse is-nav id="nav_collapse">
            <b-navbar-nav>
              <b-nav-item class="text-white" :to="'/cars'">{{$t('new_car')}}</b-nav-item>
              <b-nav-item class="text-white" :to="'/admin'" v-if="admin">{{$t('admin')}}</b-nav-item>
            </b-navbar-nav>
            <b-navbar-nav class="ml-auto">
              <b-nav-item class="text-white" :to="'login'" v-if="!$auth.loggedIn">{{$t('login')}}</b-nav-item>
              <b-nav-item class="text-white" @click="logout" v-if="$auth.loggedIn">{{$t('logout')}}</b-nav-item>
            </b-navbar-nav>
          </b-collapse>
        </b-navbar>
      </b-col>
    </b-row>
    <b-container fluid>
      <nuxt/>
    </b-container>
  </main>
</template>
<script>
  export default {
    methods: {
      logout() {
        this.$auth.logout()
      },
      admin() {
        return this.$auth.loggedIn && this.$auth.user.admin
      }
    }
  }
</script>
```

Now we have notifications integrated with bootstrap, showing a countdown timer. Next to that is a pretty bland and basic bootstrap navbar. This is showing that a user has an 'admin' boolean set on it when they login, and that can toggle elements on or off. `nuxt-auth` provides the user object for us when we log in. Toward the bottom you see a `b-container` component holding our special `nuxt` component render tag where our pages will end up.

Notice that if you configure a plugin with `ssr: false` then you also use the `no-ssr` component in the template. Otherwise you get an error from HMR complaining about the server content not matching the rendered content.
