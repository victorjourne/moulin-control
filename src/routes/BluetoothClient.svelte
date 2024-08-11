
<script>
    import { onMount } from 'svelte';

    let device = null;
    let server = null;
    let txCharacteristic = null;

    const DEVICE_NAME = 'BBC micro:bit [tupiv]';
    const UART_SERVICE_UUID = '6e400001-b5a3-f393-e0a9-e50e24dcca9e';
    
    // const RX_CHAR_UUID = '6e400002-b5a3-f393-e0a9-e50e24dcca9e';
    const RX_CHAR_UUID = '6e400003-b5a3-f393-e0a9-e50e24dcca9e';

    onMount(() => {
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
                txCharacteristic = await service.getCharacteristic(RX_CHAR_UUID);
                console.log('TX characteristic:', txCharacteristic);

                console.log('Connected');
            } catch (error) {
                console.error('Error:', error);
            }
        }

        async function writeCharacteristic() {
            if (!txCharacteristic) {
                console.log('Not connected');
                return;
            }

            try {
                const textToSend = 'hello \nbitch';
                const byteArray = new TextEncoder().encode(textToSend);

                console.log('Byte array to send:', byteArray);

                // Adjust chunk size based on the MTU
                const mtu = 20; // Adjust based on actual MTU if needed
                const chunkSize = mtu;

                for (let i = 0; i < byteArray.length; i += chunkSize) {
                    const chunk = byteArray.slice(i, i + chunkSize);

                    console.log('Writing chunk:', chunk);
                    await txCharacteristic.writeValueWithoutResponse(chunk);
                    console.log(`Wrote chunk starting at index ${i}`);
                }

                console.log('Write complete');
            } catch (error) {
                console.error('Error writing characteristic:', error);
            }
        }

        document.getElementById('connect').addEventListener('click', connectToDevice);
        document.getElementById('write').addEventListener('click', writeCharacteristic);
    });
</script>

<style>
    button {
        margin: 5px;
    }
</style>

<div>
    <button id="connect">Connect to Bluetooth Device</button>
    <button id="write">Write Characteristic</button>
</div>
