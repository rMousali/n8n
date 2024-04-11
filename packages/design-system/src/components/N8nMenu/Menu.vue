<template>
	<div
		:class="{
			['menu-container']: true,
			[$style.container]: true,
			[$style.menuCollapsed]: collapsed,
			[$style.transparentBackground]: transparentBackground,
		}"
	>
		<div v-if="$slots.header" :class="$style.menuHeader">
			<!-- <slot name="header"></slot> -->
			<div :class="$style.headerIcon">
				<svg v-if="collapsed" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 768 1024">
					<path
						fill="#7D7D87"
						d="M0 960V64h576l192 192v704zm704-640L512 128H64v768h640zM320 512H128V256h192zm-64-192h-64v128h64zm0 448h64v64H128v-64h64V640h-64v-64h128zm256-320h64v64H384v-64h64V320h-64v-64h128zm64 384H384V576h192zm-64-192h-64v128h64z"
					/>
				</svg>
				<svg v-else xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
					<defs>
						<linearGradient id="a" x1="0%" x2="100%" y1="100%" y2="0%">
							<stop offset="0%" stop-color="#055F4E" />
							<stop offset="100%" stop-color="#56C0A7" />
						</linearGradient>
					</defs>
					<g fill="none" fill-rule="evenodd">
						<path fill="url(#a)" d="M0 0h24v24H0z" />
						<path
							fill="#FFF"
							d="m12 18.14-2.426.809-.946-.631 1.03-.344-.316-.948-1.768.589L7 17.233V14.5a.5.5 0 0 0-.276-.447L5 13.19v-2.382l1.5-.75 1.5.75v1.69c0 .19.107.364.276.449l2 1 .448-.895L9 12.19v-1.382l1.724-.861A.5.5 0 0 0 11 9.5V8h-1v1.19l-1.5.75L7 9.19V6.769L8 6.1V8h1V5.435l.574-.383L12 5.86zM17.5 17a.5.5 0 1 1-.002 1 .5.5 0 0 1 .002-1m-1-11a.5.5 0 1 1-.002 1 .5.5 0 0 1 .002-1m2 6a.5.5 0 1 1-.002 1 .5.5 0 0 1 .002-1m-1.408 1c.207.58.757 1 1.408 1a1.501 1.501 0 0 0 0-3c-.651 0-1.201.42-1.408 1H13v-2h3.5a.5.5 0 0 0 .5-.5V7.908c.581-.207 1-.757 1-1.408 0-.827-.673-1.5-1.5-1.5S15 5.673 15 6.5c0 .65.419 1.2 1 1.408V9h-3V5.5a.5.5 0 0 0-.342-.474l-3-1a.5.5 0 0 0-.435.058l-3 2A.5.5 0 0 0 6 6.5v2.69l-1.724.863A.5.5 0 0 0 4 10.5v3c0 .19.107.363.276.448l1.724.86V17.5a.5.5 0 0 0 .223.416l3 2a.5.5 0 0 0 .435.058l3-1A.5.5 0 0 0 13 18.5V16h2.293l.853.854.013-.013c-.098.2-.159.422-.159.659 0 .827.673 1.5 1.5 1.5s1.5-.673 1.5-1.5-.673-1.5-1.5-1.5c-.238 0-.46.06-.659.16l.013-.013-1-1A.5.5 0 0 0 15.5 15H13v-2z"
						/>
					</g>
				</svg>
			</div>
		</div>
		<div :class="$style.menuContent">
			<div :class="{ [$style.upperContent]: true, ['pt-xs']: $slots.menuPrefix }">
				<div v-if="$slots.menuPrefix" :class="$style.menuPrefix">
					<slot name="menuPrefix"></slot>
				</div>
				<ElMenu :default-active="defaultActive" :collapse="collapsed">
					<N8nMenuItem
						v-for="item in upperMenuItems"
						:key="item.id"
						:item="item"
						:compact="collapsed"
						:tooltip-delay="tooltipDelay"
						:mode="mode"
						:active-tab="activeTab"
						:handle-select="onSelect"
					/>
				</ElMenu>
			</div>
			<div :class="[$style.lowerContent, 'pb-2xs']">
				<slot name="beforeLowerMenu"></slot>
				<ElMenu :default-active="defaultActive" :collapse="collapsed">
					<N8nMenuItem
						v-for="item in lowerMenuItems"
						:key="item.id"
						:item="item"
						:compact="collapsed"
						:tooltip-delay="tooltipDelay"
						:mode="mode"
						:active-tab="activeTab"
						:handle-select="onSelect"
					/>
				</ElMenu>
				<div v-if="$slots.menuSuffix" :class="$style.menuSuffix">
					<slot name="menuSuffix"></slot>
				</div>
			</div>
		</div>
		<div v-if="$slots.footer" :class="$style.menuFooter">
			<slot name="footer"></slot>
		</div>
	</div>
</template>

<script lang="ts" setup>
import { computed, onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';
import { ElMenu } from 'element-plus';
import N8nMenuItem from '../N8nMenuItem';
import type { IMenuItem } from '../../types';
import { doesMenuItemMatchCurrentRoute } from '../N8nMenuItem/routerUtil';

interface MenuProps {
	type?: 'primary' | 'secondary';
	defaultActive?: string;
	collapsed?: boolean;
	transparentBackground?: boolean;
	mode?: 'router' | 'tabs';
	tooltipDelay?: number;
	items?: IMenuItem[];
	modelValue?: string;
}

const props = withDefaults(defineProps<MenuProps>(), {
	type: 'primary',
	collapsed: false,
	transparentBackground: false,
	mode: 'router',
	tooltipDelay: 300,
	items: () => [],
});
const $route = useRoute();

const $emit = defineEmits<{
	(event: 'select', itemId: string);
	(event: 'update:modelValue', itemId: string);
}>();

const activeTab = ref(props.modelValue);

const upperMenuItems = computed(() => {
	console.log('props.items:', props.items);
	return props.items.filter(
		(item: IMenuItem) => item.position === 'top' && item.available !== false,
	);
});

const lowerMenuItems = computed(() =>
	props.items.filter((item: IMenuItem) => item.position === 'bottom' && item.available !== false),
);

const currentRoute = computed(() => {
	return $route ?? { name: '', path: '' };
});

onMounted(() => {
	if (props.mode === 'router') {
		const found = props.items.find((item) =>
			doesMenuItemMatchCurrentRoute(item, currentRoute.value),
		);

		activeTab.value = found ? found.id : '';
	} else {
		activeTab.value = props.items.length > 0 ? props.items[0].id : '';
	}

	$emit('update:modelValue', activeTab.value);
});

const onSelect = (item: IMenuItem): void => {
	if (props.mode === 'tabs') {
		activeTab.value = item.id;
	}

	$emit('select', item.id);
	$emit('update:modelValue', item.id);
};
</script>

<style lang="scss" module>
.container {
	height: 100%;
	display: flex;
	flex-direction: column;
	background-color: var(--menu-background, var(--color-background-xlight));
}

.menuContent {
	display: flex;
	flex-direction: column;
	justify-content: space-between;
	flex-grow: 1;

	& > div > :global(.el-menu) {
		background: none;
		padding: var(--menu-padding, 12px);
	}
}

.upperContent {
	ul {
		padding-top: 0 !important;
	}
}

.lowerContent {
	ul {
		padding-bottom: 0 !important;
	}
}

.menuCollapsed {
	transition: width 150ms ease-in-out;
	:global(.hideme) {
		display: none !important;
	}
}

.transparentBackground {
	background-color: transparent;
}

.headerIcon {
	display: flex;
	align-items: center;
	justify-content: center;
}
</style>
