<script setup lang="ts">
import { VueButton, VueDarkToggle, VueIcon, VueSelect } from '@vue/devtools-ui'
import { isInChromePanel } from '@vue/devtools-shared'
import { useDevToolsBridgeRpc, useDevToolsState } from '@vue/devtools-core'

// #region view mode
const viewMode = inject<Ref<'overlay' | 'panel'>>('viewMode', ref('overlay'))
const viewModeSwitchVisible = computed(() => viewMode.value === 'panel' && isInChromePanel)
const { toggle: toggleViewMode } = useToggleViewMode()
// #endregion

const bridgeRpc = useDevToolsBridgeRpc()
const router = useRouter()

const expandSidebar = computed({
  get: () => devtoolsClientState.value.expandSidebar,
  set: v => (devtoolsClientState.value.expandSidebar = v),
})

const splitScreenEnabled = computed({
  get: () => devtoolsClientState.value.splitScreen.enabled,
  set: v => (devtoolsClientState.value.splitScreen.enabled = v),
})

function refreshPage() {
  location.reload()
}

// #region toggle app
const devtoolsState = useDevToolsState()
const appRecords = computed(() => devtoolsState.appRecords.value.map(app => ({
  label: app.name + (app.version ? ` (${app.version})` : ''),
  value: app.id,
})))

const activeAppRecordId = ref(devtoolsState.activeAppRecordId.value)
const activeAppRecordName = computed(() => appRecords.value.find(app => app.value === activeAppRecordId.value)?.label ?? '')

watch(activeAppRecordId, (id) => {
  bridgeRpc.toggleApp(`${id}`).then(() => {
    router.push('/overview').then(() => {
      refreshCurrentPageData()
    })
  })
})
// #endregion
</script>

<template>
  <div>
    <div px3 py2 border="b base" flex="~ gap-2">
      <VueDarkToggle>
        <template #default="{ isDark, toggle }">
          <VueButton outlined type="primary" @click="toggle">
            <div i-carbon-sun dark:i-carbon-moon translate-y--1px /> {{ isDark.value ? 'Dark' : 'Light' }}
          </VueButton>
        </template>
      </VueDarkToggle>
      <VueButton outlined type="primary" @click="expandSidebar = !expandSidebar">
        <VueIcon :icon="expandSidebar ? 'i-carbon-side-panel-close' : 'i-carbon-side-panel-open'" />
        {{ expandSidebar ? 'Minimize Sidebar' : 'Expand Sidebar' }}
      </VueButton>
      <VueButton to="/settings" outlined type="primary">
        <div i-carbon-settings-adjust /> Settings
      </VueButton>
    </div>
    <div px3 py2 border="b base" flex="~ gap-2">
      <VueButton outlined type="primary" @click="splitScreenEnabled = !splitScreenEnabled">
        <div i-carbon-split-screen />
        {{ splitScreenEnabled ? 'Close Split Screen' : 'Split Screen' }}
      </VueButton>
    </div>
    <div px3 py2 flex="~ gap2">
      <template v-if="appRecords.length > 1">
        <VueSelect v-model="activeAppRecordId" :options="appRecords" :placeholder="activeAppRecordName || 'Toggle App'" :button-props="{ outlined: true, type: 'primary' }" />
      </template>
      <VueButton outlined type="primary" @click="refreshPage">
        Refresh Page
      </VueButton>
      <VueButton v-if="viewModeSwitchVisible" outlined type="primary" @click="toggleViewMode('overlay')">
        Switch to Overlay Mode
      </VueButton>
    </div>
  </div>
</template>
