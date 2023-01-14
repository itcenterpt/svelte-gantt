<script lang="ts">
    import { getContext } from 'svelte';

    import Dependency from './Dependency.svelte';
    
    import { taskStore, rowStore } from '../../core/store';

    const { visibleHeight } = getContext('dimensions');

    export let paddingTop;
    export let dependencies = [];

    let visibleDependencies = [];
    $: {
        const result = [];
        if(dependencies.length > 0 && $taskStore.ids.length > 0 && $rowStore.ids.length > 0){
            for (let i = 0; i < dependencies.length; i++) {
                const dependency = dependencies[i];
                const map = $taskStore.entities;
                const mapRow = $rowStore.entities;

                const fromTask = map[dependency.fromId];
                const toTask = map[dependency.toId];
                const fromRow = fromTask && mapRow[fromTask.model.resourceId];
                const toRow = toTask && mapRow[toTask.model.resourceId];
                const canShow = fromRow?.hidden === false && toRow?.hidden === false;
                if(
                    fromTask && toTask && canShow
                    && Math.min(fromTask.top, toTask.top) <= paddingTop + $visibleHeight 
                    && Math.max(fromTask.top, toTask.top) >= paddingTop
                ) {
                    result.push(dependency);
                }
            }
        }
        visibleDependencies = result;
    }
</script>

<div class="dependency-container">
    {#each visibleDependencies as dependency (dependency.id)}
        <Dependency {...dependency}/>
    {/each}
</div>

<style>
    .dependency-container {
        position: absolute;
        width: 100%;
        height: 100%;
        
        pointer-events: none;
        top: 0;
        float: left;
        overflow: hidden;
        z-index: 0;
    }
</style>