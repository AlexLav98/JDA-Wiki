# Getting started
Before you continue, make sure you've [setup a project](../setup/index.md) in the IDE of your choice.

## Creating a Discord Bot
1. Go to https://discordapp.com/developers/applications/ and login if you didn't already.
2. Click on "New Application" on the top right.
3. Give your application a name. This will be used as the bot's initial name (Can be altered later).
4. Click on "Create".
5. Click on the "Bot" tab at the left side.
6. Click the button "Add Bot" and confirm your choice by clicking on "Yes, Do it!"
    1. Your bot has been created now
	2. You can change the name, add/edit the avatar and set the bot to public/private.
7. Your bot is now ready to be added to Discords.

## Adding your Bot to a Discord server
1. On your application, click on the "OAuth2" tab on the left side.
2. Under the "Scopes" section, check "Bot" (Center column).
3. Select the permissions that you want to give to your bot.  
It's highly recommendet, to **not** give Administrator!
4. Click on "Copy" in the text box, to copy the URL into your clipboard.
5. Open the copied URL in a new browser tab.
6. Select the server where you want to add your Bot and click "Next".  
This requires you to at least have "Manage Server" permissions.
    1. If you set permissions does now a new section open, where I can (un)check permissions.
7. Click on "Authorize" and solve the captcha to confirm that you're not a bot.
8. Congratulations! Your bot should now be on the selected Discord.

## Connecting to Discord with the Bot account.

!!! warning "Important"
    **Never** show the bot token to other people as they can take over control of your bot with it!
	
1. On your application, click on the "Bot" tab on the left side.
2. Under the "Token" section, click on the "Copy" button, to copy the token into your clipboard.  
Do NOT click on "Regenerate" as this will recreate your token and invalidate the old one.
3. Setup a JDA project as described [here](../setup/index.md).
4. Create a JDA instance using the `JDABuilder` and your bot token:  
```java
// JDA can throw InterruptedException or LoginException on start.
public static void main(String[] args) throws LoginException, InterruptedException{
    /*
	 * It is recommendet to have the token setup in a separate
	 * file like a config file, to prevent accidental sharing
	 * on platforms like GitHub.
	 *
	 * Your Bot token WILL be invalidated automatically, if you
	 * share it on GitHub!
	 */
    JDA jda = JDABuilder
	    .createDefault("YourTokenHere")
		.build();
}
```

## Setup a simple ping command
1. Setup a [JDA instance](#connecting-to-discord-with-the-bot-account).
2. Create a separate class, which `extends ListenerAdabter`. Example:  
```java
public class PingCmd extends ListenerAdabter{

}
```
3. Add the `MessageReceivedEvent` to the class (Make sure the spelling is right!) and fill it like this:  
```java
public class PingCmd extends ListenerAdabter{

    // names of the voids are always "onX(XEvent)"
    @Override
    public void onMessageReceived(MessageReceivedEvent event){
	    /*
		 * We ignore bots, which includes our own.
		 *
		 * getAuthor() gives us the User (author) of the message
		 * This is only available for message-based events.
		 */
		if(event.getAuthor().isBot())
		    return;
		
		Message message = event.getMessage(); // Getting the message send in this event.
		
		/*
		 * getContentRaw() gives us the raw message, without any formatting.
		 * This means that f.e. mentiones could look like this: <@123456789012345678>
		 *
		 * Other options are getContentDisplay(), which would give the message as
		 * it would look like on Discord (e.g. Mention shows as @User)
		 * or getContentStripped() which is like getContentDisplay() but removes
		 * any markdown formatting.
		 */
		String content = message.getContentRaw();
		
		if(content.equalsIgnoreCase("!ping")){
		    MessageChannel channel = event.getChannel();
			channel.sendMessage("Pong!").queue(); // Remember to use .queue() when you use RestActions!
		}
	}
}
```
4. Register your listeners either through the JDABuilder or JDA directly:  
```java

// Option 1: Through JDABuilder
public static void main(String[] args) throws LoginException, InterruptedException{
    JDA jda = JDABuilder
	    .createDefault("YourTokenHere")
		.addEventListeners(new PingCmd())
		.build();
}

// Option 2: directly through JDA
public static void main(String[] args) throws LoginException, InterruptedException{
    JDA jda = JDABuilder
	    .createDefault("YourTokenHere")
		.build()
		.awaitReady(); // Blocks thread until JDA is marked "ready"
	
	jda.addEventListeners(new PingCmd());
}














```