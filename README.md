# VLora Node-RED nodes

This package contains [VLora](https://www.vioneta.com/) nodes for
using with [Node-RED](https://nodered.org/).

![example flow](https://raw.githubusercontent.com/Vioneta/node-red-contrib-vlora/main/example_flow.png)

## Installation

Please read the [Adding nodes to the palette](https://nodered.org/docs/user-guide/runtime/adding-nodes)
documentation for information on installing third-party nodes to Node-RED.

Example command:

```bash
npm install node-red-contrib-vlora
```

## Usage example

This usage example creates an echo flow, which returns the received uplink back
to the device.

### MQTT in node (Node-RED)

After the `node-red-contrib-vlora` package has been installed,
the first thing you must do is create a MQTT subscription using the _mqtt in_
node.

- Add the _mqtt in_ node to your flow.
- Under properties select _Add new mqtt-broker..._ and click the pencil icon.
- Enter the hostname of the server, when using TLS the port-number is usually `8883`, else `1883`.
- When using TLS make sure the _Use TLS_ option is checked, select _Add new tls-config..._ and click the pencil button.
  - If the MQTT broker is configured with client-certificate authentication and authorization,
    retrieve the certificate using the _Generate certificate_ within the VLora Application
    Server web-interface. Then upload the retrieved CA certificate, client-certificate and private
    key. Uncheck _verify server certificate_ as the server-certificate will already be validated
    using the provided CA certificate. The click _Add_.
- Click once more _Add_, this takes you back to the initial form.
- As _Topic_ enter `application/+/device/+/event/+` (in case you have modifications to the default
  VLora configuration, modify this topic if needed).
- Click _Done_.

After deploying the flow containing the _mqtt in_ node, you should see a green
bullet under the node with the status _connected_. If that is not the case,
validate the hostname, port and TLS configuration (if appliable).

### Device event node (VLora)

After setting up the _mqtt in_ node, add one or multiple _device event_ nodes.
For each node you can select the _Event Type_. For this usage example select
_Uplink_. Connect this node with the _mqtt in_ node.

### Debug node (Node-RED)

Add a _debug_ node and connect it with the _device event [up]_ node to log the
`msg.payload` for debugging.

### Echo function node (Node-RED)

Add a _function_ node, and add the following code to the _On Message_ event:

```js
return {
  payload: msg.payload.data,
  fPort: msg.payload.fCnt,
  confirmed: false,
  devEui: msg.payload.deviceInfo.devEui,
};
```

Connect this node with the _device event [up]_ node.

### Downlink node (VLora)

Add a _device downlink_ node and enter:

- The hostname:port of the server (e.g. `localhost:8080` or `example.com:443`).
  this is the same hostname and port which is used to access the VLora
  Application Server, but **without** the `http://` or `https://`.
- Select _Use TLS_ when the endpoint is secured with a TLS certificate.
- Generate an API Key in the VLora Application Server and paste the
  token under _API Token_.
- For this example set the _Payload Encoding_ to _Base64_, as the uplink payload
  is encoded within the JSON payload as Base64.

Connect this node with the echo function node.

### Debug node (Node-RED)

Add an other _debug_ node and connect it with the downlink node. This will
print the downlink frame-counter as debugging information. Make sure the
_Output_ is set to _complete msg object_.
