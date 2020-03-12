[map]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#map%28java.util.function.Function%29
[flatmap]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#flatMap%28java.util.function.Function%29
[delay]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/RestAction.html#delay%28java.time.Duration%29

[scheduler]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html
[schedule]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html#schedule-java.lang.Runnable-long-java.util.concurrent.TimeUnit-

# RestActions
RestActions where introduced in JDA 3.0 and are since then a key part of the project.  
The RestAction can be seen as a terminal between the JDA user and the Discord REST API, handling the requests about what your bot wants to do and executing it.  
It allows you to specify *how* JDA should deal with their requests.

That said, will you need to actually "tell" the RestAction to *do something*. It is recommendet to always check if something in JDA is returning a RestAction.  
If it does, then you have to execute it using one of the below listed operations:

- [queue(), queue(Consumer) or queue(Consumer, Consumer)](#asynchronous)  
These operations are run asynchronous, meaning they won't execute in the same Thread.  
This means you can't use procedural logic when using `queue()`, unless you use the callback Consumer (Click the link for more info).
- [complete()](#blocking)  
Calling this operation will **block** the current Thread until the request has finished and returns the response type.
- [submit()](#cancelable)  
Provides request future to cancel tasks later and avoid callback hell.
- [xAfter(...)](#planning)  
`queueAfter()`, `completeAfter()` and `submitAfter()` provide similar functionalities as their normal counterparts, but with the ability to delay the execution.

!!! info "Note"
    We recommend using `queue()` or `submit()` when possible as blocking the current Thread could cause unwanted side-effects.

Since JDA 4.1.1 can you use additional methods to prevent a so called "Callback hell" with `queue()`.
- [`map`][map] Convert the result of the `RestAction` to a different value.
- [`flatMap`][flatmap] Chain another `RestAction` on the result.
- [`delay`][delay] Delay the element of the previous step.

## AuditLog Reasons
Some operations return a special `RestAction` implementation called `AuditableRestAction`.  
This extension allows to set a reason field for that action.

These reasons are only available for accounts of `AccountType.BOT` but will silently be ignored for any non-bot accounts.  
However, any account type can use reasons on kicks and bans with the special overloads provided that allows setting a reason.

**Example**:  
```java
public class ModUtil{
    
	public static void deleteMessage(Message message, String reason){
	    message.delete().reason(reason).queue();
	}
	
	public static void banMember(Guild guild, User user, String reason){
	    guild.ban(user, 7, reason).queue();
	}
}
```

## Asynchronous

### `queue()`
The most common way for executing a RestAction is to simply call `queue()`:  
```java
public void sendMessage(MessageChannel channel, String message){
    channel.sendMessage(message).queue();
}
```

Note that this RestAction might be finished before other RestActions which used queue.  
For example: Your bot opens a DM with a user and then sends a message in said DM. Due to the nature of async methods can it happen that sending the message is finished first, causing an error, when the channel isn't available.

You can use one of the below mentioned variants of queue for fixing this, or use the [`flatMap`][flatmap] option added in JDA 4.1.1!

### `queue(Consumer)`
`queue(Consumer<T>)` allows you to execute a RestAction using queue and then performing additional actions once this has finished.

#### Example: Sending a DM
You may want to send a DM from your bot to a user to f.e. send the command help.  
Using queue is the recommendet way here, but can cause unwanted side effects as sending the DM could happen before the DM was even opened.

This is where the `success callback` (Or `Success Consumer`) comes into play!  
Let's first make a little example with our DM thing:  
```java
public void sendDM(User user, String message){
    user.openPrivateChannel().queue(new Consumer<PrivateChannel>(){ // queue returns a PrivateChannel object
	    @Override
		public void accept(PrivateChannel channel){
		    channel.sendMessage(message).queue();
		}
	});
}
```

"But that looks really ugly!" will you now say, right?  
Don't worry, there is a solution for this. Since JDA requires you to use Java 8, can you use something quite useful: Lambda expressions!  
```java
public void sendDM(User user, String message){
    // We call the callback "channel" here and use it as a reference.
    Consumer<PrivateChannel> callback = (channel) -> channel.sendMessage(message).queue();
    user.openPrivateChannel().queue(callback); // callback will execute the above Consumer.
}
```

But wait! You can even simplify it more:  
```java
public void sendDM(User user, String message){
    // We set "channel" as reference, which is an instance of Consumer<PrivateChannel>
    user.openPrivateChannel().queue((channel) -> channel.sendMessage(message));
}
```

### `queue(Consumer, Consumer)`
"But what if the user doesn't allow DMs?" For that does `queue(Consumer<T>, Consumer<Throwable>)` exist!  
The first Consumer is used on a successfull action, while the second one is used on a failed one.  
Note, that in this case, you have to use it for the queue of sending the message, as opening a Private channel rarely fails.

!!! info "Note"
    With JDA 4.1.1 where [`map`'s][map] and [`flatMap`'s][flatmap] added, which improve handling cases like the above example with sending a DM a lot.

## Blocking

### `complete()`
The `complete()` method is simply for convenience as it does nothing more than calling [`queue()`](#queue) and blocking the Thread.  
It is highly recommendet to use `queue()` or `submit()` if you don't use return values or need to wait for an action to finish before others.

## Cancelable

### `submit()`
Sometimes do you want to cancel a request before it gets executed, to prevent errors.  
This can be quite challenging when using [`queue()`](#queue) or [`complete()`](#complete).  
In `submit()` does JDA provide a `CompletableFuture` (aka promise) which allows you to cancel it when not needed.

If you don't need a `CompletableFuture` you may want to use `queue()` instead!

#### Example: Change channel settings
```java
// With just using queue()
public void setTestChannel(TextChannel channel){
    channel.getManager().setName("test-channel").queue((v) ->
	    channel.sendMessage("Update channel").queue((m) ->
		    m.delete().queueAfter(30, TimeUnit.SECONDS, (t) ->
			    logger.info("Deleted response in " + channel.getName());
			)
		)
	)
}

// Using submit() instead
public void setTestChannel(TextChannel channel){
    channel.getManager().setName("test-channel").submit()
	       .thenCompose((v) -> channel.sendMessage("Update Channel").submit())
		   .thenCompose((m) -> m.delete().submitAfter(30, TimeUnit.SECONDS))
		   .thenCompose((t) -> logger.info("Deleted response in " +  channel.getName()))
		   .whenComplete((s, error)
		       // This is called on termination (success/failure)
			   // If the result is successfull is error null
		       if(error != null) error.printStackTrace();
		   )
}
```

!!! info "Note"
    With JDA 4.1.1 can you now use [`RestAction#flatMap`][flatmap] to achieve similar results with methods like `queue()`

## Planning
The `xAfter()` are also known as **Planned Execution** as they use a [ScheduledExecutorService][scheduler] to [schedule] calls to either [`complete()`](#complete) or [`queue()`](#queue).

!!! info "Info"
    With the exception of `completeAfter()` can you provide a own ScheduledExecutorService as the last argument to any of the methods.

### `queueAfter(long, TimeUnit)`
Sends a delayed [`queue()`](#queue) action.

### `queueAfter(long, TimeUnit, Consumer)`
Sends a delayed [`queue(Consumer<T>)`](#queueconsumer) action.

### `queueAfter(long, TimeUnit, Consumer, Consumer)`
Sends a delayed [`queue(Consumer<T>, Consumer<Throwable>`](#queueconsumer-consumer) action.

----

### `completeAfter(long, TimeUnit)`
Sends a delayed [`complete()`](#complete) action.  
Note that this is the only `xAfter()` method, that doesn't provide an option to use a own ScheduledExecutorService.

----

### `submitAfter(long, TimeUnit)`
Creates a `DelayedCompletableFuture<T>` which will hold the response type as its generic value.  
This means that using `get()` on the returned Future will cause the curren thread to block and await the execution of the RestAction and receive the response type.