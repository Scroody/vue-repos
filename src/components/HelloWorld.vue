<template>
  <div>
    <form novalidate class="md-layout" @submit.prevent="validateUser">
      <md-card class="md-layout-item">
        <md-card-header>
          <div class="md-title">Users</div>
        </md-card-header>

        <md-card-content>
          <div class="md-layout md-gutter">
            <div class="md-layout-item md-small-size-100">
              <md-field :class="getValidationClass('username')">
                <label for="username">Username</label>
                <md-input name="username" id="username" v-model="form.username" :disabled="sending" />
                <span class="md-error" v-if="!$v.form.username.required">Username is required</span>
              </md-field>
            </div>
          </div>

          <div class="md-layout md-gutter">
            <div class="md-layout-item md-small-size-100">
              <md-field :class="getValidationClass('platform')">
                <label for="platform">Choose a platform</label>
                <md-select name="platform" id="platform" v-model="form.platform" md-dense :disabled="sending">
                  <md-option></md-option>
                  <md-option value="github">GitHub</md-option>
                  <md-option value="bitbucket">Bitbucket</md-option>
                </md-select>
                <span class="md-error">platform is required</span>
              </md-field>
            </div>
          </div>
        </md-card-content>

        <md-progress-bar md-mode="indeterminate" v-if="sending" />

        <md-card-actions>
          <md-button type="submit" class="md-accent" :disabled="sending">Search</md-button>
        </md-card-actions>
      </md-card>

      <md-snackbar :md-active.sync="userFound">{{ form.username }} found</md-snackbar>
    </form>

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
        <md-table-cell>
          <a class="md-accent" :href="repo.html_url" target="_blank">{{ repo.name }}</a>
        </md-table-cell>
        <md-table-cell>{{ repo.language }}</md-table-cell>
        <md-table-cell>{{ repo.updated_at }}</md-table-cell>
        <md-table-cell>{{ repo.description }}</md-table-cell>
        <md-table-cell>
          <!-- <a class="md-accent" :href="repo.create_issue_url"><md-icon>note_add</md-icon></a> -->
          <a class="md-accent" :href="repo.download_url"><md-icon>get_app</md-icon></a>
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

  import { validationMixin } from 'vuelidate'
  import {
   required,
   email,
   minLength,
   maxLength
  } from 'vuelidate/lib/validators'

  Vue.use(VueAxios, axios)
  Vue.use(VueMaterial)

  export default {
    name: 'FormValidation',
    mixins: [validationMixin],
    data: () => ({
      form: {
        username: null,
        platform: null
      },
      userFound: false,
      sending: false,
      location: null,
      name: null,
      repos: []
    }),
    validations: {
      form: {
        username: {
          required
        },
        platform: {
          required
        }
      }
    },
    methods: {
      getValidationClass (fieldName) {
        const field = this.$v.form[fieldName]

        if (field) {
          return {
            'md-invalid': field.$invalid && field.$dirty
          }
        }
      },
      clearForm () {
        this.$v.$reset()
        this.form.username = null
        this.form.platform = null
      },
      validateUser () {
        this.$v.$touch()

        if (!this.$v.$invalid) {
          this.searchUser()
        }
      },
      searchUser () {
        this.repos = [];

        if (this.form.platform === 'github') {
          Vue.axios.get(`https://api.github.com/users/${this.form.username}`)
          .then(response => {
            this.location = response.data.location;
            this.name = response.data.name;
          })

          Vue.axios.get(`https://api.github.com/users/${this.form.username}/repos`)
          .then(response => {
            for (let repo of response.data) {
              this.repos.push({
                description: repo.description,
                download_url: `https://api.github.com/repos/${this.form.username}/${repo.name}/zipball`,
                html_url: repo.html_url,
                language: repo.language,
                name: repo.name,
                updated_at: new Date(repo.updated_at).toLocaleDateString('pt-BR')
              });
            }
          })
        } else if (this.form.platform === 'bitbucket') {
          Vue.axios.get(`https://api.bitbucket.org/2.0/users/${this.form.username}`)
          .then(response => {
            this.location = "Couldn't retrieve location";
            this.name = response.data.display_name;
          })

          Vue.axios.get(`https://api.bitbucket.org/2.0/repositories/${this.form.username}`)
          .then(response => {
            for (let repo of response.data.values) {
              this.repos.push({
                description: repo.description,
                download_url: `https://bitbucket.org/${this.form.username}/${repo.name}/get/HEAD.zip`,
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

<style>
  .md-progress-bar {
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
  }
</style>
