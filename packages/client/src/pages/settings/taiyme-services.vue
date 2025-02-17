<template>
<MkTab v-model="tab" style="margin-bottom: var(--margin);">
	<option value="overview"><i class="ti ti-info-circle"></i> {{ i18n.ts.overview }}</option>
	<option value="settings"><i class="ti ti-settings"></i> {{ i18n.ts.settings }}</option>
</MkTab>

<div v-if="tab === 'overview'" class="_formRoot">
	<FormSection>
		<MkKeyValue class="_formBlock" :copy="version">
			<template #key>{{ i18n.ts.version }}</template>
			<template #value>{{ version }}</template>
		</MkKeyValue>
	</FormSection>

	<FormSection>
		<MkKeyValue class="_formBlock">
			<template #key>taiyme/misskeyについて</template>
			<template #value>taiyme/misskeyは、taiyによって無償でメンテナンスされています。<br>継続して提供するためにも、開発のサポートや寄付をお願いします。</template>
		</MkKeyValue>
		<div class="_formLinks">
			<FormLink to="https://github.com/taiyme/misskey" external>
				<template #icon><i class="ti ti-code"></i></template>
				taiyme/misskeyの開発に参加
				<template #suffix>GitHub</template>
			</FormLink>
			<FormLink to="https://u.taiy.me/donate" external>
				<template #icon><i class="ti ti-pig-money"></i></template>
				taiyに寄付
				<template #suffix>Donate</template>
			</FormLink>
		</div>
	</FormSection>

	<FormSection>
		<template #label>コントリビューター</template>
		<div class="_formLinks">
			<FormLink to="https://github.com/taiyme" external>@taiyme</FormLink>
			<FormLink to="https://github.com/cffnpwr" external>@cffnpwr</FormLink>
			<FormLink to="https://github.com/Steve-0628" external>@Steve-0628</FormLink>
		</div>
	</FormSection>

	<FormSection>
		<template #label>支援者</template>
		<div v-for="patron in patrons" :key="patron">{{ patron }}</div>
	</FormSection>
</div>

<div v-else-if="tab === 'settings'" class="_formRoot">
	<MkInfo warn class="_formBlock">設定は自動で保存されません。画面下部の保存ボタンを使用してください。</MkInfo>

	<FormSection>
		<FormSwitch v-model="tmsVerticalInstanceTicker" class="_formBlock">
			ノートのインスタンス情報を左端に表示
			<template #caption>タイムライン上のインスタンス情報を左端に表示します。</template>
		</FormSwitch>

		<FormSwitch v-model="tmsIsLongEnabled" class="_formBlock">
			ノートを畳む
			<template #caption>タイムライン上のノートが特定の条件の場合、畳んで表示します。無効にすると常に展開して表示します。</template>
		</FormSwitch>
		<FormFolder>
			<template #label>ノートを畳む条件</template>
			<template v-if="tmsIsLongEnabled" #suffix>有効</template>
			<template v-else #suffix>無効</template>

			<div class="_formRoot">
				<MkInfo class="_formBlock">0以上の整数を指定してください。0を指定すると条件が無効になります。</MkInfo>
				<FormInput v-model="tmsIsLongTextElHeight" type="number" class="_formBlock">
					<template #label>ノート本文の高さ</template>
					<template #suffix>px</template>
					<template #caption>タイムライン上のノートがこの高さを超えた場合、畳んで表示します。ピクセル単位で指定してください。</template>
				</FormInput>
				<FormInput v-model="tmsIsLongFilesLength" type="number" class="_formBlock">
					<template #label>添付ファイルの個数</template>
					<template #suffix>個</template>
					<template #caption>タイムライン上のノートの添付ファイルがこの個数を超えた場合、畳んで表示します。</template>
				</FormInput>
				<FormInput v-model="tmsIsLongUrlsLength" type="number" class="_formBlock">
					<template #label>URLの個数</template>
					<template #suffix>個</template>
					<template #caption>タイムライン上のノートのURLがこの個数を超えた場合、畳んで表示します。</template>
				</FormInput>
				<FormInput v-model="tmsIsLongPollLength" type="number" class="_formBlock">
					<template #label>アンケートの選択肢の個数</template>
					<template #suffix>個</template>
					<template #caption>タイムライン上のノートのアンケートの選択肢がこの個数を超えた場合、畳んで表示します。</template>
				</FormInput>
			</div>
		</FormFolder>
	</FormSection>

	<MkButton class="_formBlock" primary :disabled="!changed" @click="save"><i class="ti ti-device-floppy"></i> {{ i18n.ts.save }}</MkButton>
</div>
</template>

<script lang="ts" setup>
import { watch } from 'vue';
import FormLink from '@/components/form/link.vue';
import FormSwitch from '@/components/form/switch.vue';
import FormInput from '@/components/form/input.vue';
import FormSection from '@/components/form/section.vue';
import FormFolder from '@/components/form/folder.vue';
import MkButton from '@/components/MkButton.vue';
import MkKeyValue from '@/components/MkKeyValue.vue';
import MkInfo from '@/components/MkInfo.vue';
import MkTab from '@/components/MkTab.vue';
import * as os from '@/os';
import { unisonReload } from '@/scripts/unison-reload';
import { i18n } from '@/i18n';
import { version } from '@/config';
import { defaultStore } from '@/store';

const patrons = [
	'すえ',
	'Midra',
	'ゆー',
	'こちゅだぁほ',
];

let tab = $ref('overview');
let changed = $ref(false);

const tmsVerticalInstanceTicker = $ref(defaultStore.state.tmsVerticalInstanceTicker);
const tmsIsLongEnabled = $ref(defaultStore.state.tmsIsLongEnabled);
const tmsIsLongTextElHeight = $ref(defaultStore.state.tmsIsLongTextElHeight);
const tmsIsLongFilesLength = $ref(defaultStore.state.tmsIsLongFilesLength);
const tmsIsLongUrlsLength = $ref(defaultStore.state.tmsIsLongUrlsLength);
const tmsIsLongPollLength = $ref(defaultStore.state.tmsIsLongPollLength);

watch([
	$$(tmsVerticalInstanceTicker),
	$$(tmsIsLongEnabled),
	$$(tmsIsLongTextElHeight),
	$$(tmsIsLongFilesLength),
	$$(tmsIsLongUrlsLength),
	$$(tmsIsLongPollLength),
], () => {
	changed = true;
});

async function check() {
	const isNumberInRange = (x: number, min?: number, max?: number): boolean => {
		if (!Number.isInteger(x)) return false;
		if (Math.sign(x) === -1) return false;
		if (min == null) min = -Infinity;
		if (max == null) max = Infinity;
		return (min <= x) && (x <= max);
	};
	return (
		isNumberInRange(tmsIsLongTextElHeight, 0) &&
		isNumberInRange(tmsIsLongFilesLength, 0) &&
		isNumberInRange(tmsIsLongUrlsLength, 0) &&
		isNumberInRange(tmsIsLongPollLength, 0)
	);
}

async function save() {
	if (await check()) {
		defaultStore.set('tmsVerticalInstanceTicker', tmsVerticalInstanceTicker);
		defaultStore.set('tmsIsLongEnabled', tmsIsLongEnabled);
		defaultStore.set('tmsIsLongTextElHeight', tmsIsLongTextElHeight);
		defaultStore.set('tmsIsLongFilesLength', tmsIsLongFilesLength);
		defaultStore.set('tmsIsLongUrlsLength', tmsIsLongUrlsLength);
		defaultStore.set('tmsIsLongPollLength', tmsIsLongPollLength);

		const { canceled } = await os.confirm({
			type: 'info',
			text: i18n.ts.reloadToApplySetting,
		});

		if (canceled) return;

		unisonReload();
	} else {
		os.alert({
			type: 'error',
		});
	}
}
</script>
