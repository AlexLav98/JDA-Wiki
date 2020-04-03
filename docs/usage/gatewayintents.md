[GatewayIntent]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/GatewayIntent.html

[jdasetenabled]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/JDABuilder.html#setEnabledIntents(net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)
[shardsetenabled]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/sharding/DefaultShardManagerBuilder.html#setEnabledIntents(net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)

[jdasetdisabled]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/JDABuilder.html#setDisabledIntents(net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)
[shardsetdisabled]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/sharding/DefaultShardManagerBuilder.html#setDisabledIntents(net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)

[GUILD_MEMBERS]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/GatewayIntent.html#GUILD_MEMBERS
[GUILD_PRESENCES]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/requests/GatewayIntent.html#GUILD_PRESENCES

[chunkingfilter]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/utils/ChunkingFilter.html#NONE

[create]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/sharding/DefaultShardManagerBuilder.html#create(java.lang.String,net.dv8tion.jda.api.requests.GatewayIntent,net.dv8tion.jda.api.requests.GatewayIntent...)
[default]: https://ci.dv8tion.net/job/JDA/javadoc/net/dv8tion/jda/api/sharding/DefaultShardManagerBuilder.html#createDefault(java.lang.String)

# Gateway Intents
Gateway Intents are Discord's way of giving the developers a option to choose, which information their bot needs and which they don't need.  
This helps reducing traffic and resource intense operations as your bot no longer needs to listen and look for any possible action, as the API simply won't send those when not needed.

The Gateway Intents were introduced in version 6 of the Discord REST API and will be mandatory in version 7.  
This page covers some parts you may want to know about Gateway Intents and some common questions like "Which one will I need for my bot?"

## Usage in JDA
Gateway Intents are implemented into JDA as their own enum called [GatewayIntent] and can be used when building/starting the bot.

### Enabling Intents
You can use the [`JDABuilder#setEnabledIntents(GatewayIntent, GatewayIntent...)`][jdasetenabled] or [`DefaultShardManagerBuilder#setEnabledIntents(GatewayIntent, GatewayIntent...)`][shardsetenabled] method to set the **enabled** Gateway Intents.  
This will automatically disable all not set Intents.

### Disabling Intents
If you only want to disable a selected few Intents can you use [`JDA#setDisabledIntents(GatewayIntent, GatewayIntent...)`][jdasetdisabled] or [`DefaultShardManagerBuilder#setDisabledIntents(GatewayIntent, GatewayIntent...)`][shardsetdisabled] to do so.  
It is **required** to disable [Prvileged Intents](#privileged-intents) if you don't have them enabled in your developer-dashboard.

### Using defaults
Both the `JDABuilder` and the `DefaultShardManagerBuilder` offer a method called `createDefault(String)` which sets the token for the Bot and enables/disables specific Intents for the bot.  
When using this will JDA set the following things (Might change in the future):

- `setMemberCachePolicy(MemberCachePolicy)` is set to `MemberCachePolicy.DEFAULT`
- `setChunkingFilter(ChunkingFilter)` is set to `ChunkingFilter.NONE`
- `setEnabledIntents(Collection)` is set to `GatewayIntent.DEFAULT`
- The CacheFlags `CacheFlag.ACTIVITY` and `CacheFlag.CLIENT_STATUS` will be **diabled**

### Set based on used events
The GatewayIntent enum of JDA has a method called `fromEvents(Class...)` which allows you to set intents based on what Events your bot is using.  
The provided classes need to extend the GenericEvent class (i.e. through extending the ListenerAdabter) for this to work.

## Privileged Intents

!!! warning "Important"
    Bots that are on less than 100 Guilds won't require whitelisting.  
	Bots which are on more than 100 Guilds and aren't whitelisted, only have access to 100 Guilds.
	
	**Those privileged intents are currently available for all Bots without requirement of whitelisting. This may change in the future (Date: 01. Apr 2020)**

Discord has two Gateway Intents which are "privileged", meaning they require Discord to whitelist you, to access them.  
Those intents are [GUILD_MEMBERS] (Receive information about members) and [GUILD_PRESENCES] (Receive information about precense updates).

Your bot **will break** when you try to access those without being whitelisted.  

You need to also set your ChunkingFilter to [`ChunkingFilter.NONE`][chunkingfilter] when GUILD_MEMBERS isn't used.

## Changes to JDABuilder and DefaultShardManagerBuilder
The `JDABuilder` and `DefaultShardManagerBuilder` received some major changes with the introduction of Gateway Intents.  
The Constructors themself (`JDABuilder(String)`/`DefaultShardManagerBuilder(String)`) are now **deprecated** and it is recommendet to use the `create(String, GatewayIntent, GatewayIntent...)` instead.

??? note "Example JDABuilder"
    ```java
    // Old method
    public static void main(String[] args) throws LoginException{
	    // args[0] is the token of the bot.
        new JDABuilder(args[0])
	        .build();
    }
    
    // New method
    public static void main(String[] args) throws LoginException{
	    // args[0] is the token of the bot.
        JDABuilder
		    .create(args[0], GatewayIntent.GUILD_MESSAGES, GatewayIntent.GUILD_MESSAGE_REACTIONS)
		    .build();
    }
    ```

??? note "Example DefaultShardManagerBuilder"
    ```java
    // Old method
    public static void main(String[] args) throws LoginException{
	    // args[0] is the token of the bot.
        new DefaultShardManagerBuilder(args[0])
	        .build();
    }
    
    // New method
    public static void main(String[] args) throws LoginException{
	    // args[0] is the token of the bot.
        DefaultShardManagerBuilder
	        .create(args[0], GatewayIntent.GUILD_MESSAGES, GatewayIntent.GUILD_MESSAGE_REACTIONS)
		    .build();
    }
    ```

Note that you can also use [`DefaultShardManagerBuilder.createDefault(String)`][default] to setup default Gateway Intents and settings.

## Recommendet Intents
Here is a list of recommendet Intents that should be set.

### All bots
For all bots is it recommendet to have `GUILD_MESSAGES` enabled, to receive messages on the Guilds (For command execution).  
If your bot also allows commands to be used through DM, is `DIRECT_MESSAGES` required.

### Music Bots
`GUILD_VOICE_STATES` is required for tracking Voice state changes (i.e. Member joining/leaving a Voice chat), but also to **connect to the voice chat yourself**.  
Bots without this intent cannot join voice chats.

### Giveaway Bots
If your Giveaway Bot is using Reactions for its Giveaways will it require the `GUILD_MESSAGE_REACTIONS` intent to listen for Reaction events (Adding/Removing reactions).

### Moderation Bots
Bots which perform Moderation tasks like kicking or banning Members should have the `GUILD_BANS` intent to keep track of kicks/bans performed by other moderators/bots.  
Additionally if your bot also tracks invite creation/deletion as a moderation feature is `GUILD_INVITES` required.