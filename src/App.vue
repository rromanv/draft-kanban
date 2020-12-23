<template>
	<!-- https://learnvue.co/2020/01/how-to-add-drag-and-drop-to-your-vuejs-project/ -->
	<div class="bg-gray-200 min-h-screen">
		<Header />
		<div class="grid grid-cols-12 gap-4">
			<div class="col-span-11">
				<div :class="`grid-cols-${categories.length}`" class="grid gap-4 min-h-screen -mt-20 pt-24 pb-4 px-4">
					<div
						class="text-center bg-gray-400 px-4 py-2"
						v-for="c in categories"
						:key="c.id"
						@dragover.prevent
						@dragenter.prevent
						@drop="onDrop($event, c.value)"
					>
						<div class="text-2xl opa font-bold text-blue-800">{{ c.title }}</div>
						<div
							draggable="true"
							class="bg-white p-4 my-2 cursor-move w-full rounded-3xl"
							v-for="i in filteredData(c.value)"
							:key="i.id"
							@dragstart="dragging($event, i)"
							@dragend="endDragging"
						>
							{{ i.name }}
						</div>
						<input
							:id="`${c.value}Input`"
							v-if="adding == c.value"
							class="bg-white p-4 cursor-move w-full rounded-3xl focus:shadow-outline focus:outline-none"
							type="text"
							v-model="newData"
							@keyup.esc="cancelAdd"
							@keyup.enter="add(c.value)"
						/>
						<button
							v-else
							@click="isAdding(c.value)"
							class="bg-blue-500 hover:bg-blue-600 w-full flex justify-center p-3 rounded-3xl focus:outline-none"
						>
							<svg-icon class="text-gray-300" :size="32" type="mdi" :path="addIcon" />
						</button>
					</div>
				</div>
			</div>
			<div class="mt-4 -ml-8 cols-auto">
				<button
					class="mx-auto bg-blue-500 hover:bg-blue-600 flex justify-center p-3 rounded-full focus:outline-none"
					@click="addingCategory = true"
				>
					<svg-icon class="text-gray-300" :size="32" type="mdi" :path="addIcon" />
				</button>
			</div>
		</div>
	</div>
	<div
		v-if="addingCategory"
		class="absolute top-0 left-0 w-screen h-screen bg-black bg-opacity-75 flex justify-center items-center"
	>
		<div class="w-full max-w-md">
			<div class="bg-white shadow-md rounded px-8 pt-6 pb-8 mb-4">
				<div class="mb-4">
					<label class="block text-gray-700 text-sm font-bold mb-2" for="username"> New Category </label>
					<input
						class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
						id="username"
						type="text"
						placeholder="Category"
						v-model="newCategory"
					/>
				</div>
				<div class="flex items-center justify-between">
					<button
						class="w-full bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
						type="button"
						@click="addCategory"
					>
						Add Category
					</button>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	import Header from './components/Header.vue'
	import { ref, reactive, computed, nextTick } from 'vue'
	import SvgIcon from '@jamescoyle/vue-icon'
	import { mdiTooltipPlusOutline } from '@mdi/js'
	import { nanoid } from 'nanoid'
	import camelCase from 'lodash.camelcase'

	export default {
		name: 'App',
		components: {
			Header,
			SvgIcon,
		},
		setup() {
			const addIcon = mdiTooltipPlusOutline

			const newData = ref('')

			const newCategory = ref('')

			const adding = ref('')

			const addingCategory = ref(false)

			const isAdding = async (category) => {
				adding.value = category
				await nextTick()
				document.getElementById(`${category}Input`).focus()
			}

			const data = reactive([])

			const filteredData = (category) => data.filter((d) => d.category == category)

			const changeCategory = (id, category) => {
				data.map((i) => (i.id == id ? (i.category = category) : null))
			}

			const onDrop = (e, category) => {
				const itemID = e.dataTransfer.getData('theData')
				changeCategory(itemID, category)
			}

			const dragging = (e, item) => {
				e.target.classList.add('opacity-25')
				e.dataTransfer.dropEffect = 'move'
				e.dataTransfer.effectAllowed = 'move'
				e.dataTransfer.setData('theData', item.id)
			}

			const endDragging = (e) => {
				e.target.classList.remove('opacity-25')
			}

			const categories = reactive([])

			const add = (category) => {
				data.push({ id: nanoid(), name: newData.value, category })
				newData.value = ''
				adding.value = ''
			}

			const addCategory = (category) => {
				categories.push({ id: nanoid(), title: newCategory.value, value: camelCase(newCategory.value) })

				newCategory.value = ''
				addingCategory.value = false
			}

			const cancelAdd = () => {
				newData.value = ''
				adding.value = ''
			}
			return {
				addIcon,
				newData,
				adding,
				isAdding,
				data,
				categories,
				filteredData,
				onDrop,
				dragging,
				endDragging,
				add,
				cancelAdd,
				newCategory,
				addCategory,
				addingCategory,
			}
		},
	}
</script>
