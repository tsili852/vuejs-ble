<template>
  <div class="test container">
      <h1 class="text-primary">{{title}}</h1>
      <p class="text-warning">v{{version}}</p>
      <form v-on:submit="getDevices" class="form form-inline">
        <input v-model="message" class="form-control" id="message" type="text" placeholder="Message to the module">
        <br>
        <br>
        <input class="btn btn-success" type="submit" value="Connect and Write">
      </form>
      <br>
      <div id="output-panel" class="panel panel-default" hidden>
        <div class="panel-heading">
          Output
        </div>
        <div id="output" class="panel-body">
          <div id="content"></div>
          <div id="status" class="text-danger"></div>
          <pre id="log" style="text-align:left"></pre>
        </div>
      </div>
  </div>
</template>

<script>
    export default {
        name: 'test',
        data() {
            return {
                title: 'ArkPos2 BLE Configuration',
                user: {
                  firstName: 'Nick',
                  lastName: 'Tsilivis'
                },
                message: '',
                version: '0.7.2'
            }
        },
        mounted: function () {
          if (!navigator.bluetooth) {
            this.setStatus('Web Bluetooth API is not available.\n');
          }
        },
        methods: {
          getDevices: function(e) {
            e.preventDefault();

            this.clearLog();

            if (navigator.bluetooth) {

              var bluetoothDevice;

              let filters = [];

              // let serviceUuid = document.querySelector('#service').value;
              let serviceUuid = "0x00FF"
              if (serviceUuid.startsWith('0x')) {
                serviceUuid = parseInt(serviceUuid);
              }
              // this.log('Service UUID: ' + serviceUuid);

              let characteristicUuid = "0xFF01";
              // let characteristicUuid = document.querySelector('#characteristic').value;
              if (characteristicUuid.startsWith('0x')) {
                characteristicUuid = parseInt(characteristicUuid);
              }

              let message = document.querySelector('#message').value;

              // this.log('Characteristic UUID: ' + characteristicUuid);

              this.log('Requesting Bluetooth Device...');

              navigator.bluetooth.requestDevice({
                filters: [{
                  services: [
                    serviceUuid]
                  }]
                })
              .then(device => {

                bluetoothDevice = device;

                this.log('-> Name:      ' + device.name);
                // this.log('-> ID:        ' + device.id);

                bluetoothDevice.addEventListener('gattserverdisconnected', this.onDisconnected);

                return device.gatt.connect()
              })
              .then(server =>{
                // this.log('Getting Service');
                return server.getPrimaryService(serviceUuid);
              })
              .then(service => {
                // this.log('Getting characteristic');
                if (characteristicUuid) {
                  return service.getCharacteristic(characteristicUuid);
                }
              })
              .then(characteristic => {
                let encoder = new TextEncoder('utf-8');
                this.log("Encoded Message: " + encoder.encode(message));
                return characteristic.writeValue(encoder.encode(message));
              })
              .then(_ => {
                this.log("Message successfully sent !");
              })
              // .then(characteristics => {
              //   console.log(characteristics);
              //   this.log('> Characteristics: ' + characteristics.map(c => c.uuid).join('\n' + ' '.repeat(19)));
              // })
              .catch(error => {
                this.log('Error: ' + error);
              });
            }
            else {
              alert('No Bluetooth detected');
            }

          },
          onDisconnected: function (e) {
            let device = event.target;
            this.log('Device ' + device.name + ' is disconnected.');
          },
          log: function () {
            var line = Array.prototype.slice.call(arguments).map(function(argument) {
              return typeof argument === 'string' ? argument : JSON.stringify(argument);
            }).join(' ');

            document.querySelector('#log').textContent += line + '\n';

            $('#output-panel').show();
          },
          clearLog: function() {
            document.querySelector('#log').textContent = '';
            $('#output-panel').hide();
          },
          setStatus: function(status) {
              document.querySelector('#status').textContent = status;
              $('#output-panel').show();
          },
          setContent: function () {
            var content = document.querySelector('#content');
            while (content.hasChildNodes()) {
              content.removeChild(content.lastChild);
            }
            content.appendChild(newContent);

            $('#output-panel').show();
          }
        }
    }
</script>

<style scoped>

</style>
