<template>
    <div>
        <vuci-form :uciConfig="config.name" class="flexbox">
            <vuci-named-section title="MQTT Broker" :name="config.section" v-slot="{ s }" class="section">
                <vuci-form-item-switch
                    :uci-section="s"
                    label="Enable"
                    name="enabled"
                />
                <vuci-form-item-input
                    :uci-section="s"
                    label="Local port"
                    name="local_port"
                />
                <vuci-form-item-switch
                    :uci-section="s"
                    label="Enable remote access"
                    name="enable_ra"
                />
            </vuci-named-section>
            <vuci-named-section title="Broker Settings" :name="config.section" v-slot="{ s }" class="section">
                <a-tabs>
                    <a-tab-pane key="security" tab="Security">
                        <vuci-form-item-switch
                            :uci-section="s"
                            label="Use TLS/SSL"
                            name="use_tls_ssl"
                        />
                        <vuci-form-item-select
                            :uci-section="s"
                            label="TLS Type"
                            name="tls_type"
                            :options="tls_options"
                            initial="cert"
                            depend="use_tls_ssl == 1"
                        />
                        <vuci-form-item-upload
                            :uci-section="s"
                            label="CA file"
                            name="ca_file"
                            depend="use_tls_ssl == 1 && tls_type == 'cert'"
                        />
                        <vuci-form-item-upload
                            :uci-section="s"
                            label="Cert file"
                            name="cert_file"
                            depend="use_tls_ssl == 1 && tls_type == 'cert'"
                        />
                        <vuci-form-item-upload
                            :uci-section="s"
                            label="Key file"
                            name="key_file"
                            depend="use_tls_ssl == 1 && tls_type == 'cert'"
                        />
                        <vuci-form-item-select
                            :uci-section="s"
                            label="TLS version"
                            name="tls_version"
                            :options="tls_versions"
                            initial="all"
                            depend="use_tls_ssl == 1 && tls_type == 'cert'"
                        />
                        <vuci-form-item-input
                            :uci-section="s"
                            label="Pre-Shared-Key"
                            name="psk"
                            depend="use_tls_ssl == 1 && tls_type == 'psk'"
                        />
                        <vuci-form-item-input
                            :uci-section="s"
                            label="Identity"
                            name="identity"
                            depend="use_tls_ssl == 1 && tls_type == 'psk'"
                        />
                    </a-tab-pane>
                    <a-tab-pane key="miscellaneous" tab="Miscellaneous">
                        <vuci-form-item-upload
                            :uci-section="s"
                            label="ACL file"
                            name="acl_file_path"
                        />
                        <vuci-form-item-upload
                            :uci-section="s"
                            label="Password file"
                            name="password_file"
                        />
                        <vuci-form-item-switch
                            :uci-section="s"
                            label="Persistence"
                            name="persistence"
                            initial="0"
                        />
                        <vuci-form-item-switch
                            :uci-section="s"
                            label="Allow Anonymous"
                            name="anonymous_access"
                            initial="1"
                        />
                    </a-tab-pane>
                </a-tabs>
            </vuci-named-section>
        </vuci-form>
    </div>
</template>

<script>
export default {
  data () {
    return {
      config: {
        name: 'mosquitto',
        section: 'mqtt'
      },
      tls_options: [
        ['cert', 'Certificate based'],
        ['psk', 'Pre-Shared-Key based']
      ],
      tls_versions: [
        ['all', 'Support all'],
        ['tlsv1', 'TLSV1'],
        ['tlsv1.1', 'TLSV1.1'],
        ['tlsv1.2', 'TLSV1.2']
      ]
    }
  }
}
</script>

<style>
.flexbox {
    display: flex;
    gap: 10px;
}

.section {
    width: 50%;
}
</style>
