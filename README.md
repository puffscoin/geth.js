gpuffs.js
=======

Start and stop [gpuffs](https://github.com/puffscoin/go-puffscoin) from Node.js.

Usage
-----

```
$ npm install gpuffs
```
To use gpuffs.js, simply require it:
```javascript
var gpuffs = require("gpuffs");
```

### Starting and stopping gpuffs

gpuffs' `start` method accepts a configuration object, which uses the same flags as the gpuffs command line client.  (Here, the flags are organized into an object.)  Flags that are not accompanied by a value on the command line (for example, `--mine`) should be passed in as `{ flag: null }`.
```javascript
var options = {
    networkid: "420",
    port: 31313,
    rpcport: 11363,
    mine: null
};

gpuffs.start(options, function (err, proc) {
    if (err) return console.error(err);
    // gpuffs active!!
});
```
The callback's parameter `proc` is the child process, which is also attached to the `gpuffs` object (`gpuffs.proc`) for your convenience.

When you are finished your session with gpuffs, `stop` kills the underlying gpuffs process:
```javascript
gpuffs.stop(function () {
    // gpuffs stopped. 
});
```

### Listeners

The callback for `start` fires after gpuffs has successfully started.  Specifically, it looks for `"IPC service started"` in gpuffs' stderr.  If you want to change the start callback trigger to something else, this can be done by replacing gpuffs' default listeners.  `gpuffs.start` accepts an optional second parameter which should specify the listeners to overwrite, for example:
```javascript
{
    stdout: function (data) {
        process.stdout.write("I got a message!! " + data.toString());
    },
    stderr: function (data) {
        if (data.toString().indexOf("Protocol Versions") > -1) {
            geth.trigger(null, gpuffs.proc);
        }
    },
    close: function (code) {
        console.log("Puff. Puff. PASS!!!!");
    }
}
```
In the above code, `gpuffs.trigger` is the callback passed to `gpuffs.start`.  (This callback is stored so that the startup trigger can be changed if needed.)  These three listeners (`stdout`, `stderr`, and `close`) are the only listeners which can be specified in this parameter, since only these three are set by default in `geth.start`.

If you want to swap out or add other listeners (after the initial startup), you can use the `gpuffs.listen` convenience method:
```javascript
gpuffs.listen("stdout", "data", function (data) { process.stdout.write(data); });
```
This example (re-)sets the "data" listener on stdout by setting `gpuffs.proc.stdout._events.data = function (data) { process.stdout.write(data); }`.

