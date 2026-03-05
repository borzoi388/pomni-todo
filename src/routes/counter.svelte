<script lang="ts">
    let { number, show }: { number: number, show: boolean } = $props();
</script>

{#if (show)}
    <div style="display: flex; position: fixed; bottom: 0px; left: 0px;">
        <img src="./lib/images/counter.gif" height=80>
        {#each [...(number.toString(10))] as char}
            <img src="./lib/images/{char}.gif" height=70>
        {/each}
    </div>
{/if}