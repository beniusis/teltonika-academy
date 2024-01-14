<template>
  <a-modal :title="data.name" :visible="data.visible" :footer="null" :closable="false">
    <vuci-form :uciConfig="uciConfig" ref="inputs">
      <vuci-named-section :name="data.sid" v-slot="{ s }" :key="data.sid">
        <vuci-form-item-select
          :uci-section="s"
          @change="handleProtocolChange"
          :label="'Protocol'"
          name="proto"
          :options="protocols"
          :initial="'static'"
        />
        <vuci-form-item-input
          :uci-section="s"
          :label="'IP Address'"
          name="address"
          depend="proto === 'static'"
          rules="ip4addr"
        />
        <vuci-form-item-input
          :uci-section="s"
          :label="'Netmask'"
          name="netmask"
          depend="proto === 'static'"
          rules="netmask4"
        />
        <vuci-form-item-input
          :uci-section="s"
          :label="'Gateway'"
          name="gateway"
          depend="proto === 'static'"
          rules="ip4addr"
        />
        <vuci-form-item-list
          :uci-section="s"
          :label="'DNS'"
          name="dns"
          depend="proto === 'static'"
          rules="ip4addr"
        />
      </vuci-named-section>
      <template #footer>
        <a-space :size="30" style="display: flex; justify-content: center; margin-top: 5%;">
          <a-button type="primary" @click="handleSave">Save</a-button>
          <a-button type="danger" @click="handleCancel">Cancel</a-button>
        </a-space>
      </template>
    </vuci-form>
  </a-modal>
</template>

<script>
export default {
  props: {
    uciConfig: {
      type: String,
      required: true
    },
    data: {
      type: Object,
      default: null
    }
  },

  data () {
    return {
      protocols: ['static', 'dhcp']
    }
  },

  methods: {
    handleProtocolChange (self) {
      if (self.model === 'dhcp') {
        this.$uci.del(this.uciConfig, this.data.sid, 'address')
        this.$uci.del(this.uciConfig, this.data.sid, 'netmask')
        this.$uci.del(this.uciConfig, this.data.sid, 'gateway')
        this.$uci.del(this.uciConfig, this.data.sid, 'dns')
      } else {
        this.$uci.reset()
      }
    },

    handleSave () {
      const formData = this.$refs.inputs
      formData.validate()
        .then(valid => {
          if (!valid) {
            if (this.data.action === 'create') {
              this.$emit('cancelClick', { action: 'createCancel', sid: this.data.sid })
            }
            this.$message.warning('Inputs are not valid!')
          }
        })
        .then(() => formData.save())
        .then(() => formData.apply())
        .then(() => formData.load())
        .then(() => formData.reset())
        .then(() => this.$emit('cancelClick', { action: null, sid: null }))
        .catch(err => this.$message.error(err.message))
    },

    handleCancel () {
      this.$refs.inputs.reset()
      if (this.data.action === 'edit') {
        this.$emit('cancelClick', { action: 'editCancel', sid: null })
      } else {
        this.$emit('cancelClick', { action: 'createCancel', sid: this.data.sid })
      }
    }
  }
}
</script>
