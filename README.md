# Galil Controller

Galil is a manufacturer of [motion control solutions](http://www.galilmc.com/motion-controllers). 

This package uses sockets to communicate with the controller over an Ethernet 
connection and has been tested on several DMC 41x3 "Econo" motion controllers 
but may also work with other models.

<img
src="http://www.galil.com/sites/default/files/products/dmc-41x3_big_0.png" alt="Galil" />

This software, while highly functional, is being provided under an MIT open 
source license, and thus is provided as-is without warranty of any kind.

### Setting Up

Configure your Galil controller through its configuration object. The 
following keys are required to run it:

```
{
  "galil": {
    "connection": {
      "host": "192.168.1.3",
      "port": 23,
    },
    "defaultTimeout": 10000
  }
}
```

- 'galil.connection.host' must be of type `String`
- 'galil.connection.port' must be of type `Number`
- 'galil.defaultTimeout' must be of type `Number`

You can change these with a setter:

```
> Galil.config.connection = { port: 25 };
{
  "galil": {
    "connection": {
      "host": "192.168.1.3",
      "port": 25,
    }
  }
}
```

Typically you will install a program on the controller using Galil's
GalilTools software and execute subroutines within your program:

```
test.dmc

#mySub
MG "mySub:Start"
WT 5000
MG "Waited five seconds."
MG "mySub:End"
```

In this example, we've provided some additional messages at the beginning and
end of the subroutine to help the package signal when the subroutine has
completed, as it is otherwise an asynchronous operation.

To execute this subroutine using GalilTools, you'd use the XQ firmware command:

```
XQ #mySub
```

### Interacting with the Controller

The Galil package exports with a collection `Galil.collection`. This
will be written to whenever a message is received and has the following
format.

{
  "socket": "messages",
  "message": "mySub:Start",
  "timestamp": new Date(),
  "type": "mySub"
}

- `socket` is the socket that received this message. It could be either
  `messages` or `commands`.
- `message` is the _parsed_ data received from the data event.
- `timestamp` is the time that it was added to the collection.
- `type` is the type of message it was. If you delimit your messages with
  something, it will be the 0th index.  

### Responding to Events

On the server, `Galil` extends EventEmitter. That means you can emit and
respond to your own custom events:

```
Galil._messages.addListener('connect', () =>
this.emit('messages_connected'));

Galil.on('messages_connected', () => {
  console.log('The message socket is now connected!');
});
```

### Synchronous Execution

`Galil.execute` will send an execute command and wait for a message saying the 
event is done. You can configure the end message by setting the configuration 
variable of `routine_end`.

```
Galil.config.parser.routine_end = 'End';
```

You can also set a default timeout. If no data is received during this
period of time, an error will be thrown.

```
Galil.config.defaultTimeout = 60 * 1000;  // one minute without data
throws an error
```

To execute a subroutine synchronously, pass the subroutine name you wish 
to execute:

```
// Usage on the client
Galil.execute('mySub', 60 * 1000, function () {
  console.log('Finished subroutine!')
});

// Usage on the server
Galil.execute('mySub', 60 * 1000);
```
