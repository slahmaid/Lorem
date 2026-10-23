<template>
	<section ref="root" class="relative flex h-svh flex-col p-20 touch:overflow-hidden s:p-40">

		<div class="relative flex flex-1 flex-col items-center justify-center gap-10 s:flex-row">
			<div ref="band" class="aspect-[18/13] w-120 shrink-0 bg-black gpu:bg-transparent touch:origin-center touch:bg-black touch:will-change-transform s:w-180" />

			<gl-text
				tag="h1"
				:reveal="false"
				class="shrink-0 text-60 leading-9 font-medium tracking-h2 s:text-175"
			>Chlah</gl-text>
		</div>
	</section>
</template>

<script setup lang="ts">
const band = useTemplateRef<HTMLElement>('band')
const root = useTemplateRef<HTMLElement>('root')

defineExpose({ band })

// Touch devices (phones, tablets) skip the desktop WebGL panel morph in FeaturedPanel.
// This fallback grows the hero block with plain CSS transforms as you scroll, so it
// expands to fill the screen and hands off to the black projects panel below it.
const { $mouse, $write, $scrollY } = useNuxtApp()

let unWrite: (() => void) | null = null
let last = Number.NaN

const ease = (t: number) => 1 - Math.pow(1 - t, 3)

const update = () => {
	const el = band.value
	const sec = root.value
	if (!el || !sec) return

	const scroll = $scrollY()
	if (scroll === last) return
	last = scroll

	const h = sec.offsetHeight || window.innerHeight
	const p = ease(Math.min(1, Math.max(0, scroll / (h * 0.6))))

	// Layout box (ignores the current transform)
	const w = el.offsetWidth
	const bh = el.offsetHeight
	let x = 0
	let y = 0
	for (let n: HTMLElement | null = el; n && n !== sec; n = n.offsetParent as HTMLElement | null) {
		x += n.offsetLeft
		y += n.offsetTop
	}

	const vw = sec.offsetWidth
	const cover = Math.max(vw / w, h / bh) * 1.05
	const s = 1 + (cover - 1) * p

	// Move the block's centre toward the centre of the hero's lower half
	const dx = (vw / 2 - (x + w / 2)) * p
	const dy = (h * 0.75 - (y + bh / 2)) * p

	el.style.transform = `translate3d(${dx}px, ${dy}px, 0) scale(${s})`
}

onMounted(() => {
	if ($mouse) return
	if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return

	update()
	unWrite = $write(update)

	window.addEventListener('resize', onResize)
})

const onResize = () => { last = Number.NaN }

onBeforeUnmount(() => {
	unWrite?.()
	window.removeEventListener('resize', onResize)
})
</script>
