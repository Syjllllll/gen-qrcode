<template>
	<el-config-provider :locale="locale" :size="size">
		<router-view v-if="routerAlive" />
	</el-config-provider>
</template>

<script setup lang="ts">
import { computed, nextTick, onMounted, ref, provide } from 'vue'
import { RouterView } from 'vue-router'
import { useTitle } from '@vueuse/core'
import { useI18n } from 'vue-i18n'
import { messages } from '@/i18n'
import { handleThemeStyle } from '@/utils/theme'
import { useAppStore } from '@/store/modules/app'
const routerAlive = ref(true)
const reload = () => {
	routerAlive.value = false
	nextTick(() => {
		routerAlive.value = true
	})
}
provide('reload', reload)
const appStore = useAppStore()
const { t } = useI18n()
const locale = computed(() => messages[appStore.language].el)
const size = computed(() => appStore.componentSize)

// 设置标题
useTitle(t('app.title'))

onMounted(() => {
	nextTick(() => {
		// 初始化主题样式
		handleThemeStyle(appStore.theme)
	})
})
</script>
<style>

</style>
