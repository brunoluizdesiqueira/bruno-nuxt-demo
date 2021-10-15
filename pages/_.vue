<template>
  <div>
    <component v-if="story.content.component" :key="story.content._uid" :blok="story.content" :is="story.content.component | dashify"></component>
  </div>
</template>

<script>
const loadData = function({api, cacheVersion, errorCallback, version, path}) {
  return api.get(`cdn/stories/${path}`, {
    version: version,
    cv: cacheVersion
  }).then((res) => {
    return res.data
  }).catch((res) => {
    if (!res.response) {
      console.error(res)
      errorCallback({ statusCode: 404, message: 'Failed to receive content form api' })
    } else {
      console.error(res.response.data)
      errorCallback({ statusCode: res.response.status, message: res.response.data })
    }
  })
}
export default {
  data () {
    return {
      story: { content: {} }
    }
  },
  mounted () {
    this.$storybridge(() => {
      const storyblokInstance = new StoryblokBridge({
        customParent: 'https://bruno-nuxt-demo.vercel.app/',
        preventClicks: true,
        resolveRelations: ['feature.author'],
      })

      storyblokInstance.on(['input', 'published', 'change'], (event) => {
        if (event.action == 'input') {
          if (event.story.id === this.story.id) {
            this.story.content = event.story.content
          }
        } else {
          window.location.reload()
        }
      })
    }, (error) => {
      console.error(error)
    })
  },
  asyncData (context) {
    return context.app.$storyapi.get('cdn/stories/home', {
      version: 'draft'
    }).then((res) => {
      return res.data
    }).catch((res) => {
      if (!res.response) {
        console.error(res)
        context.error({ statusCode: 404, message: 'Failed to receive content form api' })
      } else {
        console.error(res.response.data)
        context.error({ statusCode: res.response.status, message: res.response.data })
      }
    })
  }
}
</script>
