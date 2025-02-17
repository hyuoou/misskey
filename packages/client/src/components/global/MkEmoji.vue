<template>
<img v-if="customEmoji" class="mk-emoji custom" :class="{ normal, noStyle }" :src="url" :alt="alt" :title="alt" decoding="async"/>
<img v-else-if="char && !useOsNativeEmojis" class="mk-emoji" :src="url" :alt="alt" :title="alt" decoding="async"/>
<span v-else-if="char && useOsNativeEmojis" class="mk-emoji-native">{{ char }}</span>
<img v-else-if="useFallbackIcon" class="mk-emoji mk-emoji-fallback" :src="char2filePath('❓')" alt="❓" title="❓" decoding="async"/>
<span v-else class="mk-emoji-fallback">{{ emoji.replace('@.', '') }}</span>
</template>

<script lang="ts" setup>
import { computed, ref, watch } from 'vue';
import { CustomEmoji } from 'misskey-js/built/entities';
import { getStaticImageUrl } from '@/scripts/get-static-image-url';
import { char2filePath } from '@/scripts/twemoji-base';
import { defaultStore } from '@/store';
import { instance } from '@/instance';

const props = defineProps<{
	emoji: string;
	normal?: boolean;
	noStyle?: boolean;
	customEmojis?: CustomEmoji[];
	isReaction?: boolean;
	useFallbackIcon?: boolean;
}>();

const isCustom = computed(() => props.emoji.startsWith(':'));
const char = computed(() => isCustom.value ? null : props.emoji);
const useOsNativeEmojis = computed(() => defaultStore.state.useOsNativeEmojis && !props.isReaction);
const ce = computed(() => props.customEmojis ?? instance.emojis ?? []);
const customEmoji = computed(() => isCustom.value ? ce.value.find(x => x.name === props.emoji.substr(1, props.emoji.length - 2)) : null);
const url = computed(() => {
	if (char.value) {
		return char2filePath(char.value);
	} else {
		return defaultStore.state.disableShowingAnimatedImages
			? getStaticImageUrl(customEmoji.value.url)
			: customEmoji.value.url;
	}
});
const alt = computed(() => customEmoji.value ? `:${customEmoji.value.name}:` : char.value);


</script>

<style lang="scss" scoped>
.mk-emoji {
	height: 1.25em;
	vertical-align: -0.25em;

	&.custom {
		height: 2.5em;
		vertical-align: middle;
		transition: transform 0.2s ease;

		&:hover {
			transform: scale(1.2);
		}

		&.normal {
			height: 1.25em;
			vertical-align: -0.25em;

			&:hover {
				transform: none;
			}
		}
	}

	&.noStyle {
		height: auto !important;
	}
}
</style>
