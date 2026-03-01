<script lang="ts">
    import Popup from "./popup.svelte";
    type Popup = {msg: string, image: string, top: number, left: number}
    let popups: Popup[] = $state([])
    let animating: number = 0

    async function sleep(ms: number): Promise<void> {
        return new Promise(
        (resolve) => setTimeout(resolve, ms));
    }
    
    export async function openPopup(popup: {image: string, msg: string}) {
        animating++;
        popups = [
            ...popups,
            {msg: popup.msg, image: popup.image, top: 25+Math.random()*50, left: 25+Math.random()*50}
        ]
        await sleep(2600);
        animating--;
        if (animating == 0) {
            popups = [];
        }

    }
</script>

<div style="pointer-events: none">
    {#each popups as popup, i }
        <Popup msg={popup.msg} image={popup.image} top={popup.top} left={popup.left} i={i}></Popup>
    {/each}
</div>