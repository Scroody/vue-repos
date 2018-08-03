<template>
  <div id="app">
    <form @submit.prevent="search">
      <input v-model="username" placeholder="Put your github username here!!"/>
    </form>
    <p v-if="data.name && data.location">
      {{ data.name }} ({{ data.login }})
      is from
      {{ data.location }}!
    </p>
    <p v-else>{{ errorMsg }}</p>
  </div>
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'

Vue.use(VueAxios, axios)

export default {
  data () {
    return {
      username: '',
      data: [],
      errorMsg: ''
    }
  },
  methods: {
    search () {
      const api = `https://api.github.com/users/${this.username}`
      vue.axios.get(api).then(response => {
        this.data = response.data
        console.log(this.data)
      }).catch(error => {
        this.errorMsg = 'No user or location!!'
        this.data = []
        console.log(error)
      })
    }
  }
}
</script>

<style>

</style>
