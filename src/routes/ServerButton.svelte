<script lang="ts">
    export let name: string;
    export let ip: string;

    export let onclick: () => void;

    function loadServer() {
        console.log("Loading server");

    }
</script>

<button class="server-button" on:click={() => {onclick(ip, name)}}>
    <h3>{name}</h3>
    <h2>{ip}</h2>
</button>


<style>
.server-button {
    background-color: #2c2c2c;
    border: none;
    border-radius: 5px;
    padding: 10px;
    margin: 10px;
    color: white;
    min-width: 200px;
    height: 100px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    cursor: pointer;
    transition: 0.2s;
}

.server-button:hover {
    background-color: #3c3c3c;
}
</style>
