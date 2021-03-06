<template>
  <div class="m-register modal-content">
     <SfImage
      src="/assets/images/capitools-uk-logo.png"
      class="capitools-logo"
    />
    <h1 class="text-primary text-center">Register</h1>
    <form @submit.prevent="register" class="form">
      <SfInput
        v-model="email"
        name="email"
        :label="$t('Your email')"
        :required="true"
        :valid="!$v.email.$error"
        :error-message="
          !$v.email.required
            ? $t('Field is required.')
            : $t('Please provide valid e-mail address.')
        "
        class="form__element"
      />
      <SfInput
        v-model="firstName"
        name="first-name"
        :label="$t('First Name')"
        :required="true"
        :valid="!$v.firstName.$error"
        :error-message="$t('Field is required.')"
        class="form__element"
      />
      <SfInput
        v-model="lastName"
        name="last-name"
        :label="$t('Last Name')"
        :required="true"
        :valid="!$v.lastName.$error"
        :error-message="$t('Field is required.')"
        class="form__element"
      />
      <SfInput
        v-model="password"
        name="password"
        :label="$t('Password')"
        :required="true"
        :valid="!$v.password.$error"
        :error-message="$t('Field is required.')"
        type="password"
        class="form__element"
      />
      <SfCheckbox
          v-model="checked"
          class="form__element form__checkbox"
          :label="label"
          :required="required"
          :valid="valid"
          :disabled="$v.email.$invalid" 
          @change="subscribeEmail"
        />
      <SfButton class="sf-button--full-width btn-primary form__submit">
        {{ $t("Create an account") }}
      </SfButton>
    </form>
    <SfButton class="sf-button--text action-button" @click.native="switchElem('login')">
      {{ `${$t("or")} ${$t("login in to your account")}` }}
    </SfButton>
  </div>
</template>

<script>
import i18n from '@vue-storefront/i18n';
import { Logger } from '@vue-storefront/core/lib/logger';
import { required, email } from 'vuelidate/lib/validators';
import { SfInput, SfButton, SfImage, SfCheckbox } from '@storefront-ui/vue';
import { ModalList } from 'theme/store/ui/modals'
import { mapActions } from 'vuex';
import Subscribe from '@vue-storefront/core/modules/newsletter/mixins/Subscribe';

export default {
  name: 'MRegister',
  mixins: [Subscribe],
  components: { SfInput, SfButton, SfImage, SfCheckbox },
  data () {
    return {
      email: '',
      password: '',
      firstName: '',
      lastName: '',
      checked: false, 
      label: "Click to join Mailing List", 
      required: false, 
      valid: true, 
      disabled: false,
    };
  },
  methods: {
    ...mapActions('ui', {
      openModal: 'openModal',
      closeModal: 'closeModal'
    }),
    switchElem (to) {
      this.$v.$reset();
      this.openModal({ name: ModalList.Auth, payload: to })
    },
    register () {
      this.$v.$touch();
      if (this.$v.$invalid) {
        this.$store.dispatch('notification/spawnNotification', {
          type: 'danger',
          message: this.$t('Please fix the validation errors'),
          action1: { label: this.$t('OK') }
        });
        return;
      }
      this.$bus.$emit(
        'notification-progress-start',
        this.$t('Registering the account ...')
      );
      this.$store
        .dispatch('user/register', {
          email: this.email,
          password: this.password,
          firstname: this.firstName,
          lastname: this.lastName
        })
        .then(result => {
          this.$bus.$emit('notification-progress-stop');
          if (result.code !== 200) {
            this.onFailure(result);
          } else {
            this.$store.dispatch('user/login', {
              username: this.email,
              password: this.password
            });
            this.onSuccess(i18n.t('You are logged in!'));
            this.closeModal({ name: ModalList.Auth });
          }
        })
        .catch(err => {
          this.onFailure({
            result:
              'Unexpected authorization error. Check your Network conection.'
          });
          this.$bus.$emit('notification-progress-stop');
          Logger.error(err, 'user')();
        });
    },
    onSuccess (message) {
      this.$store.dispatch('notification/spawnNotification', {
        type: 'success',
        message: message,
        action1: { label: i18n.t('OK') }
      });
    },
    onFailure (result) {
      this.$store.dispatch('notification/spawnNotification', {
        type: 'danger',
        message: i18n.t(result.result),
        action1: { label: i18n.t('OK') }
      });
    },
    subscribeEmail () {
      if(this.checked){
          this.subscribe(this.onSuccesfulSubmission);
      }
    },
    onSuccesfulSubmission (isSubscribed) {
      if (isSubscribed) {
        this.$store.dispatch('notification/spawnNotification', {
          type: 'success',
          message: i18n.t('You have been successfully subscribed to our newsletter!'),
          action1: { label: i18n.t('OK') }
        })
      } 
    }
  },
  validations: {
    email: {
      required,
      email
    },
    password: {
      required
    },
    firstName: {
      required
    },
    lastName: {
      required
    }
  }
}
</script>

<style lang="scss" scoped>
.modal-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.form {
  width: 100%;
  &__element {
    margin: 15px 0;
  }
  &__submit {
    margin: var(--spacer-xl) 0 0 0;
  }
}
.action-button {
  margin: var(--spacer-xl) 0;
}
</style>
