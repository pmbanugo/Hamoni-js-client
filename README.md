# Hamoni-js-client

JS client for Hamoni Chat API - https://www.producthunt.com/upcoming/hamoni.
Here's a [sample web app](https://github.com/pmbanugo/Hamoni-Example) utilising
Hamoni.

# How to use Hamoni JS client

Included in this repository is the Hamoni JS client located in
`lib/hamoni.min.js`. The client allows your app to connect to the Hamoni server.
To use it, copy and include this script in your web app. Once included, you will
have access to a global `Hamoni` variable which you will initialise. To
initiliase a new object, you'll need an applicationId and the identity of the
user using the client.

```
let hamoni = new Hamoni("APP_ID", "USER_IDENTITY"); //initialise Hamoni
```

In a real scenario, the indentity will be how you identify users in your
applications. It could be a username or their email. The appId is a way for
**Hamoni** server to identify your app and the client connecting upon which both
uses to interact. In the near future, a more secured way will be used and
probably it'll require token or cookie based authentication, but for now, the
goal is build an easy to use API for use on the client first, then add security
later (the project is being developed and I started work on it on November 2,
2017 to experiement with this idea and I'll be focusing on building the JS SDK
and securing the server, before moving to development of Android and iOS SDK).

Upon initialisation, the client tries to connect with the server. This happens
asynchronously. To know when it's done, you have to call
`Hamoni.ready(callback)` with a function to execute when it succesfully
connects.

```
hamoni.ready(() => {
    //Do whatever
    //it's similar to jQuery.ready function.
});
```

## Getting list of users

When the clent is connected, you can get a list of users in the application. You
do this by calling `Hamoni.getUsers()`

```
let users = await hamoni.getUsers()// ["janet", "damian", "gombe"]
```

This returns an array containing the identity of the users in your application,
except for the identity of the currently connected user. With this list, you can
pick any user and establish a connection to chat with them.

## One-on-One chat

To establish a One-on-One chat, you'll call `Hamoni.connectWithUser()` function.
It takes it 3 arguments:

1. The identity of the user to chat with. (You can get a list of users by
   calling `Hamoni.getusers()`).
2. A function to call when successfully connected.
3. A function to call when it fails to connect with that user.

When it succesfully establishes a connection with that user, it calls the
function passed as the second argument and passes it an object of type
`UserToUserConnection`. This object contains methods to send and receive
messages between the two users. See example below

```
hamoni.ready(() => {
    hamoni.connectWithUser(friend, chatSetupCompleted, chatSetupFailed);
});

function chatSetupFailed() {
    console.log('Chat Setup Failed. Contact Admin.');
}

function chatSetupCompleted(connection) {
    userToUserConnection = connection; //an object that describes the connection betwwen the users
    activateChatBox();
}
```

The `UserToUserConnection` has two functions, `send()` and `onNewMessage()`. To
send message to the othe user, you call `send("message to send") passing it the
message to send to the other user (at the moment you can only send text
messages. Later there'' be support for multimedia).

```
//send message
userToUserConnection.send(message);
```

To get notified when messages arrive, you pass a function to execute for every
new message to `onNewMessage()`. For every message received, it calls the passed
in function with an object that contains the message and the user who sent it

```
function addMessage(data) {
    let template = $("#new-message").html();
    template = template.replace(
        "{{body}}",
        `<b>${data.user}:</b> ${data.message}`
    );

    $(".chat").append(template);
}

userToUserConnection.onNewMessage(addMessage); //function to execute when a new message arrives
```

## Group chat

You can also enable group conversation for your apps. Hamoni provides API to
create or join groups.

### Create group

To create group, you call `Hamoni.createGroup()` with the name for the group
(unique across an application), display name, a success and failure callback.

```
hamoni.createGroup(
    { name: "plato", displayName: "It's plato's group" },
    group => {
        console.log("group created");

        group.send("Hello Prahtiba");

        group
        .getMembers()
        .then(members => console.log(members))
        .catch(error => console.log(error));
    },
    reason =>
        console.log("failed to create group. reason is") || console.log(reason)
);
```

When it fails to create the group, it calls the failure callback with the reason
it failed. If it was created, it calls the success callback with an object that
represents the group, which you'll use to send and receive message for that
group.

### Join group

To join a group, you need to know the name of the group. If you need to get a
list of group name, you call `Hamoni.getGroups()`

```
let groups = await hamoni.getGroups();
```

Once you have the name of the group, you can use it to join a group.

```
  hamoni.joinGroup(
    "plato",
    group => {
      console.log("group joined");
      group.onNewMessage(
        message => console.log(message)
      );

      group.send("hello new group");

      group
        .getMembers()
        .then(members => console.log("members are: ") || console.log(members))
        .catch(error => console.log("error occured") || console.log(error));
    },
    reason => console.log(reason)
  );
```

`getGroups` takes 3 arguments

* the name of the group
* a success callback which will receive an object that can be used to send and
  receive message from the group.
* a failure callback which get called when it couldn't add the client to a
  group.

### Group class

When you create or join a group, the success callback receives a **Group**
object. It contains 4 functions

1. **getMembers()** - This is used to retrieve the members in a group.

```
let members = await group.getMembers();
```

2. **send(message)** - Which is used to send message in the group.

```
group.send("Hello World");
```

3. **onNewMessage(callback)** - This is used to receive message sent to the
   group. It is passed a callback that gets called each time a new message
   arrives

```
group.onNewMessage(message => console.log(message));
```

4. **onNewMember(callback)** - This is used to get notified when a new member
   joins a group.

```
group.onNewMember(member => console.log(`${member} joined the group`));
```

5. **leaveGroup(successCallback, failureCallback)** - Removes the user from the
   group.

```
group.leaveGroup(()) => console.log("successfully left group", (error) => console.log(error)));
```
