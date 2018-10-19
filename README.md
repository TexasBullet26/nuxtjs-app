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
