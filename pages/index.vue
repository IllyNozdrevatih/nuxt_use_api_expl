<template>
  <div class="container">
    <div class="py-15">
      <div class="form-group">
        <input
          v-model="filters.qInTitle"
          @input="updateFilter"
          type="text"
          class="form-control"
          placeholder="Поиск..."
        />
      </div>
    </div>
    <div class="row">
      <div class="col-lg-2 order-lg-0 order-0">
        <div class="form-group">
          <label for="domains">Выбор источника</label>
          <Multiselect
            v-model="filters.domains"
            :options="domainsVariants"
            :multiple="true"
            :taggable="true"
            @input="updateFilter"
            @remove="updateFilter"
            @tag="addTag"
            selectLabel="Нажмите enter для выбора"
            selectedLabel="Выбрано"
            deselectLabel="Нажмите enter для удаления"
            placeholder="Выберите источник"
            id="domains"
          >
          </Multiselect>
        </div>
      </div>
      <div class="col-lg-8 order-lg-1 order-2">
        <div v-if="newsList.length" class="row">
          <div
            v-for="(newsItem, index) of newsList"
            :key="index"
            class="col-lg-6 col-md-6 col-sm-12 mb-15"
          >
            <NewsCard class="h-100 w-100" :newsItem="newsItem" />
          </div>
        </div>
        <div class="text-center" v-else>
          <div class="h2">Нет публикаций по заданным фильтрам</div>
          <div v-if="hasError">
            Поиск и Выбор источника обязательны (одно из выбрать)
          </div>
        </div>
      </div>
      <div class="col-lg-2 order-lg-2 order-1">
        <div class="form-group">
          <label for="lang">Выбор языка</label>
          <select
            v-model="filters.language"
            @change="updateFilter"
            class="form-control"
            id="lang"
          >
            <option value="es">Испанский</option>
            <option value="en">Английский</option>
          </select>
        </div>
        <div class="form-group">
          <label for="sortBy">Сортировка по</label>
          <select
            v-model="filters.sortBy"
            @change="updateFilter"
            class="form-control"
            id="sortBy"
          >
            <option value="publishedAt">Дате публикации</option>
            <option value="popularity">Популярности</option>
            <option value="relevancy">Актуальности</option>
          </select>
        </div>
        <div class="form-group">
          <label for="pageSize">Сортировка по</label>
          <select
            v-model="pagination.pageSize"
            @change="updateFilter"
            class="form-control"
            id="pageSize"
          >
            <option>6</option>
            <option>10</option>
            <option>14</option>
          </select>
        </div>
      </div>
    </div>
    <div v-if="newsList.length > 0" class="d-flex justify-content-center">
      <button @click="uploadNews(true)" class="btn btn-primary">
        Загрузить еще
      </button>
    </div>
  </div>
</template>

<script>
import { Multiselect } from 'vue-multiselect'
import NewsCard from '~/components/news/NewsCard'

export default {
  components: { NewsCard, Multiselect },
  data() {
    return {
      domainsVariants: ['bbc.co.uk', 'techcrunch.com', 'engadget.com'],
      filters: {
        language: 'en',
        domains: ['bbc.co.uk', 'techcrunch.com', 'engadget.com'],
        sortBy: 'publishedAt',
        qInTitle: 'apple',
      },
      newsList: [],
      pagination: {
        page: 1,
        pageSize: 6,
      },
    }
  },
  async fetch() {
    if (Object.keys(this.$route.query).length) {
      const domains = this.$route.query.domains.split(',')
      this.pagination.page = this.$route.query.page
      this.pagination.pageSize = this.$route.query.pageSize
      this.filters.language = this.$route.query.language
      this.filters.sortBy = this.$route.query.sortBy
      this.filters.qInTitle = this.$route.query.qInTitle
      this.filters.domains = domains
    }
    await this.uploadNews()
  },

  head() {
    return {
      title: 'Главная',
    }
  },

  methods: {
    async uploadNews(loadMore = false) {
      try {
        if (loadMore) {
          this.pagination.page = Number.parseInt(this.pagination.page) + 1
          this.updateQuery(this.pagination.page)
        }
        const arrFindBy = []
        if (this.filters.domains.length > 0) {
          const domainsString = this.filters.domains.join(',')
          arrFindBy.push(`domains=${domainsString}`)
        }
        if (this.filters.qInTitle) {
          arrFindBy.push(`q=${this.filters.qInTitle}`)
        }

        const findBy = arrFindBy.join('&')
        const { articles } = await this.$axios.$get(
          `https://newsapi.org/v2/everything?${findBy}&page=${
            this.pagination.page
          }&pageSize=${Number(this.pagination.pageSize)}&sortBy=${
            this.filters.sortBy
          }&language=${
            this.filters.language
          }&apiKey=6966cb6442894918a9fbd9104de4abad`
        )
        if (loadMore) {
          this.newsList.push(...articles)
        } else {
          this.newsList = articles
        }
      } catch (e) {
        this.newsList = []
        console.log(e)
      }
    },

    addTag(newTag) {
      const tag = {
        name: newTag,
        code: newTag.substring(0, 2) + Math.floor(Math.random() * 10000000),
      }
      this.domainsVariants.push(tag)
      this.filters.domains.push(tag)
    },

    updateFilter() {
      this.updateQuery()
      this.uploadNews()
    },
    updateQuery(page = 1) {
      let domainsString = ''
      if (this.filters.domains.length) {
        domainsString = this.filters.domains.join(',')
      }
      this.$router.push({
        query: {
          page,
          sortBy: this.filters.sortBy,
          pageSize: this.pagination.pageSize,
          language: this.filters.language,
          qInTitle: this.filters.qInTitle,
          domains: domainsString,
          q: this.filters.qInTitle,
        },
      })
    },
  },

  computed: {
    hasError() {
      return (
        this.filters.qInTitle.length === 0 && this.filters.domains.length === 0
      )
    },
  },
}
</script>

<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
