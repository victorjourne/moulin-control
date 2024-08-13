<script>
    import { createEventDispatcher } from 'svelte';

    export let id;
    export let min = 0;
    export let max = 100;
    export let value = 50;

    const dispatch = createEventDispatcher();

    function handleInputChange(event) {
        value = event.target.value;
    }

    function handleSliderRelease() {
        dispatch('release', { id, value });
    }
</script>

<style>
    .slider-container {
        margin: 10px 0;
    }
    .slider-label {
        display: inline-block;
        width: 50px;
    }
</style>

<div class="slider-container">
    <label class="slider-label" for={id}>{id}</label>
    <input
        type="range"
        id={id}
        min={min}
        max={max}
        value={value}
        on:input={handleInputChange}
        on:change={handleSliderRelease}
    />
</div>
