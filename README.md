# @storyblok/nuxt-auth
[![npm (scoped with tag)](https://img.shields.io/npm/v/@storyblok/nuxt-auth/latest.svg?style=flat-square)](https://npmjs.com/package/@storyblok/nuxt-auth)
[![npm](https://img.shields.io/npm/dt/@storyblok/nuxt-auth.svg?style=flat-square)](https://npmjs.com/package/@storyblok/nuxt-auth)
[![js-standard-style](https://img.shields.io/badge/code_style-standard-brightgreen.svg?style=flat-square)](http://standardjs.com)

> Storyblok's authentification module for the Nuxt.js

## Requirements

You can create a custom Storyblok app only if you are part of the [partner program](https://www.storyblok.com/partners).

## Setup
- Add `@storyblok/nuxt-auth` dependency using yarn or npm to your project
- Add `@storyblok/nuxt-auth` to `modules` section of `nuxt.config.js`

```js
{
  modules: [
    [
      '@storyblok/nuxt-auth',
      {
        id: 'Client ID from Storyblok App',
        secret: 'Secret from Storyblok App',
        redirect_uri: 'REDIRECT_URI' // Equal to callbakc URL of Oauth2 from Storyblok App
      }
    ],
 ]
}
```

## Recommended Setup

- install `dotenv` module and create `.env` file in root of your project
- keep all the confidential informations in `.env` file
- load confidental information in `nuxt.config.js` using `dotenv` package 

```js
require('dotenv').config()

export default {
  // ...
  modules: [
    '@nuxtjs/dotenv',
    [
      '@storyblok/nuxt-auth',
      {
        id: process.env.CONFIDENTIAL_CLIENT_ID,
        secret: process.env.CONFIDENTIAL_CLIENT_SECRET,
        redirect_uri: process.env.CONFIDENTIAL_CLIENT_REDIRECT_URI
      }
    ]
  ]
  // ...
}
```

```js
// .env file
CONFIDENTIAL_CLIENT_ID="Id from Storyblok App"
CONFIDENTIAL_CLIENT_SECRET="Secret from Storyblok App"
CONFIDENTIAL_CLIENT_REDIRECT_URI="callback url of your app"
```

## Usage

The module registers auth middleware in your Nuxt.js project and router for the StoryblokClient. After that you can use axios in your vue files to get data from Storyblok using the [Management API](https://www.storyblok.com/docs/api/management). **Not all features of Management API supported. Only GET supported in this beta.**

### Using Management API

To use Storyblok Management API, all paths in axios was prefixed with `/auth/`. If you want to get all stories from space ide 606, you would call with management API `spaces/606/stories/` here you call `/auth/spaces/606/stories/`.

For example, to get all stories from specific space:

```js
import axios from 'axios'

export default {
  data() {
    return {
      stories: [],
      perPage: null,
      total: null
    }
  },
  mounted() {
    if (window.top == window.self) {
      // when your app is authenticated,
      // this URL will be redirect to <YOUR_APP_URL>/space_id?<AUTHORIZED_SPACE>
      window.location.assign('https://app.storyblok.com/oauth/app_redirect')
    } else {
      // however, once authenticated and inside of Storyblok Space APP,
      // you're able to use axios to make the necessary requests
      // for example, get the stories from the authenticated space
      this.loadStories()
    }
  },
  methods: {
    loadStories() {
      // get the space id from URL and use it in requests
      axios.get(`/auth/spaces/${this.$route.query.space_id}/stories`)
        .then((res) => {
          // do what you want to do ;) 
          // this is only basic sample
          this.perPage = res.data.perPage
          this.total = res.data.total
          this.stories = res.data.stories
        })
      }
  }
}
```

Another example, to create a new story:

```js
import axios from 'axios'

export default {
  data() {
    return {
      loading: false,
      story: {
        name: ''
      }
    }
  },
  methods: {
    createStory() {
      this.loading = true

      // The request body is the same from Management API
      // https://www.storyblok.com/docs/api/management#core-resources/stories/create-story
      const body = {
        story: { ...this.story }
      }

      // get the space id from URL and use it in requests
      return axios
        .post(`/auth/spaces/${this.$route.query.space_id}/stories`, body)
        .then((res) => {
          this.loading = false
        })
      }
  }
}
```

### Get the authenticated user information

To get information about the authenticated user, you should make a `GET` request to `/auth/user` path.

Example:

```js
import axios from 'axios'

export default {
  data() {
    return {
      user: {}
    }
  },
  mounted() {
    if (window.top == window.self) {
      window.location.assign('https://app.storyblok.com/oauth/app_redirect')
    } else {
      this.loadUserInformation()
    }
  },
  methods: {
    loadUserInformation() {
      axios.get(`/auth/user?space_id=${this.$route.query.space_id}`)
        .then((res) => {
          this.user = res.data || {}
        })
      }
  }
}
```

## Example app

Check out the workflow manager app to see an example [github.com/storyblok/storyblok-workflow-app](https://github.com/storyblok/storyblok-workflow-app).

## License

[MIT License](./LICENSE)

Copyright (c) Storyblok <it@storyblok.com>
