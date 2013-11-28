Modem.js, GSM Modems on Node
============================
> Modem.js allows you to use your GSM modems on node.
It offers a very simple API.
It supports:
* Sending SMS messages
* Receiving SMS messages
* Getting delivery reports
* Deleting messages from memory
* Getting notifications when memory is full
* Getting signal quality
* Making ussd sessions
* 16bit (ucsd messages)
* 7bit (ascii) messages
* Multipart messages

Installation
------------
```bash
npm install modem```

Instantiate
-----------
```javascript
var modem = require('modem').Modem()```

Open modem
----------
```javascript
modem.open(device, callback)```

* device `String` Path to device, like `/dev/ttyUSB0`
* callback `Function` called when modem is ready for further action

Send a message
--------------
```javascript
modem.sms(message, callback)```

* message `Object`
    * text `String` message body. Longs messages will be splitted and 
        sent in multiple parts transparently.
    * msisdn `String` receiver number.
    * encoding `String`. '16bit' or '7bit'. Use 7bit in case of English messages.

callback `Function` is called with `false` as first argument in case of failure.
    If sending message(s) is succeeded, it will receive message id of each sent
    part. (Which could be used to gettind delivery reports)


Get delivery reports
--------------------
```javascript
modem.on('delivery', callback)```

* callback `Function` is called with the following arguments:

* details `Object` detailed status report
    * smsc `String` Msisdn of SMS Sender
    * reference `String` Reference number of the delivered message
    * sender  `Msisdn of receiver`
    * status `Delivery status`

* message_id `String` message_id which has been delivered
This should be obtained from callback of modem.sms()

Receive a message
-----------------
```javascript
modem.on('sms received', callback)```

* callback `Function` will be called on each new message with following arguments:
* message `Object`
    * smsc `String` MSISDN of SMS Center
    * sender `String` MSISDN of sender
    * time `Date` Send time
    * text  `String` message body

Get stored messages
-------------------
```javascript
modem.getMessages(callback)```
* callback `Function` will be called with a single argument
  messages `Array`, which contains stores messages

Delete a message
----------------
```javascript
modem.deleteMessage(message_index, callback)```

* message_index `Int` is the message index to be deleted
* callback `Function` called when message is deleted

Get notified when memory is full
--------------------------------
```javascript
modem.on('memory full', callback)```
* callback `Function` will be called when modem has no more space
for further messages

Running custom AT commands
==========================
> Modem.js allows you to run AT commands in a queue and manage them without messing the modem.  
API is still quite simple.

Run a command
-------------
```javascript
job = modem.execute(at_command, [callback], [priority], [timeout])```

* at_command `String` AT command you would like to execute
* callback `Function` called when execution is done, in form of `(escape_char, [response])`
    * escape_char `String` could be 'OK', 'ERROR' or '>'.
    * response `String` modem's response
* prior `Boolean` if set to true, command will be executed immediately
* timeout `Integer` timeout, in milliseconds, defaults to 60 seconds.
* job `EventEmitter` represents the added job.
    * it will emit `timeout` if executing job times out

USS Sessions
============
> Modem.js allows you ro tun ussd sessions.

Instantiate
-----------
```javascript
var Session = require('modem').Ussd_Session```

Create a session
----------------
```javascript
var Session = require('modem').Ussd_Session
var CheckBalance = function(c) {
    var session = new Session;
    session.callback = c;

    session.parseResponse = function(response_code, message) {
        this.close();

        var match = message.match(/([0-9,\,]+)\sRial/);
        if(!match) {
            if(this.callback)
                this.callback(false);
            return ;
        }


        if(this.callback)
            this.callback(match[1]);

        session.modem.credit = match[1];
    }

    session.execute = function() {
        this.query('*141*#', session.parseResponse);
    }

    return session;
}```
