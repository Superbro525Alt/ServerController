<script lang="ts">
	import ServerButton from "./ServerButton.svelte";
	import Popup from "./Popup.svelte";
	import CustomMenu from "./CustomMenu.svelte";

	// import sockets


	let servers = [{
		name: "Server 1",
		ip: "202.90.245.22:6161"
	}, {
		name: "Server 1",
		ip: "202.90.245.22:6161"
	}];

	let create_server_popup_visible = false;

	let server_name = "";
	let server_ip = "";

	function add_server() {
		create_server_popup_visible = !create_server_popup_visible;
	}

	function reset_popup() {
		create_server_popup_visible = false;
	}

	function load_server(ip: string, name: string) {
		console.log(name, ip);
	}

	function add_server_to_list() {
		if (server_name == "" || server_ip == "") {
			return;
		}
		servers = [...servers, {
			name: server_name,
			ip: server_ip
		}];
		reset_popup();
	}

</script>

<svelte:head>
	<title>Select Server</title>
	<meta name="description" content="Server Controller" />
</svelte:head>

<section class="server-holder">

	{#if create_server_popup_visible}
		<Popup title="Add Server" id="Add Server" onClose={reset_popup}>
			<input type="text" placeholder="IP" bind:value={server_ip} />
			<input type="text" placeholder="Name" bind:value={server_name} />
			<button on:click={add_server_to_list} >Add Server</button>
		</Popup>
	{/if}
	{#if servers.length === 0}
		<p>No servers found</p>
	{/if}
	{#if servers.length > 0}
		{#each servers as server}
			<ServerButton name={server.name} ip={server.ip} onclick={load_server}/>
		{/each}
	{/if}

	<button on:click={add_server} class="add-server" >Add Server</button>

	<CustomMenu />
</section>

<style>
.server-holder {
	display: flex;
	flex-direction: row;
	align-items: center;
	justify-content: space-evenly;
}

.add-server {
	position: fixed;
	bottom: 20px;
	right: 20px;
	width: 100px;
	height: 100px;
	border-radius: 50%;
	background-color: #2f3136;
	border: none;
	color: white;
	font-size: 20px;
	cursor: pointer;
	/* add a box shadow to make it look a little bit elevated */
	box-shadow: 0px 0px 10px 0px rgba(0,0,0,0.75);
}

.add-server:hover {
	background-color: #36393f;
}

input {
    width: 100%;
    height: 40px;
    border: 1px solid #ccc;
    border-radius: 5px;
    padding: 0 10px;
    box-sizing: border-box;
    outline: none;
    font-size: 16px;
    color: #000;
	margin-bottom: 10px;
}

button {
	width: 100%;
	height: 40px;
	border: none;
	border-radius: 5px;
	background-color: #2f3136;
	color: #fff;
	font-size: 16px;
	cursor: pointer;
	transition: background-color 0.2s ease;
}

button:hover {
	background-color: #36393f;
}
</style>
