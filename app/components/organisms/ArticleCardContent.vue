<template>
  <div class="area-article-card-content">
    <article-card-content-title :title="article.title" />
    <no-ssr>
      <article-card-content-tags :tags="article.tags"/>
    </no-ssr>
    <article-card-content-data :article="article"/>
    <article-card-content-token-amount :tokenAmount="article.alisToken"/>
  </div>
</template>

<script>
import ArticleCardContentTitle from '../atoms/ArticleCardContentTitle'
import ArticleCardContentTags from '../molecules/ArticleCardContentTags'
import ArticleCardContentData from '../molecules/ArticleCardContentData'
import ArticleCardContentTokenAmount from '../atoms/ArticleCardContentTokenAmount'

export default {
  props: {
    article: {
      type: Object,
      required: true
    }
  },
  components: {
    ArticleCardContentTitle,
    ArticleCardContentTags,
    ArticleCardContentData,
    ArticleCardContentTokenAmount
  }
}
</script>

<style lang="scss" scoped>
.area-article-card-content {
  background: white;
  display: grid;
  grid-area: article-card-content;
  /* prettier-ignore */
  grid-template-areas:
    "title title"
    "tags  tags "
    "...   ... "
    "data  ...  ";
  grid-template-columns: 248px auto;
  grid-template-rows: 48px 42px auto;
  position: relative;
  grid-gap: 4px;
}
</style>
