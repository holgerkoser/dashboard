<!--
SPDX-FileCopyrightText: 2021 SAP SE or an SAP affiliate company and Gardener contributors

SPDX-License-Identifier: Apache-2.0
 -->

 <template>
  <secret-dialog
    :value=value
    :data="secretData"
    :data-valid="valid"
    :secret="secret"
    cloud-provider-kind="gcp"
    create-title="Add new Google Secret"
    replace-title="Replace Google Secret"
    @input="onInput">

    <template v-slot:secret-slot>
      <div>
        <v-textarea
          ref="serviceAccountKey"
          color="primary"
          filled
          v-model="serviceAccountKey"
          label="Service Account Key"
          :error-messages="getErrorMessages('serviceAccountKey')"
          @input="$v.serviceAccountKey.$touch()"
          @blur="$v.serviceAccountKey.$touch()"
          hint="Enter or drop a service account key in JSON format"
          persistent-hint
        ></v-textarea>
      </div>
    </template>
    <template v-slot:help-slot>
      <div class="help-content">
        <p>
          A service account is a special account that can be used by services and applications running on your Google
          Compute Engine instance to interact with other Google Cloud Platform APIs. Applications can use service
          account credentials to authorize themselves to a set of APIs and perform actions within the permissions
          granted to the service account and virtual machine instance.
        </p>

        <p>
          Ensure that the service account has at least the roles below.
          <ul>
            <li>Service Account Admin</li>
            <li>Service Account Token Creator</li>
            <li>Service Account User</li>
            <li>Compute Admin</li>
          </ul>

        </p>

        <p>
          The Service Account has to be enabled for the Google Identity and Access Management API.
        </p>

        <p>
          Read the
          <a href="https://cloud.google.com/compute/docs/access/service-accounts"
              target="_blank" rel="noopener">
            Service Account Documentation<v-icon style="font-size: 80%">mdi-open-in-new</v-icon></a> on how to apply for credentials
          to service accounts.
        </p>
      </div>
    </template>

  </secret-dialog>

</template>

<script>
import SecretDialog from '@/components/dialogs/SecretDialog'
import { required } from 'vuelidate/lib/validators'
import { serviceAccountKey } from '@/utils/validators'
import { handleTextFieldDrop, getValidationErrors, setDelayedInputFocus } from '@/utils'

const validationErrors = {
  serviceAccountKey: {
    required: 'You can\'t leave this empty.',
    serviceAccountKey: 'Not a valid Service Account Key'
  }
}

export default {
  components: {
    SecretDialog
  },
  props: {
    value: {
      type: Boolean,
      required: true
    },
    secret: {
      type: Object
    }
  },
  data () {
    return {
      serviceAccountKey: undefined,
      validationErrors,
      dropHandlerInitialized: false
    }
  },
  validations () {
    // had to move the code to a computed property so that the getValidationErrors method can access it
    return this.validators
  },
  computed: {
    valid () {
      return !this.$v.$invalid
    },
    secretData () {
      return {
        'serviceaccount.json': this.serviceAccountKey
      }
    },
    validators () {
      const validators = {
        serviceAccountKey: {
          required,
          serviceAccountKey
        }
      }
      return validators
    },
    isCreateMode () {
      return !this.secret
    }
  },
  methods: {
    onInput (value) {
      this.$emit('input', value)
    },
    reset () {
      this.$v.$reset()

      this.serviceAccountKey = ''

      if (!this.isCreateMode) {
        setDelayedInputFocus(this, 'serviceAccountKey')
      }
    },
    getErrorMessages (field) {
      return getValidationErrors(this, field)
    },
    initializeDropHandlerOnce () {
      if (this.dropHandlerInitialized) {
        return
      }

      this.dropHandlerInitialized = true
      const onDrop = (value) => {
        this.serviceAccountKey = value
      }
      handleTextFieldDrop(this.$refs.serviceAccountKey, /json/, onDrop)
    }
  },
  watch: {
    value: function (value) {
      if (value) {
        this.reset()

        // Mounted does not guarantee that all child components have also been mounted.
        // In addition, the serviceAccountKey ref is within a slot of a v-dialog, which is by default lazily loaded.
        // We initialize the drop handler once the dialog is shown by watching the `value`.
        // We use $nextTick to make sure the entire view has been rendered
        this.$nextTick(() => {
          this.initializeDropHandlerOnce()
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>

  ::v-deep .v-input__control textarea {
    font-family: monospace;
    font-size: 14px;
  }

    .help-content {
    ul {
      margin-top: 20px;
      margin-bottom: 20px;
      list-style-type: none;
      border-left: 4px solid #318334 !important;
      margin-left: 20px;
      padding-left: 24px;
      li {
        font-weight: 300;
        font-size: 16px;
      }
    }
  }

</style>
