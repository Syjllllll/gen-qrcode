<template>
	<div class="flex">
		<VueDraggable
			v-model="list1"
			:animation="150"
			:group="{ name: 'people', pull: 'clone', put: false }"
			:sort="false"
			class="list1"
			@add="onAdd"
			@end="onEnd"
			@clone="onClone"
		>
			<div v-for="item in list1" :key="item.id" class="list1-item">
				{{ item.userName }}
			</div>
		</VueDraggable>
		<VueDraggable v-model="seat1" :disabled="seat1[0].id ? true : false" :animation="150" group="people" class="seat1" @add="onAdd">
			<div class="seat-item">{{ seat1[0].userName }}</div>
			<div v-if="seat1[0].id" class="close" @click="handleRemove(seat1[0].seatId)">×</div>
		</VueDraggable>
		<VueDraggable v-model="seat2" :disabled="seat2[0].id ? true : false" :animation="150" group="people" class="seat2" @add="onAdd">
			<div class="seat-item">{{ seat2[0].userName }}</div>
			<div v-if="seat2[0].id" class="close" @click="handleRemove(seat2[0].seatId)">×</div>
		</VueDraggable>
		<VueDraggable v-model="seat3" :disabled="seat3[0].id ? true : false" :animation="150" group="people" class="seat3" @add="onAdd">
			<div class="seat-item">{{ seat3[0].userName }}</div>
			<div v-if="seat3[0].id" class="close" @click="handleRemove(seat3[0].seatId)">×</div>
		</VueDraggable>
	</div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import { SortableEvent, VueDraggable } from 'vue-draggable-plus'
// const isSeat1Disabled = ref(false)
const list1 = ref([
	{
		userName: '张思思',
		id: '1'
	},
	{
		userName: '王微微',
		id: '2'
	},
	{
		userName: '色色色',
		id: '3'
	},
	{
		userName: '洒洒水',
		id: '4'
	}
])
interface Seat {
	seatName: string
	seatId: string
	userName?: string
	id?: string
}
const seat1 = ref<Seat[]>([{ seatName: '11-01', seatId: '1' }])
const seat2 = ref<Seat[]>([{ seatName: '11-02', seatId: '2' }])
const seat3 = ref<Seat[]>([{ seatName: '11-03', seatId: '3' }])

function onClone() {
	console.log('clone')
}

const onAdd = (event: SortableEvent): void => {
	console.log('add', event)
	if (event.to.className === 'seat1' && event.from.className === 'list1') {
		const obj1 = seat1.value[0]
		const obj2 = seat1.value[1]
		seat1.value = [{ ...obj1, ...obj2 }]
	}
	if (event.to.className === 'seat2' && event.from.className === 'list1') {
		const obj1 = seat2.value[0]
		const obj2 = seat2.value[1]
		seat2.value = [{ ...obj1, ...obj2 }]
	}
	if (event.to.className === 'seat3' && event.from.className === 'list1') {
		const obj1 = seat3.value[0]
		const obj2 = seat3.value[1]
		seat3.value = [{ ...obj1, ...obj2 }]
	}
}
const onEnd = (event: SortableEvent) => {
	console.log('end', event)
	console.log('seat1', seat1.value[0].id)
}
const handleRemove = (seatId: string) => {
	if (seat1.value[0].seatId === seatId) {
		seat1.value = [{ seatName: '11-01', seatId: '1' }]
	}
	if (seat2.value[0].seatId === seatId) {
		seat2.value = [{ seatName: '11-02', seatId: '2' }]
	}
	if (seat3.value[0].seatId === seatId) {
		seat3.value = [{ seatName: '11-03', seatId: '3' }]
	}
}
</script>

<style>
.flex {
	display: flex;
	box-sizing: border-box;
}

.list1 {
	padding: 1rem;
	background-color: #6b72800d;
	border-radius: 0.25rem;
	grid-gap: 0.5rem;
	gap: 0.5rem;
	overflow: auto;
	display: flex;
	flex-direction: column;
	text-align: center;
	width: 80px;
	min-height: 50px;
	margin: auto;
	overflow: hidden;
}

/* seat1 继承 list1 */
.seat1 {
	position: relative;
	padding: 1rem;
	background-color: #6b72800d;
	border-radius: 0.25rem;
	grid-gap: 0.5rem;
	gap: 0.5rem;
	overflow: auto;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	width: 80px;
	height: 40px;
	margin: auto;
	overflow: hidden;
}

.seat2 {
	position: relative;
	padding: 1rem;
	background-color: #6b72800d;
	border-radius: 0.25rem;
	grid-gap: 0.5rem;
	gap: 0.5rem;
	overflow: auto;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	width: 80px;
	height: 40px;
	margin: auto;
	overflow: hidden;
}
.seat3{
	position: relative;
	padding: 1rem;
	background-color: #6b72800d;
	border-radius: 0.25rem;
	grid-gap: 0.5rem;
	gap: 0.5rem;
	overflow: auto;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	width: 80px;
	height: 40px;
	margin: auto;
	overflow: hidden;

}

.list1-item {
	cursor: move;
	padding: 0.75rem;
	background-color: #6b72800d;
	border-radius: 0.25rem;
}

.close {
	position: absolute;
	top: 2px;
	right: 2px;
	width: 28px;
	height: 28px;
	font-size: 24px;
	color: rgb(192, 186, 186);
	display: flex;
	justify-content: center;
	align-items: center;
	cursor: pointer;
	transition: background-color 0.3s;
	border-radius: 50%;
}

.close:hover {
	background-color: rgba(238, 238, 238, 0.8);
}

/* .seat-item {
	cursor: move;
	padding: 0.75rem;
	background-color: #6b72800d;
	border-radius: 0.25rem;
} */
/* .seat-item::after {
	content: '座位';
	color: #bfb8b8;
} */
</style>
