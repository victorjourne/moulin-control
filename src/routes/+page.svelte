<script>
    import BluetoothClient from './BluetoothClient.svelte';
    import BluetoothSender from './BluetoothSender.svelte';
    import BluetoothReceiver from './BluetoothReceiver.svelte';
    import Slider from './Slider.svelte';

    let sender;

    function handleConnection(event) {
        console.log('Bluetooth device connected:', event.detail.connected);
    }

    function handleSliderInput(event) {
        const { id, value } = event.detail;
        console.log(`Slider ${id} value changed to ${value}`);
        sender.writeCharacteristic(`${id}:${value}`); // Send the value over Bluetooth
    }
</script>

<BluetoothClient on:connected={handleConnection}>
    <BluetoothSender bind:this={sender} />
    <BluetoothReceiver />
    <Slider id="1" min="0" max="100" on:input={handleSliderInput} />
    <Slider id="2" min="0" max="100" on:input={handleSliderInput} />
</BluetoothClient>
