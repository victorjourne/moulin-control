<script>
    import { onMount } from 'svelte';

    let device = null;
    let server = null;
    let rxCharacteristic = null;
    let receivedMessages = [];

    const DEVICE_NAME = 'BBC micro:bit [tupiv]';
    const UART_SERVICE_UUID = '6e400001-b5a3-f393-e0a9-e50e24dcca9e';
    const RX_CHAR_UUID = '6e400002-b5a3-f393-e0a9-e50e24dcca9e';

    onMount(() => {
        connectToDevice();
    });

    async function connectToDevice() {
        try {
            console.log('Requesting Bluetooth device...');
            device = await navigator.bluetooth.requestDevice({
                filters: [{ name: DEVICE_NAME }],
                optionalServices: [UART_SERVICE_UUID]
            });

            console.log('Connecting to GATT server...');
            server = await device.gatt.connect();
            console.log('Connection status:', device.gatt.connected);

            console.log('Getting UART service...');
            const service = await server.getPrimaryService(UART_SERVICE_UUID);

            console.log('Getting RX characteristic...');
            rxCharacteristic = await service.getCharacteristic(RX_CHAR_UUID);

            // Enable notifications
            await rxCharacteristic.startNotifications();
            rxCharacteristic.addEventListener('characteristicvaluechanged', handleCharacteristicValueChanged);

            console.log('Connected and listening for notifications');
        } catch (error) {
            console.error('Error:', error);
        }
    }

    function handleCharacteristicValueChanged(event) {
        const value = event.target.value;
        const decodedValue = new TextDecoder().decode(value);

        console.log('Received value:', decodedValue);
        receivedMessages = [...receivedMessages, decodedValue];
    }
</script>

<style>
    .message {
        margin: 5px 0;
        padding: 5px;
        border-bottom: 1px solid #ccc;
    }
</style>

<div>
    <h2>Received Messages</h2>
    {#each receivedMessages as message}
        <div class="message">{message}</div>
    {/each}
</div>
