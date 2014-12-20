# Imp Audio Protocol version 1.0

An audio processing standard based on independent DSP processes communicating via simple ZeroMQ sockets.

## Actors

### 1. Processor
A processor is an independent actor in an Imp network (either a process or a number of processes operating as a single unit). It can expose any number of input and output ports for other applications to connect to, and supplies a metadata file so these ports can be located.

#### 1.1 In/output ports

> Can receive: `process`  
> Will send: `process`

These are the objects between which audio is transferred in an Imp network. An application can connect to another's input port and supply it with audio in the form of `process` messages. In the same way, it can connect to another applications output port and start receiving `process` messages from this other application. In/output ports are implemented as ZeroMQ SUB/PUB sockets. Therefore, one should connect to an input port using a PUB socket (i.e. an output port) and vice versa. No use is made of the subscription tag feature; pub/sub sockets are mainly used so multiple processors can easily connect to a single output.

#### 1.1 Configuration port

> Can receive: `set-option`, `get-option`, `request-instance`  
> Will send: `result`

The configuration port is implemented using a ZeroMQ REP socket (so should be connected to using a REQ socket or equivalent).

### 2. Discovery hub

> Can receive: `discover`  
> Will send: multiple `result`

The discovery hub is a system-wide port that, when sent a `discover` message, will reply with a `result` message from each processor indicating which processor is available and where their configuration port is located. By default, the discovery hub is located at `tcp://ip-address:25685`. Processes wishing to broadcast their existence should connect to the hub using an XPUB socket, and send out an identifiying message as soon as they connect, and each time they receive a `discover` message. Programs wishing to discover others should connect to the hub using a XSUB socket, on which they will receive message from processes identifying themselves.

## Messages

### 1. Base message structure

Message objects are serialized using MessagePack, and always contain one property: `type`. This is a string containing one of the message type names detailed below.

### 2. `process`

A process message is a message containing audio data to be processed. It can contain the following properties:
 - `data`: A byte array containing the audio samples in the format specified, or if none is specified, the default format for the current port
 - `format`: Optionally, the format in which `data` is supplied

### 3. `set-options`

 - `port`: The port to apply the option to
 - `format`: The new default format for this port

### 4. `get-options`

 - `port`: The port to get the option from
 - `options`: The names of the options requested

### 5. `result`

 - `operation`: The type of the message this is a result of

### 6. `request-instance`

### 7. `discover`



