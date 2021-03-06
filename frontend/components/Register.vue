<template>
  <v-card flat>
    <v-form ref="form" v-model="valid">
      <v-container fluid grid-list-lg>
        <v-flex xs12>
          <v-card-title class="flex-column align-start">
            <span class="headline font-weight-bold grey--text text--darken-1"> Register </span>
            <v-icon v-show="modal" class="icon-align--right outlined" small @click="reset"> close </v-icon>
            <div class="subheadline font-weight-thin pt-3">
              <nuxt-link :to="{ name: 'login' }" @click.native="close"> Already have an account? </nuxt-link>
            </div>
          </v-card-title>
          <v-card-text>
            <v-flex xs12>
              <v-text-field
                ref="email"
                v-model="email"
                type="email"
                label="Email"
                :rules="[rules.required, rules.email]"
                :error-messages="emailErrors"
                color="primary"
                required
                @keyup.enter="register"
              />
            </v-flex>
            <v-flex xs12>
              <v-text-field
                v-model="password"
                type="password"
                label="Password"
                :rules="[rules.required, rules.min]"
                color="primary"
                required
                @keyup.enter="register"
              />
            </v-flex>
          </v-card-text>
          <v-card-actions>
            <v-btn class="app" block color="primary" dark large :loading="loading" :disabled="!valid" @click="register">
              Register
            </v-btn>
          </v-card-actions>
        </v-flex>
      </v-container>
    </v-form>
  </v-card>
</template>

<script>
import { mapActions } from 'vuex'

export default {
  props: {
    modal: Boolean,
    close: {
      type: Function,
      default: _ => _,
    },
  },

  data: function () {
    return {
      valid: false,
      email: '',
      password: '',
      rules: {
        required: v => !!v || 'Required',
        email: v =>
          /^(([^<>()\]\\.,;:\s@"]+(\.[^<>()\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/.test(
            v,
          ) || 'Invalid email address',
        min: v => (v ? v.length > 3 || 'Minimum 4 characters' : true),
      },
      emailErrors: [],
      loading: false,
      bearerToken: null,
    }
  },

  watch: {
    async email(v) {
      if (!v) {
        return
      }

      if (this.rules.email(v) === true) {
        await this.$axios
          .get(`/modules/apostrophe-users/check-email/${v}`)
          .then(res => (this.emailErrors = res.data.exists ? ['Email already exists'] : []))
          .catch(() => (this.emailErrors = ['Invalid email address']))
      }
    },
  },

  methods: {
    ...mapActions('snackbar', ['displaySnack']),

    async register() {
      if (this.$refs.form.validate()) {
        this.loading = true
        try {
          await this.$axios.post('/modules/apostrophe-users/register', {
            email: this.email,
            password: this.password,
          })

          this.loading = false
          this.reset()
          this.displaySnack({ message: 'Registration confirmed' })
        } catch (error) {
          this.loading = false
          this.close()
          this.displaySnack({ message: 'Something went wrong', color: 'error' })
        }
      }
    },

    reset() {
      this.close()
      this.$refs.form.reset()
    },
  },
}
</script>
