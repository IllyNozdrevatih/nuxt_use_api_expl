<template>
  <div class="container">
    <div class="py-15" v-if="article !== null">
      <h1 v-text="article.title" class="card-title" />
      <div class="row">
        <div class="col-lg-4">
          <ul>
            <li>
              <div class="font-weight-bold">Автор</div>
              <div v-text="article.author" />
            </li>
            <li>
              <div class="font-weight-bold">Ссылка</div>
              <div v-text="article.url" />
            </li>
            <li>
              <div class="font-weight-bold">Источник</div>
              <div v-text="article.source.name" />
            </li>
            <li>
              <div class="font-weight-bold">Дата публикации</div>
              <div v-text="article.publishedAt" />
            </li>
          </ul>
        </div>
        <div class="col-lg-4">
          <p v-text="article.description" class="card-text" />
        </div>
        <div class="col-lg-4">
          <ul v-for="(articleItem, index) of articles" :key="index">
            <li v-text="articleItem.title"></li>
          </ul>
        </div>
      </div>
      <img
        class="card-img-top"
        width="100%"
        :src="article.urlToImage"
        alt="Card image cap"
      />
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      article: null,
      articles: [],
    }
  },
  async fetch() {
    const { articles } = await this.$axios.$get(
      `https://newsapi.org/v2/everything?q=${this.$route.params.slug}&apiKey=6966cb6442894918a9fbd9104de4abad`
    )

    if (articles.length) {
      this.article = articles[0]
    }
    const res = await this.$axios.$get(
      `https://newsapi.org/v2/everything?domains=bbc.co.uk,techcrunch.com,engadget.com&q=apple&page=1&pageSize=10&sortBy=publishedAt&language=en&apiKey=6966cb6442894918a9fbd9104de4abad`
    )
    if (res?.articles.length) {
      this.articles = res.articles
    }
  },

  head() {
    return {
      title: this.article?.title ? this.article.title : 'Страница',
    }
  },
}
</script>
