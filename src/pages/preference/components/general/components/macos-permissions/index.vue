<script setup lang="ts">
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow'
import { message } from '@tauri-apps/plugin-dialog'
import { Space } from 'ant-design-vue'
import { checkInputMonitoringPermission, requestInputMonitoringPermission } from 'tauri-plugin-macos-permissions-api'
import { onMounted, ref } from 'vue'

import ProList from '@/components/pro-list/index.vue'
import ProListItem from '@/components/pro-list-item/index.vue'
import { isMac } from '@/utils/platform'

const authorized = ref(false)

onMounted(async () => {
  authorized.value = await checkInputMonitoringPermission()

  if (authorized.value) return

  const appWindow = getCurrentWebviewWindow()

  await appWindow.setAlwaysOnTop(true)

  await message('如果权限已开启，先选中后点击“-”按钮将其删除，再重新手动添加，并重启应用以确保权限生效。', {
    title: '输入监控权限',
    okLabel: '前往开启',
    kind: 'warning',
  })

  await appWindow.setAlwaysOnTop(false)

  requestInputMonitoringPermission()
})
</script>

<template>
  <ProList
    v-if="isMac"
    title="权限设置"
  >
    <ProListItem
      description="开启输入监控权限，以便接收系统的键盘和鼠标事件来响应你的操作。"
      title="输入监控权限"
    >
      <Space
        v-if="authorized"
        class="text-success font-bold"
        :size="4"
      >
        <div class="i-solar:verified-check-bold text-4.5" />

        <span>已授权</span>
      </Space>

      <Space
        v-else
        class="cursor-pointer text-primary font-bold"
        :size="4"
        @click="requestInputMonitoringPermission"
      >
        <div class="i-solar:round-arrow-right-bold text-4.5" />

        <span>去授权</span>
      </Space>
    </ProListItem>
  </ProList>
</template>
