<template>
  <div class="test">
      <h1>{{title}}</h1>
      <form v-on:submit="getDevices">
        <label for="allDevices">All Devices</label>
        <input type="checkbox" id="allDevices">
        <input id="service" type="text" size="17" list="services" placeholder="Bluetooth Service">
        <input id="name" type="text" size="17" placeholder="Device Name">
        <input id="namePrefix" type="text" size="17" placeholder="Name Prefix">
        <input type="submit" value="Get Devices">
      </form>
      <datalist id="services">
        <option value="alert_notification">alert_notification</option>
        <option value="automation_io">automation_io</option>
        <option value="battery_service">battery_service</option>
        <option value="blood_pressure">blood_pressure</option>
        <option value="body_composition">body_composition</option>
        <option value="bond_management">bond_management</option>
        <option value="continuous_glucose_monitoring">continuous_glucose_monitoring</option>
        <option value="current_time">current_time</option>
        <option value="cycling_power">cycling_power</option>
        <option value="cycling_speed_and_cadence">cycling_speed_and_cadence</option>
        <option value="device_information">device_information</option>
        <option value="environmental_sensing">environmental_sensing</option>
        <option value="generic_access">generic_access</option>
        <option value="generic_attribute">generic_attribute</option>
        <option value="glucose">glucose</option>
        <option value="health_thermometer">health_thermometer</option>
        <option value="heart_rate">heart_rate</option>
        <option value="human_interface_device">human_interface_device (blacklisted)</option>
        <option value="immediate_alert">immediate_alert</option>
        <option value="indoor_positioning">indoor_positioning</option>
        <option value="internet_protocol_support">internet_protocol_support</option>
        <option value="link_loss">link_loss</option>
        <option value="location_and_navigation">location_and_navigation</option>
        <option value="next_dst_change">next_dst_change</option>
        <option value="phone_alert_status">phone_alert_status</option>
        <option value="pulse_oximeter">pulse_oximeter</option>
        <option value="reference_time_update">reference_time_update</option>
        <option value="running_speed_and_cadence">running_speed_and_cadence</option>
        <option value="scan_parameters">scan_parameters</option>
        <option value="tx_power">tx_power</option>
        <option value="user_data">user_data</option>
        <option value="weight_scale">weight_scale</option>
      </datalist>
      <h3>Output</h3>
      <div id="output" class="output">
        <div id="content"></div>
        <div id="status"></div>
        <pre id="log"></pre>
      </div>
  </div>
</template>

<script>
    export default {
        name: 'test',
        data() {
            return {
                title: 'BLE Test',
                user: {
                  firstName: 'Nick',
                  lastName: 'Tsilivis'
                }
            }
        },
        mounted: function () {
          if (/Chrome\/(\d+\.\d+.\d+.\d+)/.test(navigator.userAgent)){
            if (45 > parseInt(RegExp.$1)) {
              this.setStatus('Warning! Keep in mind this sample has been tested with Chrome ' + 45 + '.');
            }
          }

          if (!navigator.bluetooth) {
            this.setStatus('Web Bluetooth API is not available.\n');
          }
        },
        methods: {
          getDevices: function(e) {
            e.preventDefault();

            this.clearLog();

            if (navigator.bluetooth) {
              let filters = [];

              let filterService = document.querySelector('#service').value;
              if (filterService.startsWith('0x')) {
                filterService = parseInt(filterService);
              }
              if (filterService) {
                filters.push({services: [filterService]});
              }

              let filterName = document.querySelector('#name').value;
              if (filterName) {
                filter.push({name: filterName});
              }

              let filterNamePrefix = document.querySelector('#namePrefix').value;
              if (filterNamePrefix) {
                filters.push({namePrefix: filterNamePrefix});
              }

              let options = {};
              if (document.querySelector('#allDevices').checked) {
                options.acceptAllDevices = true;
              } else {
                options.filters = filters;
              }

              this.log('Requestiong Bluetooth Device...');
              this.log('with ' + JSON.stringify(options));
              navigator.bluetooth.requestDevice(options)
              .then(device => {
                log('-> Name:      ' + device.name);
                log('-> ID:        ' + device.id);

                return device.gatt.connect()
              })
              .then(server =>{
                log('-> Connected: ' + device.gatt.connected);
              })
              .catch(error => {
                log('Argh! ' + error);
              });
            }
            else {
              alert('No Bluetooth');
            }

          },
          log: function () {
            var line = Array.prototype.slice.call(arguments).map(function(argument) {
              return typeof argument === 'string' ? argument : JSON.stringify(argument);
            }).join(' ');

            document.querySelector('#log').textContent += line + '\n';
          },
          clearLog: function() {
            document.querySelector('#log').textContent = '';
          },
          setStatus: function(status) {
              document.querySelector('#status').textContent = status;
          },
          setContent: function () {
            var content = document.querySelector('#content');
            while (content.hasChildNodes()) {
              content.removeChild(content.lastChild);
            }
            content.appendChild(newContent);
          }
        }
    }
</script>

<style scoped>

</style>
