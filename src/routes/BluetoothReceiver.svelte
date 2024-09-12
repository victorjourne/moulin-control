<script>
    import { getContext } from 'svelte';
    import { writable } from 'svelte/store';

    let service;
    let rxCharacteristic = null;
    const RX_CHAR_UUID = '6e400002-b5a3-f393-e0a9-e50e24dcca9e';

    // Create a Svelte store to hold the received value
    const receivedValue = writable('');

    // Retrieve the Bluetooth service from context
    service = getContext('bluetoothService');

    // Function to handle incoming data
    function handleValueChanged(event) {
        const value = new TextDecoder().decode(event.target.value);
        console.log('Received:', value);
        receivedValue.set(value);  // Update the store with the received value
    }


    const listen = async (service) => {
        try {
            if (service) {
                // Get the RX characteristic from the service
                rxCharacteristic = await service.getCharacteristic(RX_CHAR_UUID);

                // Add event listener for characteristic value changes
                rxCharacteristic.addEventListener('characteristicvaluechanged', handleValueChanged);

                // Start notifications so the characteristic can send updates
                await rxCharacteristic.startNotifications();

                console.log('Notifications started for RX characteristic');
            } else {
                console.error('Bluetooth service not available.');
            }
        } catch (error) {
            console.error('Error initializing BluetoothReceiver:', error);
        }
    }

    $: {
        listen($service)
    }
</script>

<!-- Display the received value in the UI -->
<div>
    <h3>Received Data</h3>
    <p>{$receivedValue}</p>
</div>

<style>
    div {
        margin-top: 20px;
    }
    h3 {
        font-size: 1.2em;
    }
    p {
        font-family: monospace;
        color: #333;
    }
</style>
