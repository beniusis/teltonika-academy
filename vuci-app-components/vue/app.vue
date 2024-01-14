<template>
  <div>
    <vuci-form :uciConfig="uciConfig" v-if="upToDate">
      <vuci-typed-section type="interface" :columns="columns">
        <template #name="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="name" />
        </template>
        <template #address="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="address" />
        </template>
        <template #netmask="{ s }">
          <vuci-form-item-dummy :uci-section="s" name="netmask" />
        </template>
        <template #actions="{ s }">
          <a-space :size="30">
            <a-button type="primary" @click="handleEdit(s)">Edit</a-button>
            <a-button type="danger" @click="handleDelete(s)">Delete</a-button>
          </a-space>
        </template>
      </vuci-typed-section>
      <template #footer>
        <div class="space-align-block">
          <a-space>
            Interface name
            <a-input v-model="interfaceName" />
            <a-button type="primary" @click="handleCreate">Create</a-button>
          </a-space>
        </div>
      </template>
    </vuci-form>
    <Modal
      :data="interfaceData"
      :uciConfig="uciConfig"
      @cancelClick="handleCancelClick"
      :key="interfaceData.sid"
    />
  </div>
</template>

<script>
import Modal from './components/Modal.vue'

export default {
  components: {
    Modal
  },

  data () {
    return {
      uciConfig: 'vuci_components_task',
      columns: [
        { name: 'name', label: 'Interface name' },
        { name: 'address', label: 'Address' },
        { name: 'netmask', label: 'Netmask' },
        { name: 'actions' }
      ],
      interfaceName: '',
      interfaceData: {
        visible: false,
        action: '',
        sid: '',
        name: ''
      },
      upToDate: true
    }
  },

  methods: {
    handleEdit (s) {
      this.interfaceData.action = 'edit'
      this.interfaceData.visible = true
      this.interfaceData.sid = s['.name']
      this.interfaceData.name = `Interface ${s.name}`
    },

    handleDelete (s) {
      this.$confirm({
        title: `Are you sure you want to delete ${s.name}?`,
        onOk: () => {
          this.$uci.del(this.uciConfig, s['.name'])
          this.$uci.save()
            .then(() => this.$uci.apply())
            .then(() => this.updateData())
            .then(() => {
              this.$message.success(`Successfully deleted ${s.name} interface!`)
            })
            .catch(err => this.$message.error(`Failed to delete ${s.name} interface: ${err.message}`))
        }
      })
    },

    handleCreate () {
      if (this.interfaceName === '') {
        this.$message.warning('Interface name must be entered!')
        return
      }

      this.$spin()
      const sid = this.$uci.add(this.uciConfig, 'interface')
      this.$uci.set(this.uciConfig, sid, 'name', this.interfaceName)
      this.$uci.save()
        .then(() => this.$uci.apply())
        .then(() => this.getLastSid())
        .then(lastSid => {
          if (lastSid === null) throw new Error('Sid failure!')
          this.interfaceData.action = 'create'
          this.interfaceData.visible = true
          this.interfaceData.sid = lastSid
          this.interfaceData.name = `Interface ${this.interfaceName}`
          this.interfaceName = ''
        })
        .catch(err => this.$message.error(`Failed to create an interface: ${err.message}`))
        .finally(() => {
          this.$spin(false)
          this.updateData()
        })
    },

    handleCancelClick ({ action, sid }) {
      if (action === 'createCancel') {
        this.$spin()
        this.$uci.del(this.uciConfig, sid)
        this.$uci.save()
          .then(() => this.$uci.apply())
          .catch(err => this.$message.error(err.message))
          .finally(() => this.$spin(false))
      }
      this.interfaceData.visible = false
      this.updateData()
    },

    async getLastSid () {
      await this.$uci.load(this.uciConfig)
      const last = this.$uci.sections(this.uciConfig, 'interface').at(-1)
      return last['.name']
    },

    async updateData () {
      this.upToDate = false
      setTimeout(() => {
        this.upToDate = true
      }, 100)
    }
  }
}
</script>
