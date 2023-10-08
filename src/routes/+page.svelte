<script lang="ts">
	import ServerButton from "./ServerButton.svelte";
	import Popup from "./Popup.svelte";
	import CustomMenu from "./CustomMenu.svelte";

	import {Tabs, TabItem, Input, Textarea, Button, ButtonGroup, Toast} from 'flowbite-svelte';
	import {
		AdjustmentsHorizontalSolid,
		CodeForkSolid,
		PenSolid,
		DatabaseSolid,
		DotsHorizontalOutline,
		CheckCircleSolid
	} from "flowbite-svelte-icons";

	import Highlight from "svelte-highlight";
  	import {json} from "svelte-highlight/languages";

    // import client/socket from socketio
	import ioClient from 'socket.io-client';
	import {CodeBlock} from "svhighlight";
	import {onMount} from "svelte";
	import Service from "./Service.svelte";


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

	let server_services = {};
	let server_services_new = {};

	let pushed_changes = false;

	let socket;

	let started_service_success = false;
	let started_service_fail = false;


	function add_server() {
		create_server_popup_visible = !create_server_popup_visible;
	}

	function reset_popup() {
		create_server_popup_visible = false;
	}

	function load_server(ip: string, name: string) {
		socket = ioClient(ip);

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
				socket.on('services', (services: string) => {
					var s = JSON.parse(JSON.parse(services));

					if (s.services == undefined) {
						s = JSON.parse(s);
					}

					server_services = s;
				});
			} else {
				console.log("Password incorrect");
				var error = document.querySelector(".error");
				error.innerHTML = "Error connecting to server: Password incorrect";
				socket.close();
			}
		});

		socket.on('service_started', (s) => {
			if (s) {
				started_service_success = true;
			} else {
				started_service_fail = true;
			}

			setTimeout(() => {
				started_service_success = false;
				started_service_fail = false;
			}, 3000);
		});

		socket.on('service_log', (log: string) => {
			console.log(log);
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

	onMount(() => {

		document.addEventListener('keydown', function (e) {
			if (e.key == "Tab") {
				// make it so that it adds spaces
				e.preventDefault();

				// insert spaces at the cursor position without using the text area (only using e)
				var start = e.target.selectionStart;
				var end = e.target.selectionEnd;

				// set textarea value to: text before caret + tab + text after caret
				e.target.value = e.target.value.substring(0, start) +
						"    " + e.target.value.substring(end);

				// put caret at right position again
				e.target.selectionStart =
						e.target.selectionEnd = start + 4;



			}
		});
	});

	function save_file_on_server(filePath: string, fileData: string) {
		socket.emit('change_file', {
			file: filePath,
			data: fileData
		});

		pushed_changes = true;

		setTimeout(() => {
			pushed_changes = false;
		}, 3000);
	}

	function start_service(service: object) {
		socket.emit('start_service', JSON.stringify(service));
	}

	function stop_service(service: object) {

	}



</script>

{#if pushed_changes}
	<div class="fixed bottom-0 right-0 mb-4 mr-4" style="min-width: 15%; margin-right: 0">
		<Toast color="green" align={false}>
			<svelte:fragment slot="icon">
				<CheckCircleSolid class="w-5 h-5" />
				<span class="sr-only">Check icon</span>
			</svelte:fragment>
			Changes pushed
		</Toast>
	</div>
{/if}

{#if started_service_success}
	<div class="fixed bottom-0 right-0 mb-4 mr-4" style="min-width: 15%; margin-right: 0">
		<Toast color="green" align={false}>
			<svelte:fragment slot="icon">
				<CheckCircleSolid class="w-5 h-5" />
				<span class="sr-only">Check icon</span>
			</svelte:fragment>
			Started service
		</Toast>
	</div>
{/if}

{#if started_service_fail}
	<div class="fixed bottom-0 right-0 mb-4 mr-4" style="min-width: 15%; margin-right: 0">
		<Toast color="red" align={false}>
			<svelte:fragment slot="icon">
				<CheckCircleSolid class="w-5 h-5" />
				<span class="sr-only">Check icon</span>
			</svelte:fragment>
			Service failed to start
		</Toast>
	</div>
{/if}

<svelte:head>
	<title>Select Server</title>
	<meta name="description" content="Server Controller" />
</svelte:head>

{#if !loaded_server}
	<h2>Select Server</h2>
{/if}

	{#if !loaded_server}
		<section class="server-holder">
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
		</section>

	{/if}
	{#if loaded_server}


<Tabs style="underline">
  <TabItem open>
    <div slot="title" class="flex items-center gap-2">
      <AdjustmentsHorizontalSolid size="sm" />
      Dashboard
    </div>
    <p class="text-sm text-gray-500 dark:text-gray-400">

    </p>
  </TabItem>
  <TabItem>
    <div slot="title" class="flex items-center gap-2">
      <DatabaseSolid size="sm" />
      Services
    </div>

	  <div class="text-center">
		  {#if server_services.services.length === 0}
			  <p>No services found</p>
		  {/if}
		  {#if server_services.services.length > 0}

			  		{#each server_services.services as service}
						{#if service != {}}
							<Service service={service} startService={start_service} stopService={stop_service}/>
						{/if}
					{/each}


			  {/if}
	  </div>

  </TabItem>
  <TabItem>
    <div slot="title" class="flex items-center gap-2">
      <CodeForkSolid size="sm" />

		Raw Data
    </div>
	<!-- Make an input thats starting value s the current server services but make it editable with donstant syntax highlighting for json-->

	<div class="mb-6">
	  <Textarea value={JSON.stringify(server_services, null, 2)} rows="25" on:input={(e) => server_services_new = e.target.value} />
	</div>
		<ButtonGroup>
				  <Button on:click={() => {if (server_services_new == {}) {server_services_new = server_services};
					  save_file_on_server("./services.json", JSON.stringify(server_services_new));}} >Push Changes</Button>
				  <Button on:click={() => server_services_new = JSON.stringify(server_services, null, 2)} >Reset</Button>
	</ButtonGroup>

  </TabItem>
  <TabItem>
    <div slot="title" class="flex items-center gap-2">
      <PenSolid size="sm" />
      Edit
    </div>

  </TabItem>
    <TabItem>
    <div slot="title" class="flex items-center gap-2">
      <DotsHorizontalOutline size="sm" />
      More
    </div>

  </TabItem>
</Tabs>
	{/if}
	<CustomMenu />


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
