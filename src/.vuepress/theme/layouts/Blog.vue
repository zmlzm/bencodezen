<template>
  <div class="blog">
    <div class="blog__header">
      <p class="publish-date"><time :datetime="$frontmatter.date">{{ publishDate }}</time></p>
      <h1 class="blog__title">{{ $page.title }}</h1>
    </div>

    <Content custom />

    <section class="share">
      <h2>Share</h2>
      <a class="share__button" 
        :href="`https://twitter.com/intent/tweet?text=${urlPostTitle} by @bencodezen ${$themeConfig.domain}${$page.path}`"
        target="_blank"
      >
        <i class="fab fa-twitter"></i> Tweet
      </a>
    </section>

    <div class="page-edit">
      <div
        class="edit-link"
        v-if="editLink"
      >
        <a
          href="https://github.com/bencodezen/bencodezen/issues/new"
          target="_blank"
          rel="noopener noreferrer"
        >
          {{ editLinkText }} <OutboundLink />
        </a>
      </div>
      <div
        class="last-updated"
        v-if="lastUpdated"
      >
        <span class="prefix">{{ lastUpdatedText }}: </span>
        <time class="time" :datetime="$page.lastUpdated">{{ lastUpdated }}</time>
      </div>
    </div>

    <div class="page-nav" v-if="prev || next">
      <p class="inner">
        <span
          v-if="prev"
          class="prev"
        >
          ←
          <router-link
            v-if="prev"
            class="prev"
            :to="prev.path"
          >
            {{ prev.title || prev.path }}
          </router-link>
        </span>

        <span
          v-if="next"
          class="next"
        >
          <router-link
            v-if="next"
            :to="next.path"
          >
            {{ next.title || next.path }}
          </router-link>
          →
        </span>
      </p>
    </div>

    <slot name="bottom"/>
  </div>
</template>

<script>
import { resolvePage, normalize, outboundRE, endingSlashRE } from '../mixins/util'

export default {
  name: 'Blog',

  props: ['sidebarItems'],

  computed: {
    lastUpdated () {
      if (this.$page.lastUpdated) {
        const dateFormat = new Date(this.$page.lastUpdated)

        const options = {
            year: 'numeric',
            month: 'long',
            day: 'numeric'
        } 
        
        return `${dateFormat.toLocaleDateString(this.$lang, options)}, ${dateFormat.toLocaleTimeString(this.$lang)}`
      }
    },

    lastUpdatedText () {
      if (typeof this.$themeLocaleConfig.lastUpdated === 'string') {
        return this.$themeLocaleConfig.lastUpdated
      }
      if (typeof this.$site.themeConfig.lastUpdated === 'string') {
        return this.$site.themeConfig.lastUpdated
      }
      return 'Last Updated'
    },

    prev () {
      const prev = this.$page.frontmatter.prev
      if (prev === false) {
        return
      } else if (prev) {
        return resolvePage(this.$site.pages, prev, this.$route.path)
      } else {
        return resolvePrev(this.$page, this.sidebarItems)
      }
    },

    next () {
      const next = this.$page.frontmatter.next
      if (next === false) {
        return
      } else if (next) {
        return resolvePage(this.$site.pages, next, this.$route.path)
      } else {
        return resolveNext(this.$page, this.sidebarItems)
      }
    },

    editLink () {
      if (this.$page.frontmatter.editLink === false) {
        return
      }
      const {
        repo,
        editLinks,
        docsDir = '',
        docsBranch = 'master',
        docsRepo = repo
      } = this.$site.themeConfig

      let path = normalize(this.$page.path)
      if (endingSlashRE.test(path)) {
        path += 'README.md'
      } else {
        path += '.md'
      }
      if (docsRepo && editLinks) {
        return this.createEditLink(repo, docsRepo, docsDir, docsBranch, path)
      }
    },

    editLinkText () {
      return (
        this.$themeLocaleConfig.editLinkText ||
        this.$site.themeConfig.editLinkText ||
        `Edit this page`
      )
    },

    publishDate() {
        const dateFormat = new Date(this.$frontmatter.date)
        const options = {
            year: 'numeric',
            month: 'long',
            day: 'numeric'
        } 
        
        return dateFormat.toLocaleDateString(this.$lang, options)
    },

    urlPostTitle() {
      return encodeURIComponent(this.$page.title)
    }
  },

  methods: {
    createEditLink (repo, docsRepo, docsDir, docsBranch, path) {
      const bitbucket = /bitbucket.org/
      if (bitbucket.test(repo)) {
        const base = outboundRE.test(docsRepo)
          ? docsRepo
          : repo
        return (
          base.replace(endingSlashRE, '') +
           `/${docsBranch}` +
           (docsDir ? '/' + docsDir.replace(endingSlashRE, '') : '') +
           path +
           `?mode=edit&spa=0&at=${docsBranch}&fileviewer=file-view-default`
        )
      }

      const base = outboundRE.test(docsRepo)
        ? docsRepo
        : `https://github.com/${docsRepo}`

      return (
        base.replace(endingSlashRE, '') +
        `/edit/${docsBranch}` +
        (docsDir ? '/' + docsDir.replace(endingSlashRE, '') : '') +
        path
      )
    }
  },

  mounted() {
    let tweets = document.querySelectorAll('.twitter-tweet')

    if (tweets && tweets.length > 0) {
      tweets.forEach(tweet => {
        let id = tweet.dataset.twitterId
        twttr.widgets.createTweet(id, tweet)
        tweet.setAttribute('style', 'border: 0; padding: 0; margin-right: 0;')
        tweet.children[0].setAttribute('style', 'display: none;')
      })
    }
  }
}

function resolvePrev (page, items) {
  return find(page, items, -1)
}

function resolveNext (page, items) {
  return find(page, items, 1)
}

function find (page, items, offset) {
  const res = []
  items.forEach(item => {
    if (item.type === 'group') {
      res.push(...item.children || [])
    } else {
      res.push(item)
    }
  })
  for (let i = 0; i < res.length; i++) {
    const cur = res[i]
    if (cur.type === 'page' && cur.path === page.path) {
      return res[i + offset]
    }
  }
}
</script>

<style lang="stylus">
@import '../styles/config.styl'
@require '../styles/wrapper.styl'

.blog {
  @extend $wrapper
}

.blog__header {
  padding-top: 4.6rem;
}

.blog__title {
  margin-top: 0;
}

.publish-date {
  margin-bottom: 0.5rem;
  font-family: 'Poppins';
}

.page-edit
  @extend $wrapper
  padding-top 1rem
  padding-bottom 1rem
  padding-left 0
  padding-right 0
  overflow auto
  .edit-link
    display inline-block
    a
      color lighten($textColor, 25%)
      margin-right 0.25rem
  .last-updated
    float right
    font-size 0.9em
    .prefix
      font-weight 500
      color lighten($textColor, 25%)
    .time
      font-weight 400
      color #aaa

.page-nav
  padding-top 1rem
  padding-bottom 0
  .inner
    min-height 2rem
    margin-top 0
    border-top 1px solid $borderColor
    padding-top 1rem
    overflow auto // clear float
  .next
    float right

.share {
  margin-bottom: 2rem;

  &__button {
    padding: 0.5rem 1rem;
    border: 1px solid $accentColor;
    border-radius: 10px;
    background-color: $accentColor;
    color: #fff;
    transition: 0.2s ease-in background-color,
      0.2s ease-in color;

    &:hover {
      background-color: #fff;
      color: $accentColor;
    }
  }
}

.twitter-tweet-rendered {
  margin: 10px auto;
}

@media (max-width: $MQMobile)
  .page-edit
    .edit-link
      margin-bottom .5rem
    .last-updated
      font-size .8em
      float none
      text-align left

@media (max-width: $MQMobileNarrow) {
  .blog__title {
    font-size: 2.441rem;
  }
}
</style>
