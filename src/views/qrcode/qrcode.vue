<template>
	<div class="qr" :id="uuid">
		<img class="qr__img" :src="qrBase64" @load="imgLoad" />
		<slot />
	</div>
</template>

<script>
import QRcode from 'qrcode'
import html2canvas from 'html2canvas'
export default {
	props: {
		qrContent: {
			type: String,
			default: ''
		},
		value: {
			type: String,
			default: ''
		}
	},
	data() {
		return {
			uuid: `qr_${this._uid}_code`, // 组件唯一ID
			qrBase64: ''
		}
	},
	watch: {
		qrContent: {
			handler: function () {
				this.qrContent && this.init()
			},
			immediate: true
		}
	},
	methods: {
		async init() {
			this.qrBase64 = await this.createQR(this.qrContent)
		},
		createQR(content, options) {
			return new Promise((resolve, reject) => {
				QRcode.toDataURL(content, options).then(url => {
					resolve(url)
				})
			})
		},
		createCanvas() {
			return new Promise((resolve, reject) => {
				const container = document.querySelector(`#${this.uuid}`)
				html2canvas(container).then(canvans => {
					const base64 = canvans.toDataURL('image/png')
					resolve(base64)
				})
			})
		},
		// 需要等图片加载完才能生成canvas
		async imgLoad() {
			const canvansBase64 = await this.createCanvas()
            this.$emit('input', canvansBase64) // 把目标base64丢出去
			console.log(canvansBase64) // 生成二维码+文案图片
		}
	}
}
</script>
