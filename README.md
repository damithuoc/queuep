# <a href='http://queuep.netlify.com/'><img src='http://i.imgur.com/24TwZl4.png' height='100'></a>

[![Build Status](https://travis-ci.org/pupudu/queuep.svg?branch=master)](https://travis-ci.org/pupudu/queuep) 
[![Code Climate](https://codeclimate.com/github/pupudu/queuep/badges/gpa.svg)](https://codeclimate.com/github/pupudu/queuep)
[![Coverage Status](https://coveralls.io/repos/github/pupudu/queuep/badge.svg?branch=master)](https://coveralls.io/github/pupudu/queuep?branch=master)

Pronounced: "queue-pea" https://pupudu.gitbooks.io/queuep/content/ http://queuep.netlify.com/

An API which will be consumed by multiple clients will eventually reach a point when the resources
of the hosting server is insufficient to handle the incoming load. QueueP is a framework designed 
for congestion control in such scenarios to **avoid using resources unneccessarily** saving money
and time.
 
QueueP is more often useful for scenarios where redundant requests should be ignored. 
However, QueueP can be used in any other case where performance is affected by a heavy load of data.

### Why QueueP?
None of the similar queue libraries have a concept of avoiding duplicate requests
(At least from the ones I found). QueueP filters out redundant requests and allows the API to process useful
 data without requiring you to upgrade physical resources unnecessarily. 
 
 QueueP also allows you to customize the logic of deciding whether or not to process a data chunk via the 
 concept of dirty checkers. While you can write the dirty checker all by yourself, QueueP provides several
 configurable dirty checker templates which can be quite handy to use.  
 
### Documentation
We have started writing a gitbook to give the users a thorough understanding about the framework. 
The book is still not complete, but do visit  https://pupudu.gitbooks.io/queuep/content/ or http://queuep.netlify.com/  and have a look
to see where it is heading. We promise to finish it soon. 

### Installation
To install the stable version:

    npm install --save queuep

### Requirements
QueueP is in ES5 syntax meaning that you can directly use it in your webapps.
If using NodeJs, you will need NodeJs v4 or later.

### Basic Usage
Let's assume that 5000 devices are sending data to an endpoint about their online status.

First, initialize a queue(can have multiple queues) with the minimal required configurations. 
Constructor of a module is a good place to keep the initQueue code.

```js
import qp from 'queuep';

qp.initQueue("app_online_status", {
    consumer: updateOnlineStatus
});
```

Then you can publish data to the queue.

```js
qp.publish("app_online_status", deviceId, onlineStatus);
```

initQueue and publish method calls can be in the same module or in different modules. 
I personally prefer to keep the initQueue and publish methods in the same module. 
But that is completely up to you to decide.

* Note 1: The argument **id** is used to identify the queue. 
An application can have any number of queuep queues.

* Note 2: The argument **consumer** should be a function reference which should be either;

A function which accepts 3 arguments. 
First two arguments will give the key and the data published from the publish method. 
The 3rd argument is an error-first callback that should be called to signal QueueP that the consume task 
has finished.
See example:

```js
let updateOnlineStatus = (key, data, callback) => {

    let onlineStatus = data,
        deviceId = key;

    deviceDao.updateOnlineStatus(deviceId, onlineStatus, function (err) {
        if (err) {
            return callback(err)
        }
        return callback();
    });
}
```
    
A function which accepts 2 arguments and returns a promise. First 2 arguments are as same as above. 
The promise can be resolved or rejected to signal QueueP that the consume task has finished.
See example:

```js
let updateOnlineStatus = (key, data) => {

    let onlineStatus = data,
        deviceId = key;

    return new Promise((resolve, reject) => {
        deviceDao.updateOnlineStatus(deviceId, onlineStatus)
            .then(resolve)
            .catch(reject);
    });
}
```
    
### Background
I wrote queuep to fix an issue in a project I was working on. 
The story in brief and the fundamental advantages of using QueueP can be found at 
https://pupudu.gitbooks.io/queuep/content/background.html

### License
MIT
