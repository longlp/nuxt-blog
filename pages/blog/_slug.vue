<template>
  <article>
    <nav>
      <ul>
        <li v-for="link of article.toc" :key="link.id">
          <NuxtLink :to="`#${link.id}`">{{ link.text }}</NuxtLink>
        </li>
      </ul>
    </nav>
    <h1>{{ article.title }}</h1>    
    <p>{{ article.description }}</p>
    <p> 
      <span v-for="tag of tags" :key="tag.id"> 
        <NuxtLink :to="`/blog/tag/${tag.slug}`"> 
          <span>{{ tag.name }}</span>
        </NuxtLink>
      </span>
    </p>
    <p>Article last updated: {{ formatDate(article.updatedAt) }}</p>    
    <nuxt-content :document="article" />    
    <author :author="article.author" />    
    <prev-next :prev="prev" :next="next" />
  </article>
</template>


<script>
  export default {
    async asyncData({ $content, params }) {
      const article = await $content('articles', params.slug).fetch()   
      const tagsList = await $content('tags')
      .only(['name', 'slug'])
      .where({ name: { $containsAny: article.tags } })
      .fetch()
      const tags = Object.assign({}, ...tagsList.map((s) => ({ [s.name]: s })))   
      const [prev, next] = await $content('articles')
      .only(['title', 'slug'])
      .sortBy('createdAt', 'asc')
      .surround(params.slug)
      .fetch()

    return {
      article,
      tagsList,
      tags,
      prev,
      next
    }
    },
    methods: {
    formatDate(date) {
      const options = { year: 'numeric', month: 'long', day: 'numeric' }
      return new Date(date).toLocaleDateString('en', options)
    }
 }
  }  
</script>
