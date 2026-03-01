<script lang="ts">

    import { animate } from "animejs";
    import { onMount } from "svelte";
	let { msg, image, top, left, i }: { msg: string, image: string, top: number, left: number, i: number } = $props();

    let div: HTMLDivElement;
    let text: HTMLDivElement;
    let show: string = $state("true");

    async function sleep(ms: number): Promise<void> {
        return new Promise(
        (resolve) => setTimeout(resolve, ms));
    }
    onMount(() => {
        spin();
    })
    async function spin() {
        try {
            animate(text, {
                    opacity: [0, 1],
                    rotate: [-360, 0],
                    scale: [0.5, 1],
                    duration: 300
                }
            )
            await animate(div, {
                    opacity: [0, 1],
                    scale: [0.8, 1],
                    translateX: [-187, -150],
                    translateY: [-187, -150],
                    duration: 300
                }
            )
            await sleep(2000)
            animate(text, {
                    scale: [1, 0.5],
                    opacity: [1, 0],
                    rotate: [0, 360],
                    duration: 300
                }
            )
            await animate(div, {
                    opacity: [1, 0],
                    scale: [1, 1.2],
                    translateX: [-150, -125],
                    translateY: [-150, -125],
                    duration: 300
                }
            )
        } catch {
            console.log("failure")
        }
    }
</script>
<div bind:this={div} class="dialog {image}" style="background:center center / cover; top: {top}%; left: {left}%; opacity: 0">
    </div>
<div bind:this={text} style="background: purple; color: lemonchiffon; position: fixed; top: {top}%; left: {left}%; font-size: 2em; opacity: 0; z-index: 500">{msg}</div>
<style>
	.dialog {
        position: fixed;
        height:300px;
        width: 300px;
        transform: translate(-50%, -50%)
	}
</style>