<template>
  <div class="edit-article-container">
    <app-header showEditHeaderNav showPostArticleLink class="logo-original"/>
    <article-editor :title="title"/>
  </div>
</template>

<script>
import { debounce } from 'lodash'
import { mapGetters, mapActions } from 'vuex'
import AppHeader from '../organisms/AppHeader'
import ArticleEditor from '../atoms/ArticleEditor'

export default {
  components: {
    AppHeader,
    ArticleEditor
  },
  computed: {
    ...mapGetters('article', ['title', 'body', 'isSaving'])
  },
  methods: {
    ...mapActions('article', ['putDraftArticle', 'setIsSaving', 'setIsSaved', 'gotArticleData']),
    putArticle: debounce(async function() {
      const article = {
        title: this.title === '' ? ' ' : this.title,
        body: this.body.replace(/<div class="medium-insert-buttons"[\s\S]*/, '') + ' '
      }
      const { articleId } = this.$route.params
      this.setIsSaving({ isSaving: true })
      try {
        if (this.isSaving) await this.putDraftArticle({ article, articleId })
        this.setIsSaved({ isSaved: true })
      } catch (e) {
        console.error(e)
      }
    }, 2500)
  },
  watch: {
    title(newTitle, oldTitle) {
      if (this.gotArticleData) {
        this.setIsSaved({ isSaved: false })
        this.setIsSaving({ isSaving: false })
        this.putArticle()
      }
    },
    body(newBody, oldBody) {
      if (this.gotArticleData) {
        this.setIsSaved({ isSaved: false })
        this.setIsSaving({ isSaving: false })
        this.putArticle()
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.edit-article-container {
  display: grid;
  grid-template-rows: 100px 50px 650px 75px;
  grid-template-columns: 1fr 640px 1fr;
  /* prettier-ignore */
  grid-template-areas:
    "app-header  app-header app-header"
    "...         ...        ...       "
    "...         editor     ...       "
    "...         ...        ...       ";
}
</style>
