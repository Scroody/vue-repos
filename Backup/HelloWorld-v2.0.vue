<template>
  <div>
    <div class="error-notification" v-if="errorMsg">
      <h3>{{ errorMsg }}</h3>
    </div>

    <form @submit.prevent="search">
      <div class="group">
        <input v-model="username" required>
        <label>Username</label>
      </div>

      <select id="vcs">
        <option>Chose a git</option>
        <option value="github">github</option>
        <option value="bitbucket">bitbucket</option>
      </select>
    </form>

    <h3 v-if="name && location">{{ name }} is from {{ location }}!</h3>

    <table id="repos" v-if="repos">
      <thead>
        <tr>
          <th>Name</th>
          <th>Language</th>
          <th>Last updated</th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="repo in repos">
          <td><a :href="repo.html_url">{{ repo.name }}</a></td>
          <td>{{ repo.language }}</td>
          <td>{{ repo.updated_at }}</td>
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
        repos: [],
        name: '',
        location: '',
        username: ''
      }
    },
    methods: {
      search () {
        const vcsValue = document.getElementById('vcs').value;
        this.repos = [];

        if (vcsValue === 'github') {
          Vue.axios.get(`https://api.github.com/users/${this.username}`)
          .then(response => {
            this.location = response.data.location;
            this.name = response.data.name;
          })
          .catch(error => this.errorMsg = 'User not found');

          Vue.axios.get(`https://api.github.com/users/${this.username}/repos`)
          .then(response => {
            for (let repo of response.data) {
              this.repos.push({
                html_url: repo.html_url,
                language: repo.language,
                name: repo.name,
                updated_at: new Date(repo.updated_at).toLocaleDateString('pt-BR')
              });
            }
          })
          .catch(error => this.errorMsg = 'No repos found');
        } else if (vcsValue === 'bitbucket') {
          Vue.axios.get(`https://api.bitbucket.org/2.0/users/${this.username}`)
          .then(response => {
            this.location = "Couldn't retrieve location";
            this.name = response.data.display_name;
          })
          .catch(error => this.errorMsg = 'User not found');

          Vue.axios.get(`https://api.bitbucket.org/2.0/repositories/${this.username}`)
          .then(response => {
            for (let repo of response.data.values) {
              this.repos.push({
                html_url: repo.links.html.href,
                language: repo.language,
                name: repo.name,
                updated_at: new Date(repo.updated_on).toLocaleDateString('pt-BR')
              });
            }
          })
        }
      }
    }
  }
</script>
