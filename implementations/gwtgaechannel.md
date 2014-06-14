# GWT + GAE Channel

Real-time collaborative TodoMVC app made with Google Web Toolkit and Google App Engine.


## [Demo](http://gwttodomvc.appspot.com)


## Snippets

From `server.service.CommandServiceImpl.java`:

```java
// This method executes commands on the server and then broadcasts the command
// to all listening clients. ChannelService.sendMessage is from GAE Channel API.
@Override
public Command<?> executeCommand(Command<?> command, String originClientId) {
    Command<?> updateCommand = executor.execute(command);
    String commandMessage = serializer.serializeCommand(updateCommand);
    checkState(!editors.isEmpty(), "There must be at least one editing client");
    for (String editor : editors) {
        if (!editor.equals(originClientId)) {
            channelService.sendMessage(new ChannelMessage(editor, commandMessage));
        }
    }
    return updateCommand;
}
```

The command will be handled in the client by `client.command.CommandController.java`:

```java
private class CommandChannelListener implements SocketListener {

    @Override
    public void onOpen() {
        logger.info("channel open");
    }

    @Override
    public void onMessage(String message) {
        try {
            Command<?> command = deserializer.read(message);
            logger.fine("received command: " + command);
            executor.execute(command);
        } catch (CommandSerialization.SerializationException e) {
            logger.severe("unable to de-serialize message: " + message);
        }
    }

    @Override
    public void onError(ChannelError error) {
        logger.severe("channel error: " + error.getCode() + " : " + error.getDescription());
    }

    @Override
    public void onClose() {
        logger.severe("channel closed: will not receive further commands from the server");
    }
}
```

Note that [client](http://dprotti.github.io/gwttododoc/java/com/todomvc/client/command/todo/ClientToDoCommandExecutor.java.html) and [server](http://dprotti.github.io/gwttododoc/java/com/todomvc/server/command/todo/ServerToDoCommandExecutor.java.html) executors differ since they load objects in a different way.


## Goal

Demonstrate how to implement a real-time collaborative TodoMVC app using GWT.


## Design patterns used

- MVP (Model-View-Presenter) to handle UI interactions
- Command Pattern to share changes among clients and the server. Commands carry only deltas (changes) to objects and not whole objects.


## Relationship to other solutions/technologies

GWT eases client-to-server communication but doesn't have built-in support for real-time collaboration. This demo uses GWT serialization facilities to serialize Commands and GAE Channel API to server-push Commands to clients so as to orchestrate real-time collaboration.


## Outstanding features

Supports real-time collaboration.


## Key Concepts

- [Command Pattern](http://en.wikipedia.org/wiki/Command_pattern): separates the action to be executed, from the execution (who and when) of the action, and both from the target object upon which the action finally executes (allows execution of same action on both client *and* server, by loading different objects in each case).
- GWT Serialization: its `RPC.encodeResponseForSuccess()` method, originally designed to encode responses from `RemoteService`'s, it's being used instead as a general serializer: we use it to serialize Commands sent to clients through GAE channels.
- [GAE Channel API](https://developers.google.com/appengine/docs/java/channel/): used to create a persistent connection between client app and server, allowing the server to send messages to JavaScript client apps in real time without the use of polling. A perfect fit here and a better choice than polling since in TodoMVC the updates can't be predicted: they come from human users.


## Starting year

2014


## Author

[Duilio Protti](https://github.com/dprotti/)


## Dependencies

GWT 2.6.0, appengine-api-1.0-sdk 1.7.7, gwt-gae-channel 2.0.0, Guava 16.0.1, Guice 3.0, Gin 2.1.2, JUnit 4.8.1, Mockito 1.8.4.


## Community

Issues, comments and PRs welcomed at the [project site](https://github.com/dprotti/todomvc/tree/gwtgaechannel/labs/architecture-examples/gwt-gaechannel).


## Articles & Guides

- Implementation is heavily influenced and partially based on Justin Fagnani's [10 minutes presentation](http://www.youtube.com/watch?v=wWhd9ZwvCyw&t=29m44s) in Google I/O 2011.
