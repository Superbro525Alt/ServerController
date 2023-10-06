<script lang="ts">
    let pos = { x: 0, y: 0 };
	let showMenu = false;

	async function onRightClick(e) {
		if (showMenu) {
			showMenu = false;
			await new Promise(res => setTimeout(res, 100));
		}

		pos = { x: e.clientX, y: e.clientY };
		showMenu = true;
	}

	function closeMenu() {
		showMenu = false;
	}
</script>

{#if showMenu}
<div class="context-menu">
    <div class="context-menu__item">
        <a href="#" class="context-menu__link" data-action="Edit">Edit</a>
    </div>
    <div class="context-menu__item">
        <a href="#" class="context-menu__link" data-action="Delete">Delete</a>
    </div>
</div>
{/if}

<svelte:body on:contextmenu|preventDefault={onRightClick} />

<style>
.context-menu {
    display: none;
    position: absolute;
    z-index: 10;
}
</style>
