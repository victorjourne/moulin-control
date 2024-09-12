<script>
    import { onMount, setContext, createEventDispatcher } from 'svelte';
    import { writable } from 'svelte/store';

    let device = null;
    let server = null;
    const DEVICE_NAME = 'BBC micro:bit [tupiv]'; // Your device name
    const UART_SERVICE_UUID = '6e400001-b5a3-f393-e0a9-e50e24dcca9e';
    const dispatch = createEventDispatcher();

    // Provide the service context to child components


    let service = writable()

    setContext('bluetoothService', service);

    async function connectToDevice() {
        try {
            console.log('Requesting Bluetooth Device...');
            device = await navigator.bluetooth.requestDevice({
                filters: [{ name: DEVICE_NAME }],
                optionalServices: [UART_SERVICE_UUID]
            });

            console.log('Connecting to GATT Server...');
            server = await device.gatt.connect();

            console.log('Getting Primary Service...');
            $service = await server.getPrimaryService(UART_SERVICE_UUID);

            // Dispatch an event to indicate connection success
            dispatch('connected', { connected: true });

            console.log('Connected to Bluetooth Device');
        } catch (error) {
            console.error('Error:', error);
        }
    }

    async function disconnectFromDevice() {
        if (device && device.gatt.connected) {
            console.log('Disconnecting from device...');
            device.gatt.disconnect();
            device = null;
            server = null;
            service = null;

            // Dispatch an event to indicate disconnection
            dispatch('connected', { connected: false });

            console.log('Disconnected from Bluetooth Device');
        }
    }

    onMount(() => {
        // Handle any logic that should run when the component mounts
    });
</script>

<style>
    button {
        margin: 5px;
    }
</style>

<div>
    <!-- Button to trigger Bluetooth connection -->
    <button on:click={connectToDevice}>Connect to Bluetooth Device</button>
    <button on:click={disconnectFromDevice}>Disconnect</button>
    <slot></slot>
</div>
