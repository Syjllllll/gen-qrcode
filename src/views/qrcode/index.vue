<template>
	<el-card style="height: 100vh">
		<template #header>
			<div class="card-header">
				<el-tooltip class="box-item" effect="customized" placement="right">
					<template #content>
						<div style="color: rgb(37, 30, 30)">
							<p>批量模式下</p>
							<p>1、编号区间不能大于1000</p>
							<p>2、预览仅支持首尾两张</p>
						</div>
					</template>
					<el-badge class="badge" value="🔗" style="cursor: pointer">
						<div style="width: 77px">二维码生成</div>
					</el-badge>
				</el-tooltip>
				<el-icon :size="20" color="#66e" @click="reload" class="refresh"><RefreshRight /></el-icon>
			</div>
		</template>
		<el-form label-position="left" label-width="100px">
			<el-form-item label="模式">
				<el-radio-group :disabled="mode === 2 && !isDownloadFinish" v-model="mode" @change="onModeChange">
					<el-radio :label="2" border>批量</el-radio>
					<el-radio :label="1" border>单个</el-radio>
				</el-radio-group>
			</el-form-item>
			<el-form-item label="编号" v-if="mode === 1">
				<el-input v-model="num" maxlength="15" @change="onNumChange" show-word-limit style="width: 400px">
					<template #prepend>
						<el-select @visible-change="clearCustomPrefixInput" v-model="prefixStr" placeholder="前缀" style="width: 100px">
							<el-option v-for="item in prefixs" :key="item.value" :label="item.label" :value="item.value" />
							<template #footer>
								<el-button v-if="!isAdding" plain @click="onAddOption"
									>自定义<el-icon><EditPen /></el-icon
								></el-button>
								<template v-else>
									<el-input v-model="optionName" style="margin-bottom: 10px" class="option-input" placeholder="请输入前缀" />
									<el-button type="primary" @click="onConfirm">确认</el-button>
									<el-button @click="clearCustomPrefixInput">取消</el-button>
								</template>
							</template>
						</el-select>
					</template>
				</el-input>
				<div class="red-tips">{{ singleContent }}</div>
			</el-form-item>
			<el-form-item label="编号" v-if="mode === 2">
				<el-input placeholder="开始编号" v-model="startNum" maxlength="15" show-word-limit style="width: 340px">
					<template #prepend>
						<el-select @visible-change="clearCustomPrefixInput" v-model="prefixStr" placeholder="前缀" style="width: 100px">
							<el-option v-for="item in prefixs" :key="item.value" :label="item.label" :value="item.value" />
							<template #footer>
								<el-button v-if="!isAdding" plain @click="onAddOption"
									>自定义<el-icon><EditPen /></el-icon
								></el-button>
								<template v-else>
									<el-input v-model="optionName" style="margin-bottom: 10px" class="option-input" placeholder="请输入前缀" />
									<el-button type="primary" @click="onConfirm">确认</el-button>
									<el-button @click="clearCustomPrefixInput">取消</el-button>
								</template>
							</template>
						</el-select>
					</template>
				</el-input>
				<span style="margin: 0 10px">至</span>
				<el-input placeholder="结束编号" v-model="endNum" maxlength="15" show-word-limit style="width: 240px" />
				<el-button type="primary" plain style="margin-left: 10px" @click="handleConfirmGenerate">开始生成</el-button>
				<div v-if="startNum" class="red-tips">{{ prefixStr + startNum }}</div>
				<span v-if="endNum">~</span>
				<div v-if="endNum" class="red-tips">{{ prefixStr + endNum }}</div>
			</el-form-item>
			<el-form-item label="预览">
				<div v-if="mode === 1">
					<div class="preview" v-if="num">
						<qrcode-vue :value="singleContent" :size="size" :margin="2" :foreground="color" level="H" class="single" />
						<div class="num">{{ num }}</div>
					</div>
					<span class="no-data" v-else>暂无数据</span>
					<canvas id="singleCanvas" width="400" height="450" style="display: none"></canvas>
				</div>
				<div v-if="mode === 2" style="display: flex">
					<div v-for="(content, index) in batchContent" :key="index">
						<div class="preview" :style="index === 0 || index === batchContent.length - 1 ? '' : 'display: none'">
							<qrcode-vue :value="content" :size="size" :margin="2" :foreground="color" level="H" :class="`batch${index}`" />
							<div class="num">{{ content.split('=')[1] }}</div>
							<div class="tip" v-if="batchContent.length > 0 && index === 0">(第一张)</div>
							<div class="tip" v-if="batchContent.length > 0 && index === batchContent.length - 1">(最后一张)</div>
						</div>
						<canvas :id="`batch${index}`" width="400" height="450" style="display: none"></canvas>
					</div>
					<span class="no-data" v-if="batchContent.length === 0">暂无数据</span>
				</div>
			</el-form-item>
			<el-form-item>
				<el-button v-if="mode === 1" type="primary" @click="download">下载</el-button>
				<el-button v-if="mode === 2" type="primary" @click="batchDownload">批量下载</el-button>
			</el-form-item>
		</el-form>
	</el-card>
	<div
		v-if="loading"
		class="el-loading-mask"
		style="background-color: rgba(255, 255, 255, 0.9); display: flex; justify-content: center; flex-direction: column; align-items: center"
	>
		<div class="sk-chase">
			<div class="sk-chase-dot"></div>
			<div class="sk-chase-dot"></div>
			<div class="sk-chase-dot"></div>
			<div class="sk-chase-dot"></div>
			<div class="sk-chase-dot"></div>
			<div class="sk-chase-dot"></div>
		</div>
		<div style="position: absolute; bottom: 2vh; right: 2vw">
			<div class="loading-text" style="text-align: center; margin: 10px 0; color: #66e">请稍候...</div>
		</div>
	</div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed, inject, nextTick } from 'vue'
import QrcodeVue from 'qrcode.vue'
import { ElMessage } from 'element-plus'
import JSZip from 'jszip'
import FileSaver from 'file-saver'
const reload = inject('reload') as Function
const loading = ref(false)
const mode = ref(2)
const num = ref('') // 单个二维码编号
const singleContent = computed(() => {
	if (!num.value) return ''
	return `${prefixStr.value}${num.value}`
})
const isConfirmGenerate = ref(false)
const batchContent = ref<string[]>([])
const handleConfirmGenerate = () => {
	if (!startNum.value || !endNum.value) return ElMessage.error('请输入开始编号和结束编号')
	if (Number(startNum.value) > Number(endNum.value)) return ElMessage.error('开始编号不能大于结束编号')
	if (Number(startNum.value) === Number(endNum.value)) return ElMessage.error('开始编号不能等于结束编号')
	if (Number(endNum.value) - Number(startNum.value) > 1000) return ElMessage.error('一次最多生成1000张二维码')
	if (
		batchContent.value &&
		batchContent.value[0] === `${prefixStr.value}${startNum.value}` &&
		batchContent.value.at(-1) === `${prefixStr.value}${endNum.value}`
	)
		return ElMessage.error('未改变编号，无需重新生成')
	isDownloadFinish.value = false
	loading.value = true

	let tempArr = []
	for (let i = Number(startNum.value); i <= Number(endNum.value); i++) {
		tempArr.push(i)
	}
	batch.value = tempArr.map(item => String(item))

	setTimeout(() => {
		let arr = []
		for (let i = Number(startNum.value); i <= Number(endNum.value); i++) {
			arr.push(`${prefixStr.value}${i}`)
		}
		batchContent.value = arr

		handleGenerate()
	}, 200)
}

const startNum = ref('') // 批量二维码开始编号
const endNum = ref('') // 批量二维码结束编号

const size = ref(400)
const color = ref('#000000')
onMounted(() => {})

const isAdding = ref(false)
const optionName = ref('')
const onAddOption = () => {
	isAdding.value = true
}
const onConfirm = () => {
	const isEmpty = optionName.value.split('').every(item => item === ' ') // '   'split后为['', '', '', '']
	if (optionName.value && !isEmpty) {
		prefixs.value.push({
			label: optionName.value,
			value: optionName.value
		})
	} else {
		prefixs.value.push({
			label: `空`,
			value: ''
		})
	}
	clearCustomPrefixInput()
}

const clearCustomPrefixInput = () => {
	optionName.value = ''
	isAdding.value = false
}
const prefixStr = ref('http://youkac.cn?boxNum=')
const prefixs = ref([
	{ label: '优卡充', value: 'http://youkac.cn?boxNum=' },
	{ label: '格鲁斯', value: 'http://hitc.org.cn/charge?boxNum=' },
	{ label: '昆山豪杰', value: 'http://kshaojie.cn/charge?boxNum=' },
	{ label: '万桩之家', value: 'https://rc.youkac.net:58882?boxNum=' }
])

const batch = ref<string[]>([])

const onNumChange = () => {
	const canvas: HTMLCanvasElement = <HTMLCanvasElement>document.getElementsByClassName('single')[0]
	const canvas2: HTMLCanvasElement = <HTMLCanvasElement>document.getElementById('singleCanvas')
	const ctx = canvas2?.getContext('2d')
	const img = new Image()
	img.src = canvas?.toDataURL('image/png')
	img.onload = function () {
		ctx!.drawImage(img, 0, 0, 400, 400)
		ctx!.fillStyle = '#fff'
		ctx!.fillRect(0, 400, 400, 50)
		ctx!.fillStyle = '#000'
		ctx!.font = 'bold 45px Arial'
		ctx!.textAlign = 'center'
		ctx!.textBaseline = 'middle'
		ctx!.fillText(num.value, 200, 420)
	}
}

const handleGenerate = () => {
	const len = batch.value.length
	if (len === 0) return
	// 分段生成 一次最多生成20张
	const batchArr = []
	for (let i = 0; i < len; i += 20) {
		batchArr.push(batch.value.slice(i, i + 20))
	}

	batchArr.map((arr, arrIndex) => {
		setTimeout(() => {
			arr.map((item, index) => {
				const canvas: HTMLCanvasElement = <HTMLCanvasElement>document.getElementsByClassName(`batch${arrIndex * 20 + index}`)[0]
				const canvas2: HTMLCanvasElement = <HTMLCanvasElement>document.getElementById(`batch${arrIndex * 20 + index}`)
				const ctx = canvas2?.getContext('2d')
				const img = new Image()
				img.src = canvas?.toDataURL('image/png')
				img.onload = function () {
					ctx!.drawImage(img, 0, 0, 400, 400)
					ctx!.fillStyle = '#fff'
					ctx!.fillRect(0, 400, 400, 50)
					ctx!.fillStyle = '#000'
					ctx!.font = 'bold 45px Arial'
					ctx!.textAlign = 'center'
					ctx!.textBaseline = 'middle'
					ctx!.fillText(item, 200, 420)
				}
			})
		}, 100)
		if (arrIndex === batchArr.length - 1) {
			setTimeout(() => {
				ElMessage.success('二维码生成成功')
				loading.value = false
			}, 100)
		}
	})
}
const isDownloadFinish = ref(true)
const onModeChange = (val: number) => {
	// if (val === 2) {
	// 	reload()
	// 	// loading.value = true
	// 	// setTimeout(() => {
	// 	// 	reload()
	// 	// }, 2000)
	// }

	// num.value = ''
	// startNum.value = ''
	// endNum.value = ''
}

const download = () => {
	if (!num.value) return ElMessage.error('请输入编号')
	try {
		const canvas: HTMLCanvasElement = <HTMLCanvasElement>document.getElementById('singleCanvas')
		const a = document.createElement('a')
		a.href = canvas.toDataURL('image/png')
		a.download = `${num.value}.jpg`
		a.click()
		setTimeout(() => {
			ElMessage.success('下载成功')
		}, 0)
	} catch (error) {
		console.log(error)
		ElMessage.error('下载失败')
	}
}

const getYYMMDD = (date: Date) => {
	const year = date.getFullYear()
	const month = date.getMonth() + 1
	const day = date.getDate()
	return `${year}${month < 10 ? '0' + month : month}${day < 10 ? '0' + day : day}`
}
const batchDownload = () => {
	// if (batch.value.length === 0) return ElMessage.error('请输入编号')
	if (batchContent.value.length === 0) return ElMessage.error('请先生成二维码')
	try {
		loading.value = true
		setTimeout(() => {
			const data = batch.value.map((item, index) => {
				const canvas: HTMLCanvasElement = <HTMLCanvasElement>document.getElementById(`batch${index}`)
				return canvas.toDataURL('image/png')
			})
			const zip = new JSZip()
			data.map((item, index) => {
				zip.file(`${batch.value[index]}.jpg`, item.split('base64,')[1], { base64: true })
			})
			zip.generateAsync({ type: 'blob' }).then(content => {
				const zipFileName = `${getYYMMDD(new Date())}.zip`
				FileSaver.saveAs(content, zipFileName)
			})
		}, 200)
		setTimeout(() => {
			loading.value = false
			ElMessage.success('下载成功')
			isDownloadFinish.value = true
			reload()
		}, 1200)
	} catch (error) {
		console.log(error)
		ElMessage.error('下载失败')
		loading.value = false
	}
}
</script>
<style scoped lang="scss">
.card-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
}
.preview {
	width: 400px;
	height: 450px;
	display: flex;
	flex-direction: column;
}

.num {
	font-size: 45px;
	font-weight: bold;
	color: #000;
	text-align: center;
}

.tip {
	margin-top: 10px;
	font-size: 14px;
	color: #606266;
	text-align: center;
}

.no-data {
	font-size: 14px;
	color: #606266;
	text-align: center;
}

.red-tips {
	color: red;
	font-weight: bold;
	font-size: 16px;
	font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
	margin: 0 10px;
}
.sk-chase {
	width: 40px;
	height: 40px;
	position: relative;
	animation: sk-chase 2.5s infinite linear both;
}

.sk-chase-dot {
	width: 100%;
	height: 100%;
	position: absolute;
	left: 0;
	top: 0;
	animation: sk-chase-dot 2s infinite ease-in-out both;
}

.sk-chase-dot:before {
	content: '';
	display: block;
	width: 25%;
	height: 25%;
	background-color: #66e;
	border-radius: 100%;
	animation: sk-chase-dot-before 2s infinite ease-in-out both;
}

.sk-chase-dot:nth-child(1) {
	animation-delay: -1.1s;
}
.sk-chase-dot:nth-child(2) {
	animation-delay: -1s;
}
.sk-chase-dot:nth-child(3) {
	animation-delay: -0.9s;
}
.sk-chase-dot:nth-child(4) {
	animation-delay: -0.8s;
}
.sk-chase-dot:nth-child(5) {
	animation-delay: -0.7s;
}
.sk-chase-dot:nth-child(6) {
	animation-delay: -0.6s;
}
.sk-chase-dot:nth-child(1):before {
	animation-delay: -1.1s;
}
.sk-chase-dot:nth-child(2):before {
	animation-delay: -1s;
}
.sk-chase-dot:nth-child(3):before {
	animation-delay: -0.9s;
}
.sk-chase-dot:nth-child(4):before {
	animation-delay: -0.8s;
}
.sk-chase-dot:nth-child(5):before {
	animation-delay: -0.7s;
}
.sk-chase-dot:nth-child(6):before {
	animation-delay: -0.6s;
}

@keyframes sk-chase {
	100% {
		transform: rotate(360deg);
	}
}

@keyframes sk-chase-dot {
	80%,
	100% {
		transform: rotate(360deg);
	}
}

@keyframes sk-chase-dot-before {
	50% {
		transform: scale(0.4);
	}
	100%,
	0% {
		transform: scale(1);
	}
}
.loading-text {
	animation: blink 1.8s infinite;
}
@keyframes blink {
	0% {
		opacity: 1;
	}
	50% {
		opacity: 0;
	}
	100% {
		opacity: 1;
	}
}

.refresh:hover {
	cursor: pointer;
}
.refresh {
	animation: rotate 1s 1;
}

@keyframes rotate {
	0% {
		transform: rotate(0deg);
	}
	100% {
		transform: rotate(360deg);
	}
}

::v-deep(.el-badge__content--danger) {
	background-color: #fff;
}

::v-deep(.badge:hover .el-badge__content) {
	animation: blink 0.6s 2;
}
</style>
