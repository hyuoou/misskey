<template>
<XColumn :column="column" :is-stacked="isStacked" @parent-focus="$event => emit('parent-focus', $event)">
	<template #header><i class="ti ti-star" style="margin-right: 8px;"></i>{{ column.name }}</template>

	<MkPagination ref="pagingComponent" :pagination="pagination">
		<template #empty>
			<div class="_fullinfo">
				<img src="https://xn--931a.moe/assets/info.jpg" class="_ghost"/>
				<div>{{ i18n.ts.noNotes }}</div>
			</div>
		</template>

		<template #default="{ items }">
			<XList v-slot="{ item }" :items="items" :direction="'down'" :no-gap="false" :ad="false">
				<XNote :key="item.id" :note="item.note" :class="$style.note"/>
			</XList>
		</template>
	</MkPagination>
</XColumn>
</template>

<script lang="ts" setup>
import { ref } from 'vue';
import XColumn from './column.vue';
import { Column } from './deck-store';
import MkPagination from '@/components/MkPagination.vue';
import XNote from '@/components/MkNote.vue';
import XList from '@/components/MkDateSeparatedList.vue';
import { i18n } from '@/i18n';

defineProps<{
	column: Column;
	isStacked: boolean;
}>();

const emit = defineEmits<{
	(ev: 'parent-focus', direction: 'up' | 'down' | 'left' | 'right'): void;
}>();

const pagingComponent = ref<InstanceType<typeof MkPagination>>();

const pagination = {
	endpoint: 'i/favorites' as const,
	limit: 10,
};
</script>

<style lang="scss" module>
.note {
	background: var(--panel);
	border-radius: var(--radius);
}
</style>
