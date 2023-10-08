<script lang="ts">
    import {Button, Card, Indicator} from "flowbite-svelte";

    export let service: object;

    export let stopService: () => void;
    export let startService: () => void;

    console.log(service);
</script>

<div style="width: 100%;">
    <Card href="#" padding="sm" size="xl">
        <div class="flex flex-row items-center justify-between">
            {#if service.active}
                <Indicator color="green" size="lg" />
            {:else}
                <Indicator color="red" size="lg" />
            {/if}
        <h5 class="mb-1 text-xl font-medium text-gray-900 dark:text-white">{service.name}</h5>

            {#if service.active}
                <Button color="red" size="sm" on:click={() => {stopService(service)}}>Stop</Button>
            {:else}
                <Button color="green" size="sm" on:click={() => {startService(service)}}>Start</Button>
            {/if}

        </div>
    </Card>
</div>

<style>

</style>
