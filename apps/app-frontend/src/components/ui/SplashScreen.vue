<template>
	<Transition name="splash-fade" @after-leave="onAfterLeave">
		<div v-if="!doneLoading" class="splash-screen dark">
			<div class="app-logo-wrapper" data-tauri-drag-region>
				<svg
					class="app-logo"
					viewBox="0 0 1215 175"
					xml:space="preserve"
					style="
						fill-rule: evenodd;
						clip-rule: evenodd;
						stroke-linejoin: round;
						stroke-miterlimit: 2;
					"
					color="var(--color-contrast)"
				>
					<g transform="translate(10,10)" style="fill:none;stroke:var(--color-brand);stroke-width:11;stroke-linecap:round;">
						<line x1="77.5" y1="8" x2="77.5" y2="147" />
						<line x1="17" y1="42.5" x2="138" y2="112.5" />
						<line x1="17" y1="112.5" x2="138" y2="42.5" />
						<line x1="77.5" y1="8" x2="60" y2="26" />
						<line x1="77.5" y1="8" x2="95" y2="26" />
						<line x1="77.5" y1="147" x2="60" y2="129" />
						<line x1="77.5" y1="147" x2="95" y2="129" />
						<line x1="17" y1="42.5" x2="17" y2="66" />
						<line x1="17" y1="42.5" x2="38" y2="34" />
						<line x1="138" y1="112.5" x2="138" y2="89" />
						<line x1="138" y1="112.5" x2="117" y2="121" />
						<line x1="17" y1="112.5" x2="17" y2="89" />
						<line x1="17" y1="112.5" x2="38" y2="121" />
						<line x1="138" y1="42.5" x2="138" y2="66" />
						<line x1="138" y1="42.5" x2="117" y2="34" />
					</g>
					<text
						x="200"
						y="118"
						font-family="Arial, 'Segoe UI', Helvetica, sans-serif"
						font-weight="800"
						font-size="98"
						letter-spacing="-1"
						style="fill: currentColor"
					>Xmas<tspan style="fill: var(--color-brand)">Legacy</tspan></text>
				</svg>
				<ProgressBar class="loading-bar" :progress="Math.min(loadingProgress, 100)" />
				<span v-if="message">{{ message }}</span>
			</div>
			<div class="gradient-bg" data-tauri-drag-region></div>
			<div class="cube-bg"></div>
			<div class="base-bg"></div>
		</div>
	</Transition>
</template>

<script setup>
import { injectLoadingState } from '@modrinth/ui'
import { ref, watch } from 'vue'

import ProgressBar from '@/components/ui/ProgressBar.vue'
import { loading_listener } from '@/helpers/events.js'

const doneLoading = ref(false)
const loadingProgress = ref(0)
const message = ref()

const MIN_DISPLAY_MS = 500
const mountedAt = Date.now()

const loading = injectLoadingState()

function onAfterLeave() {
	loading.setEnabled(true)
}

watch(
	[loading.barEnabled, loading.pending],
	([barEnabled, pending]) => {
		if (barEnabled) {
			return
		}

		if (pending) {
			loadingProgress.value = 0
			fakeLoadingIncrease()
			return
		}

		const elapsed = Date.now() - mountedAt
		const delay = Math.max(0, MIN_DISPLAY_MS - elapsed)

		setTimeout(() => {
			if (loading.pending.value) {
				return
			}
			doneLoading.value = true
		}, delay)
	},
	{ immediate: true },
)

function fakeLoadingIncrease() {
	if (loadingProgress.value < 95) {
		setTimeout(() => {
			loadingProgress.value += 2
			fakeLoadingIncrease()
		}, 5)
	}
}

loading_listener(async (e) => {
	if (e.event.type === 'directory_move') {
		loadingProgress.value = 100 * (e.fraction ?? 1)
		message.value = 'Updating app directory...'
	} else if (e.event.type === 'checking_for_updates') {
		loadingProgress.value = 100 * (e.fraction ?? 1)
		message.value = 'Checking for updates...'
	}
})
</script>

<style scoped lang="scss">
.splash-screen {
	position: fixed;
	inset: 0;
	z-index: 10000;
}

.splash-fade-leave-active {
	transition: opacity 0.3s ease-in-out;
}

.splash-fade-leave-to {
	opacity: 0;
}

.app-logo-wrapper {
	position: absolute;
	height: 100vh;
	width: 100%;

	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;

	gap: 1rem;

	z-index: 9998;
}

.app-logo {
	height: 2.25rem;
	width: fit-content;
}

.loading-bar {
	max-width: 20rem;
}

.gradient-bg {
	position: absolute;
	height: 100vh;
	width: 100vw;
	background:
		linear-gradient(180deg, rgba(66, 120, 182, 0.275) 0%, rgba(17, 30, 43, 0.5) 97.29%),
		linear-gradient(0deg, rgba(22, 24, 28, 0.64), rgba(22, 24, 28, 0.64));
	z-index: 9997;
}

.cube-bg {
	position: absolute;

	left: 50%;
	top: 50%;
	transform: translate(-50%, -50%);

	width: 180vw;
	height: 180vh;
	opacity: 0.8;
	background: #16181c url('@/assets/loading/cube.png') center no-repeat;
	background-size: contain;

	z-index: 9996;
}

.base-bg {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
	background: var(--color-bg);
	z-index: 9995;
}
</style>
