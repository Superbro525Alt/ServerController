<script lang="ts">


    export let title: string;

    export let id: string;

    export let onClose: () => void;

    function destroy() {
        const component = document.querySelector(`.popup`);
        component.remove();

        onClose();
    }
</script>

<div class="popup">
    <div class="popup__content">
        <div class="popup__header">
            <div class="popup__title">{title}</div>
            <svg class="popup__close" viewBox="0 0 12 12" on:click={destroy}>
                <circle cx=6 cy=6 r=6 />
                <line x1=3 y1=3 x2=9 y2=9 />
                <line x1=9 y1=3 x2=3 y2=9 />
            </svg>
        </div>
        <div class="popup__body">
            <slot />
        </div>
    </div>
</div>

<style>
.popup {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 1000 !important;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,.5);
    display: flex;
    justify-content: center;
    align-items: center;
    transition: all .3s ease-in-out;

}

.popup__title {
    font-size: 24px;
    font-weight: 700;
    color: #000;
}

.popup__close {
    width: 20px;
    height: 20px;
    background-repeat: no-repeat;
    background-size: contain;
    cursor: pointer;
    fill:#F44;
}

.popup__close line {
    stroke: #ffffff;
    stroke-width: 2px;
}

.popup__close:hover {
    transform: scale(1.5);
}

.popup__content {
    width: 100%;
    max-width: 500px;
    background-color: #fff;
    border-radius: 10px;
    padding: 20px;
    box-sizing: border-box;
}

.popup__header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}

.popup__body {
    margin-bottom: 20px;
}

.popup__footer {
    display: flex;
    justify-content: flex-end;
}


</style>
