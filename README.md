# DISCORDJS-EASYCOMMANDS

## Features

**Global**
- Load commands **recursively** from a given directory.
- Allow you to **create** a **command** **quickly**, **without difficulties** and in **object-oriented** programming by extending the Command class.
- Create many (or one!) **subcommand**(s) in a Command.

**Commands**

- Command **name** and **description**
- Command **category**
- Command **args** and **usage**
- Command **guildOnly**
- Command **adminOnly** and customizable **permissions**
- Command **cooldowns**
- Creating **subcommands** quickly

## Usage
### Main file (with Discord.js)
```js
const Discord = require('discord.js');
const { CommandHandler, Logger } = require('discordjs-easycommands');

// Load config from your config file
const config = {
    status: 'ONLINE',
    presence: 'GAME NAME',
    prefix: '!',
    token: 'YOUR_TOKEN'
}

const client = new Discord.Client();

client.on('ready', () => {
  try {
    CommandHandler.init('./commands', client).then(async () => {
      await client.user.setPresence({ game: { name: config.presence, status: config.status } });
      Logger.add('success', `Playing: ${client.user.presence.game}`);
      Logger.add('success', `Status: ${client.user.presence.status}`);
      Logger.add('success', `Connected to: ${client.guilds.size} guilds`);

    //   console.log(global.COMMANDS)
    });
  } catch (e) {
    Logger.add('error', e.stack);
  }
});

client.on('message', (message) => {
  try {
    CommandHandler.preExecute(message, config.prefix, (command, args) => {
        (command.isSubCommand) ? command.execute(message, args) : command.constructor.execute(message, args);
    });
  } catch (e) {
    Logger.add('error', e.stack);
  }
});

client.login(config.token);
```

### Creating an Example command
```js
const Command = require('discordjs-easycommands').Command;
const Discord = require('discord.js');

class ExampleCommand extends Command {
    constructor() {
        super();
        this
            .setName('example') // Required: Name of the command
            // Following is default values, you can change anything you'll need
            .setDescription('') // Description of the command. Usefull when using help command
            .setArgs(true) // Is this command require arguments ?
            .setUsage() // <example> <usage> @user
            .setGuildOnly(false) // Is this command available in dms ?
            .setAdminOnly(false)
            .setPermissions([]) // All permissions available: https://discord.js.org/#/docs/main/stable/class/Permissions?scrollTo=s-FLAGS
            .setCooldown(3);
            
        /** Subcommand is a little bit different from a parent Command. He's generated by the parent Command (example, here), you can change everyhing like
         * in the parent command but its super constructor is discordjs-easycommands.Command and not the parent command. So if you don't type anything for
         * a parameter, this parameter will equal the default value of Command. (eg: The cooldown of the following subcommand would be the 3 default value).
         * Finally, this following subcommand would be accessible by typing: {prefix}example say "A simple message to repeats"
         */
        this.createSubCommand({
            name: 'say', // Required
            description: 'Repeat yourself',
            args: true,
            usage: '<message>',
            guildOnly: true,
            permissions: ['SEND_MESSAGES'],
            execute: this.constructor.sayExecute // Required
        });

        // this.primaryColor = '#c27c0e';
        // this.dangerColor = '#FF0000';
    }

    // Command execution
    static execute(message, args) {
        message.reply('you called the Example command ? There is!');
    }

    /**
     * Subcommand execution. This is just an example because SAY isn't a real Subcommand of EXAMPLE. 
     * It's better to create a new SayCommand.js file for creating a say command.
     */
    static sayExecute(message, args) {
        const emb = new Discord.RichEmbed()
        .setColor(this.primaryColor)
        .setDescription(args.join(" "));

        message.channel.send(emb);
    }
}

module.exports = new ExampleCommand;
```
