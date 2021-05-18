<template>
  <div class="card" style="width: 18rem">
    <img
      class="card-img-top"
      width="100%"
      height="220"
      :src="newsItem.urlToImage"
      alt="Card image cap"
    />
    <div class="card-body">
      <h5 v-text="newsItem.title" class="card-title" />
      <p v-text="newsItem.description" class="card-text" />
      <nuxt-link :to="newsItem.title" class="btn btn-primary">
        Читать подробнее
      </nuxt-link>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    newsItem: {
      type: Object,
      required: true,
    },
  },
}
</script>
