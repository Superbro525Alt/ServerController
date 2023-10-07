<script lang="ts">
	import ServerButton from "./ServerButton.svelte";
	import Popup from "./Popup.svelte";
	import CustomMenu from "./CustomMenu.svelte";

	// import client/socket from socketio
	import ioClient from 'socket.io-client';


	let loaded_server = false;
	let loaded_server_data = {
		name: "",
		ip: ""
	};

	let servers = [{
		name: "Server 1",
		ip: "202.90.245.22:6161"
	}, {
		name: "Dev",
		ip: "127.0.0.1:6161"
	}, {
		name: "Dev 2",
		ip: "http://localhost:3000"
	}];

	let create_server_popup_visible = false;

	let server_name = "";
	let server_ip = "";
	let server_password = "";

	let show_load_server_popup = false;

	function add_server() {
		create_server_popup_visible = !create_server_popup_visible;
	}

	function reset_popup() {
		create_server_popup_visible = false;
	}

	function load_server(ip: string, name: string) {
		const socket = ioClient(ip);

				// check if server is alive
		socket.on('connect_error', (err: any) => {
			console.log("Error connecting to server");
			console.log(err);
			var error = document.querySelector(".error");
			error.innerHTML = "Error connecting to server: Server not alive";
			socket.close();
		});

		socket.on('connect', () => {
			console.log("Connected to server");
			socket.emit('password', server_password);
		});

		socket.on('login', (correct: boolean) => {
			if (correct) {
				console.log("Password correct");
				var error = document.querySelector(".error");
				error.innerHTML = "";
				loaded_server = true;
				loaded_server_data = {
					name: name,
					ip: ip
				};


				socket.on('disconnect', () => {
					console.log("Disconnected from server");
					loaded_server = false;
					loaded_server_data = {
						name: "",
						ip: ""
					};
				});

				// get if server sends a message
				socket.on('data', (services: string) => {
					var s = JSON.parse(services);

					console.log(s);
				});
			} else {
				console.log("Password incorrect");
				var error = document.querySelector(".error");
				error.innerHTML = "Error connecting to server: Password incorrect";
				socket.close();
			}
		});


	}

	function load_server_popup(ip: string, name: string) {
		server_ip = ip;
		server_name = name;
		show_load_server_popup = true;
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

{#if !loaded_server}
	<h2>Select Server</h2>
{/if}

<section class="server-holder">
	{#if !loaded_server}

		{#if create_server_popup_visible}
			<Popup title="Add Server" id="Add Server" onClose={reset_popup}>
				<input type="text" placeholder="Name" bind:value={server_name} />
				<input type="text" placeholder="IP" bind:value={server_ip} />
				<button on:click={add_server_to_list} >Add Server</button>
			</Popup>
		{/if}
		{#if show_load_server_popup}
			<Popup title="Load Server" id="Load Server" onClose={() => show_load_server_popup = false}>
				<input type="text" placeholder="Password" bind:value={server_password} />
				<button on:click={() => load_server(server_ip, server_name)} >Load Server</button>
				<p class="error"></p>
			</Popup>
		{/if}

		{#if servers.length === 0}
			<p>No servers found</p>
		{/if}
		{#if servers.length > 0}
			{#each servers as server}
				<ServerButton name={server.name} ip={server.ip} onclick={load_server_popup}/>
			{/each}
		{/if}
		{#if loaded_server}
			<p>Server Loaded</p>
		{/if}

		<button on:click={add_server} class="add-server" >Add Server</button>

	{/if}

	<CustomMenu />
</section>

<style>

	h2 {
		margin-bottom: 1%;
		font-size: 30px;
		text-align: center;
		margin-top: 0;
	}
	.error {
		color: red;
		margin-bottom: 0;
	}
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
