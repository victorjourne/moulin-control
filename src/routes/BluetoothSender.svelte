<script>
    import { getContext } from 'svelte';

    let service;
    let txCharacteristic = null;
    const TX_CHAR_UUID = '6e400003-b5a3-f393-e0a9-e50e24dcca9e';

    // Get Bluetooth service from context
    service = getContext('bluetoothService');

    async function writeCharacteristic(message) {
        txCharacteristic = await $service.getCharacteristic(TX_CHAR_UUID);
        if (txCharacteristic) {
            console.log("send message", `$${message}#`);
            const value = new TextEncoder().encode(`${message}\n`);
            await txCharacteristic.writeValue(value);
        }
    }

    export { writeCharacteristic };
</script>
