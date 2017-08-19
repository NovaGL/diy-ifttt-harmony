# DIY IFTTT for Harmony
IFTTT Harmony commands for Google Assistant and Alexa

This is a DIY Harmony project to allow Alexa or Googgle Assistant to change to a specific Foxtel Channel by voice.

An example would be "Ok Google, watch ABC" or "Alexa, trigger, watch ABC"

This process has several preequistes that might not make it possible or advisable to use for some users.
There are offical methods that maybe better for some users, but this method gives me greater flexibility.
If you have any suggestions please create an issue.

- A device capable of running Node.JS (Raspberry PI, Windows, OSX, Linux)
- [IFTTT Account](http://ifttt.com)
- [BST](https://github.com/bespoken/bst)  (this will receive the actions from IFTTT)
- [PM2](https://www.npmjs.com/package/pm2) (used for keeping BST in memory)
- [Node-Red](http://nodered.org) (this will peform the actions that IFTTT sends out)
- [Harmony-API](https://github.com/maddox/harmony-api) (This sends the actions to your hub)

### Node-Red Installation

Get Node-Red using the following command 

```bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)```

This should install Node.JS which is used for BST and PM2

## Harmony API

Follow the setup instructions on the GitHub page.
Open the webpage eg: localhost:8282.

It will give you a list of hub names that we will use later. Eg. my-room

## BST Proxy setup
**Make sure you are using at least v1.0.7 for better security with a key in the url**
**If v1.0.7 is not out yet add @1.0.7 to end of below command**

Installed BST using `npm install bespoken-tools -g`
To test BST Proxy we use the following command after BST is installed.
```bst proxy http 1880```

It will then say something similar to.
```Your public URL for accessing your local service:```
```https://myskill.bespoken.link?bespoken-key=XXXXXXXX-XXXX-XXXX-XXXX-XXXX-XXXXXXXX```

This URL will be used for our webhook commands in the IFTTT section.

## PM2 Setup

To make this run in background we use PM2.
```npm install pm2 -g```

Create a directory to store the BST command in.

Eg make a directory called "bst" 
Store a file in there called proxy.sh
In that file save the same command as above ```bst proxy http 80 --secure```

Now to save that command to BST, we do the following in the same directory as the script.

```pm2 start proxy.sh --name="bst-proxy"```
```pm2 startup```

This will save the script under the name "bst-proxy" and make sure it starts on boot.


## IFTTT

What we want to do with ifttt is to create a trigger using either Alexa or Google Assistant depending on your platform.

Choose what you want to say. (If you have multiple rooms, it's best to include the room name)

#is for number $ is for text

### "Say a phrase with a text ingredient" (Selecting channel by name)

Title| Action
------------ | -------------
What do you want to say? | Channel $
What's another way to say it? (optional) | Watch $
And another way? (optional)| Switch to $
What do you want the Assistant to say in response?| Changing to $

### "Say a phrase with a number" (Selecting channel by number)

Title| Action
------------ | -------------
What do you want to say? | Channel #
What's another way to say it? (optional) | Watch #
And another way? (optional)| Switch to #
What do you want the Assistant to say in response?| Changing to #

### Make a web request
Next we want to choose the "Webhook" Action
The URL is the one we got from BST and add IFTTT to the end 

Title| Action
------------ | -------------
URL | https://[myskill].bespoken.link?bespoken-key=XXXXXXXX-XXXX-XXXX-XXXX-XXXX-XXXXXXXX/IFTTT
Method| POST
Content Type| application/json
Body | {"command": "{{TextField}}","type": "FavoriteChannelRequest","hub": "my-room","ip": "localhost:8282"}

#### Body Values
The body of the request is broken down into several values to be used to determine the action you want to perform

Key| Value
------------ | -------------
command | TextField or NumberField (click add ingredient)
type | FavoriteChannelRequest or ChannelRequest
hub | Your hub's name, get from Harmony-API if unsure.
ip | the ip number and port for Harmony-API (no need for http)

On the next step give it a descriptive name and save.

## Node-Red Flow
Once you have Node-Red up and running. Copy harmony-ifft.json to clipboard.

Menu > Clipboard > Import > paste the example and press deploy.

This should create a HTTP endpoint which will accept HTTP POST requests at /IFTTT

Now assuming that everything is setup correctly, you should be able to say "Ok Google, switch to ABC" and the channel will change.

Feel free to modify the list to suit your needs.
