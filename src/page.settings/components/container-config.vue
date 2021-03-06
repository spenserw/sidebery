<template lang="pug">
.PanelConfig(@wheel="onWheel")
  TextInput.title(
    ref="name"
    v-debounce.250="updateName"
    :value="conf.name"
    :or="t('container_dashboard.name_placeholder')"
    @input="onNameInput")

  SelectField.-no-separator(
    label="container_dashboard.icon_label"
    :value="icon"
    :opts="iconOpts"
    :color="color"
    @input="updateIcon")

  SelectField(
    label="container_dashboard.color_label"
    :value="color"
    :opts="colorOpts"
    :icon="icon"
    @input="updateColor")

  ToggleField(
    label="container_dashboard.rules_include"
    :title="t('container_dashboard.rules_include_tooltip')"
    :value="conf.includeHostsActive"
    @input="toggleIncludeHosts")
  .sub-fields.-nosep(v-if="conf.includeHostsActive")
    .field
      TextInput.text(
        ref="includeHostsInput"
        or="---"
        :value="conf.includeHosts"
        :valid="includeHostsValid"
        @input="onIncludeHostsInput")

  ToggleField(
    label="container_dashboard.rules_exclude"
    :title="t('container_dashboard.rules_exclude_tooltip')"
    :value="conf.excludeHostsActive"
    @input="toggleExcludeHosts")
  .sub-fields.-nosep(v-if="conf.excludeHostsActive")
    .field
      TextInput.text(
        ref="excludeHostsInput"
        or="---"
        :value="conf.excludeHosts"
        :valid="excludeHostsValid"
        @input="onExcludeHostsInput")

  SelectField(
    label="container_dashboard.proxy_label"
    optLabel="container_dashboard.proxy_"
    noneOpt="direct"
    :value="proxied"
    :opts="proxyOpts"
    @input="switchProxy")
  .sub-fields(v-if="proxied !== 'direct'")
    TextField(
      ref="proxyHost"
      label="Host"
      :or="t('container_dashboard.proxy_host_placeholder')"
      :line="true"
      :value="proxyHost"
      :valid="proxyHostValid"
      @input="onProxyHostInput"
      @keydown="onFieldKeydown($event, 'proxyPort', 'name')")
    TextField(
      ref="proxyPort"
      label="Port"
      :or="t('container_dashboard.proxy_port_placeholder')"
      :line="true"
      :value="proxyPort"
      :valid="proxyPortValid"
      @input="onProxyPortInput"
      @keydown="onFieldKeydown($event, 'proxyUsername', 'proxyHost')")
    TextField(
      ref="proxyUsername"
      label="Username"
      :or="t('container_dashboard.proxy_username_placeholder')"
      :line="true"
      :value="proxyUsername"
      @input="onProxyUsernameInput"
      @keydown="onFieldKeydown($event, 'proxyPassword', 'proxyPort')")
    TextField(
      ref="proxyPassword"
      label="Password"
      :or="t('container_dashboard.proxy_password_placeholder')"
      :line="true"
      :value="proxyPassword"
      :password="true"
      @input="onProxyPasswordInput"
      @keydown="onFieldKeydown($event, null, 'proxyUsername')")
    ToggleField(
      v-if="isSomeSocks"
      label="container_dashboard.proxy_dns_label"
      :value="proxyDNS"
      @input="toggleProxyDns")

  ToggleField(
    label="container_dashboard.user_agent"
    :value="conf.userAgentActive"
    @input="toggleUserAgent")
  .sub-fields.-nosep(v-if="conf.userAgentActive")
    .field
      TextInput.text(
        ref="userAgentInput"
        or="---"
        :value="conf.userAgent"
        @input="onUserAgentInput")
</template>

<script>
import TextInput from '../../components/text-input'
import ToggleField from '../../components/toggle-field'
import SelectField from '../../components/select-field'
import TextField from '../../components/text-field'
import State from '../store/state'
import Actions from '../actions'

const HOSTS_RULE_RE = /^.+$/m
const PROXY_HOST_RE = /^.{3,65536}$/
const PROXY_PORT_RE = /^\d{2,5}$/

export default {
  name: 'PanelConfig',

  components: {
    TextInput,
    TextField,
    ToggleField,
    SelectField,
  },

  props: {
    conf: {
      type: Object,
      default: () => ({}),
    },
    index: Number,
  },

  data() {
    return {
      iconOpts: [
        { value: 'fingerprint', icon: 'fingerprint' },
        { value: 'briefcase', icon: 'briefcase' },
        { value: 'dollar', icon: 'dollar' },
        { value: 'cart', icon: 'cart' },
        { value: 'circle', icon: 'circle' },
        { value: 'gift', icon: 'gift' },
        { value: 'vacation', icon: 'vacation' },
        { value: 'food', icon: 'food' },
        { value: 'fruit', icon: 'fruit' },
        { value: 'pet', icon: 'pet' },
        { value: 'tree', icon: 'tree' },
        { value: 'chill', icon: 'chill' },
        { value: 'fence', icon: 'fence' },
      ],
      colorOpts: [
        { value: 'blue', color: 'blue' },
        { value: 'turquoise', color: 'turquoise' },
        { value: 'green', color: 'green' },
        { value: 'yellow', color: 'yellow' },
        { value: 'orange', color: 'orange' },
        { value: 'red', color: 'red' },
        { value: 'pink', color: 'pink' },
        { value: 'purple', color: 'purple' },
        { value: 'toolbar', color: 'toolbar' },
      ],
      proxyOpts: ['http', 'https', 'socks4', 'socks', 'direct'],
    }
  },

  computed: {
    name() {
      return this.conf.name || ''
    },

    icon() {
      return this.conf.icon || 'fingerprint'
    },

    color() {
      return this.conf.color || 'blue'
    },

    proxied() {
      if (!this.conf.proxy) return 'direct'
      return this.conf.proxy.type
    },

    isSomeSocks() {
      return this.proxied === 'socks' || this.proxied === 'socks4'
    },

    includeHostsValid() {
      if (!this.conf.includeHosts) return ''
      if (HOSTS_RULE_RE.test(this.conf.includeHosts)) return 'valid'
      else return 'invalid'
    },

    excludeHostsValid() {
      if (!this.conf.excludeHosts) return ''
      if (HOSTS_RULE_RE.test(this.conf.excludeHosts)) return 'valid'
      else return 'invalid'
    },

    proxyHost() {
      if (!this.conf.id || !this.conf.proxy) return ''
      return this.conf.proxy.host
    },

    proxyPort() {
      if (!this.conf.id || !this.conf.proxy) return ''
      return this.conf.proxy.port
    },

    proxyHostValid() {
      if (!this.proxyHost) return ''
      if (PROXY_HOST_RE.test(this.proxyHost)) return 'valid'
      else return 'invalid'
    },

    proxyPortValid() {
      if (!this.proxyPort) return ''
      if (PROXY_PORT_RE.test(this.proxyPort)) return 'valid'
      else return 'invalid'
    },

    proxyUsername() {
      if (!this.conf.id || !this.conf.proxy) return ''
      return this.conf.proxy.username
    },

    proxyPassword() {
      if (!this.conf.id || !this.conf.proxy) return ''
      return this.conf.proxy.password
    },

    proxyDNS() {
      if (!this.conf.id || !this.conf.proxy) return false
      return this.conf.proxy.proxyDNS
    },
  },

  mounted() {
    this.init()
  },

  methods: {
    onWheel(e) {
      let scrollOffset = this.$el.scrollTop
      let maxScrollOffset = this.$el.scrollHeight - this.$el.offsetHeight
      if (scrollOffset === 0 && e.deltaY < 0) e.preventDefault()
      if (scrollOffset === maxScrollOffset && e.deltaY > 0) e.preventDefault()
    },

    onNameInput(value) {
      this.conf.name = value
    },

    updateContainer() {
      if (!this.conf.name || !this.conf.id) return
      browser.contextualIdentities.update(this.conf.id, {
        name: this.conf.name,
        icon: this.conf.icon,
        color: this.conf.color,
      })
    },

    updateName() {
      if (!this.name) return
      this.updateContainer()
    },

    async updateIcon(icon) {
      this.conf.icon = icon
      this.updateContainer()
    },

    async updateColor(color) {
      this.conf.color = color
      this.updateContainer()
    },

    async init() {
      await this.$nextTick()
      if (this.$refs.name) this.$refs.name.recalcTextHeight()
      if (this.$refs.includeHostsInput) {
        this.$refs.includeHostsInput.recalcTextHeight()
      }
      if (this.$refs.excludeHostsInput) {
        this.$refs.excludeHostsInput.recalcTextHeight()
      }
      if (this.$refs.userAgentInput) {
        this.$refs.userAgentInput.recalcTextHeight()
      }
      if (this.$refs.scrollBox) this.$refs.scrollBox.recalcScroll()
    },

    async toggleIncludeHosts() {
      if (!this.conf.includeHostsActive) {
        if (!State.permAllUrls) {
          window.location.hash = 'all-urls'
          State.selectedContainer = null
          this.switchProxy('direct')
          return
        }
      }

      this.conf.includeHostsActive = !this.conf.includeHostsActive
      Actions.saveContainers()
      await this.$nextTick()

      if (this.$refs.scrollBox) this.$refs.scrollBox.recalcScroll()
      if (this.$refs.includeHostsInput) this.$refs.includeHostsInput.focus()
    },

    onIncludeHostsInput(value) {
      this.conf.includeHosts = value
      Actions.saveContainersDebounced()
    },

    async toggleExcludeHosts() {
      if (!this.conf.excludeHostsActive) {
        if (!State.permAllUrls) {
          window.location.hash = 'all-urls'
          State.selectedContainer = null
          this.switchProxy('direct')
          return
        }
      }

      this.conf.excludeHostsActive = !this.conf.excludeHostsActive
      Actions.saveContainers()
      await this.$nextTick()

      if (this.$refs.scrollBox) this.$refs.scrollBox.recalcScroll()
      if (this.$refs.excludeHostsInput) this.$refs.excludeHostsInput.focus()
    },

    onExcludeHostsInput(value) {
      this.conf.excludeHosts = value
      Actions.saveContainersDebounced()
    },

    async switchProxy(type) {
      // Check permissions
      if (type !== 'direct') {
        if (!State.permAllUrls) {
          window.location.hash = 'all-urls'
          this.switchProxy('direct')
          State.selectedContainer = null
          return
        }
      }

      this.conf.proxy = {
        type,
        host: this.proxyHost,
        port: this.proxyPort,
        username: this.proxyUsername,
        password: this.proxyPassword,
        proxyDNS: this.proxyDNS,
      }

      if (type === 'direct') this.conf.proxified = false
      else this.conf.proxified = this.proxyHostValid && this.proxyPortValid

      Actions.saveContainers()

      await this.$nextTick()

      if (this.$refs.scrollBox) this.$refs.scrollBox.recalcScroll()
    },

    onProxyHostInput(value) {
      if (!this.conf.id || !this.conf.proxy) return
      this.conf.proxy.host = value
      this.conf.proxified = this.proxyHostValid && this.proxyPortValid

      Actions.saveContainersDebounced()
    },

    onProxyPortInput(value) {
      if (!this.conf.id || !this.conf.proxy) return
      this.conf.proxy.port = value
      this.conf.proxified = this.proxyHostValid && this.proxyPortValid

      Actions.saveContainersDebounced()
    },

    onProxyUsernameInput(value) {
      if (!this.conf.id || !this.conf.proxy) return
      this.conf.proxy.username = value

      Actions.saveContainersDebounced()
    },

    onProxyPasswordInput(value) {
      if (!this.conf.id || !this.conf.proxy) return
      this.conf.proxy.password = value

      Actions.saveContainersDebounced()
    },

    toggleProxyDns() {
      if (!this.conf.id || !this.conf.proxy) return
      this.conf.proxy.proxyDNS = !this.conf.proxy.proxyDNS

      Actions.saveContainers()
    },

    onFieldKeydown(e, nextFieldName, prevFieldName) {
      if (e.code === 'Enter' || e.code === 'Tab') {
        if (e.shiftKey) {
          if (this.$refs[prevFieldName]) this.$refs[prevFieldName].focus()
        } else {
          if (this.$refs[nextFieldName]) this.$refs[nextFieldName].focus()
        }
        e.preventDefault()
      }
    },

    async toggleUserAgent() {
      if (!this.conf.userAgentActive) {
        if (!State.permWebRequestBlocking) {
          window.location.hash = 'web-request-blocking'
          this.conf.excludeHostsActive = false
          State.selectedContainer = null
          return
        } else if (!State.permAllUrls) {
          window.location.hash = 'all-urls'
          this.conf.excludeHostsActive = false
          State.selectedContainer = null
          return
        }
      }

      this.conf.userAgentActive = !this.conf.userAgentActive
      Actions.saveContainers()
      await this.$nextTick()

      if (this.$refs.scrollBox) this.$refs.scrollBox.recalcScroll()
      if (this.$refs.userAgentInput) this.$refs.userAgentInput.focus()
    },

    onUserAgentInput(value) {
      this.conf.userAgent = value
      Actions.saveContainersDebounced()
    },
  },
}
</script>
