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

      <select id="vcs" class="select-box" style="width:100px;">
        <option>Chose a git</option>
        <option value="github">github</option>
        <option value="bitbucket">bitbucket</option>
      </select>
    </form>

    <md-button class="md-accent">Primary</md-button>

    <h3 v-if="name && location">{{ name }} is from {{ location }}!</h3>

    <md-table>
      <md-table-row>
        <md-table-head>Name</md-table-head>
        <md-table-head>Language</md-table-head>
        <md-table-head>Last update</md-table-head>
        <md-table-head>Description</md-table-head>
        <md-table-head>Actions</md-table-head>
      </md-table-row>

      <md-table-row v-for="repo in repos">
        <md-table-cell><a :href="repo.html_url">{{ repo.name }}</a></md-table-cell>
        <md-table-cell>{{ repo.language }}</md-table-cell>
        <md-table-cell>{{ repo.updated_at }}</md-table-cell>
        <md-table-cell>{{ repo.description }}</md-table-cell>
        <md-table-cell>
          <a :href="repo.download_url">Download</a>
          <a :href="repo.create_issue_url">Create issue</a>
        </md-table-cell>
      </md-table-row>
    </md-table>
  </div>
</template>

<script>
  import Vue from 'vue'
  import axios from 'axios'
  import VueAxios from 'vue-axios'
  import VueMaterial from 'vue-material'
  import 'vue-material/dist/vue-material.min.css'
  import 'vue-material/dist/theme/black-green-dark.css'

  Vue.use(VueAxios, axios)
  Vue.use(VueMaterial)

  export default {
    name: 'TableBasic',
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
              console.log(response.data);
              this.repos.push({
                description: repo.description,
                download_url: `https://api.github.com/repos/${this.username}/${repo.name}/zipball`,
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
                description: repo.description,
                download_url: `https://bitbucket.org/${this.username}/${repo.name}/get/HEAD.zip`,
                html_url: repo.links.html.href,
                issue: '',
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
