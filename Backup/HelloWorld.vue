<template>
  <div>
    <form @submit.prevent="search">
      <div class="group">
        <input v-model="username" required>
        <label>Username</label>
      </div>
    </form>

    <h3 v-if="data.name && data.location">
      {{ data.name }} ({{ data.login }})
      is from
      {{ data.location }}!
    </h3>

    <small v-else>{{ errorMsg }}</small>
    <small v-else>{{ errorMsg2 }}</small>

    <table id="repos">
      <thead>
        <tr>
          <th>Name</th>
          <th>Language</th>
          <th class="align-right">Open issues</th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="repo in repos">
          <td><a :href="repo.html_url">{{ repo.name }}</a></td>
          <td>{{ repo.language }}</td>
          <td class="align-right">{{ repo.open_issues }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import Vue from 'vue';
import axios from 'axios';
import VueAxios from 'vue-axios';

Vue.use(VueAxios, axios);

export default {
  data () {
    return {
      errorMsg: '',
      errorMsg2: '',
      data: [],
      repos: [],
      username: ''
    }
  },
  methods: {
    search () {
      vue.axios.get("https://api.github.com/users/" + this.username)
      .then(response => this.data = response.data)
      .catch(error => this.errorMsg = 'No user found!!');

      vue.axios.get("https://api.github.com/users/" + this.username + "/repos")
      .then(response => this.repos = response.data)
      .catch(error => this.errorMsg2 ='This user has no repos!!');
    }
  }
}
</script>

<style>

</style>
