<script lang="ts">
    import { compileCrossword, newCrosswordController } from "$lib/crossword";
    import crosswordDefinition from "$lib/test.json";
    import "$lib/crossword.css";
	import { onMount } from "svelte";
    let gridEl: HTMLElement
    let cluesEl: HTMLElement
    onMount(()=>{
      let controller = newCrosswordController(
        crosswordDefinition,
        gridEl,
        cluesEl,
      );
    })
</script>
<div bind:this={gridEl} />
<div bind:this={cluesEl}/>
