<script>
    import { onMount } from 'svelte';
    import Slider from './Slider.svelte';

    let device = null;
    let server = null;
    let txCharacteristic = null;

    const DEVICE_NAME = 'BBC micro:bit [tupiv]';
    const UART_SERVICE_UUID = '6e400001-b5a3-f393-e0a9-e50e24dcca9e';
    // const TX_CHAR_UUID = '6e400002-b5a3-f393-e0a9-e50e24dcca9e';
    const TX_CHAR_UUID = '6e400003-b5a3-f393-e0a9-e50e24dcca9e';
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

            console.log('Getting TX characteristic...');
            txCharacteristic = await service.getCharacteristic(TX_CHAR_UUID);

            console.log('Connected');
        } catch (error) {
            console.error('Error:', error);
        }
    }

    function handleSliderRelease(event) {
        if (!txCharacteristic) {
            console.log('Not connected');
            return;
        }

        const { id, value } = event.detail;
        const message = `$${id}:${value}#`;
        const byteArray = new TextEncoder().encode(message);

        // Adjust chunk size based on the MTU
        const mtu = 20;
        const chunkSize = mtu;

        for (let i = 0; i < byteArray.length; i += chunkSize) {
            const chunk = byteArray.slice(i, i + chunkSize);
            txCharacteristic.writeValueWithoutResponse(chunk)
                .then(() => {
                    console.log(`Sent: ${message}`);
                })
                .catch(error => {
                    console.error('Error writing characteristic:', error);
                });
        }
    }
</script>

<style>
    button {
        margin: 5px;
    }
</style>

<div>
    <button on:click={connectToDevice}>Connect to Bluetooth Device</button>
    
    <!-- Slider Components -->
    <Slider id="slider1" min="0" max="100" on:release={handleSliderRelease} />
    <Slider id="slider2" min="0" max="100" on:release={handleSliderRelease} />
</div>
