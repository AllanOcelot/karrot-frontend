<template>
  <div
    class="edit-box bg-primary splash-md"
  >
    <div
      class="white-box shadow-6 q-py-md q-px-sm"
      v-if="hasInvalidToken">
      <h1><QIcon name="fas fa-sad-tear" /> {{ $t('GLOBAL.INVALID_LINK') }}</h1>
    </div>
    <div
      class="white-box shadow-6 q-py-md q-px-sm"
      v-else-if="hasSuccess">
      <h1><QIcon name="fas fa-smile-beam" /> {{ $t('UNSUBSCRIBE.SUCCESS') }}</h1>
    </div>
    <form
      v-else
      @submit.prevent="save">
      <div class="white-box shadow-6 q-py-md q-px-sm">
        <QOptionGroup
          class="q-ma-sm"
          type="radio"
          v-model="choice"
          :options="options"
        />
      </div>
      <div
        v-if="hasError"
        class="error"
      >
        <i class="fas fa-exclamation-triangle" />
        {{ $t('GLOBAL.GENERIC_ERROR') }}
      </div>
      <div class="actions">
        <QBtn
          type="submit"
          class="submit shadow-4"
          flat
        >
          {{ $t('BUTTON.UNSUBSCRIBE') }}
        </QBtn>
      </div>
    </form>
  </div>
</template>
<script>
import { QOptionGroup, QIcon } from 'quasar-framework'
import api from '@/unsubscribe/api/unsubscribe'
import QBtn from 'quasar-framework/src/components/btn/QBtn'
import { parseToken } from '@/unsubscribe/utils'

export default {
  components: {
    QBtn,
    QOptionGroup,
    QIcon,
  },
  data () {
    return {
      hasInvalidToken: false,
      hasError: false,
      hasSuccess: false,
      token: this.$router.currentRoute.params.token,
      choice: null,
    }
  },
  methods: {
    async save () {
      this.hasError = false
      try {
        await api.unsubscribe(this.token, { choice: this.choice })
        this.hasSuccess = true
      }
      catch (err) {
        this.hasError = true
      }
    },
    parseToken (token) {
      try {
        return parseToken(token)
      }
      catch (err) {
        this.hasInvalidToken = true
      }
    },
    ensureDefaultChoice () {
      if (this.choice || this.options.length === 0) return
      this.choice = this.options[0].value
    },
  },
  mounted () {
    this.ensureDefaultChoice()
  },
  computed: {
    tokenData () {
      return this.token ? this.parseToken(this.token) : {}
    },
    options () {
      const options = []
      if (this.tokenData.conversationId) {
        options.push({
          label: this.$t('UNSUBSCRIBE.FROM_CONVERSATION'),
          value: 'conversation',
          color: 'secondary',
        })
      }
      else if (this.tokenData.threadId) {
        options.push({
          label: this.$t('UNSUBSCRIBE.FROM_THREAD'),
          value: 'thread',
          color: 'secondary',
        })
      }
      if (this.tokenData.notificationType) {
        const notificationType = this.$t(`GROUP.NOTIFICATION_TYPES.${this.tokenData.notificationType}.NAME`)
        options.push({
          label: this.$t('UNSUBSCRIBE.FROM_NOTIFICATION_TYPE', { notificationType }),
          value: 'notification_type',
          color: 'amber',
        })
      }
      if (this.tokenData.groupId) {
        options.push({
          label: this.$t('UNSUBSCRIBE.FROM_GROUP', { groupName: this.tokenData.groupName }),
          value: 'group',
          color: 'red',
        })
      }
      return options
    },
  },
}
</script>

<style scoped lang="stylus">
  .edit-box
    color #0c0c0c
</style>
