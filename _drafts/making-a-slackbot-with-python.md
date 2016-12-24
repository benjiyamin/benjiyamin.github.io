---
layout: post
img: horatio-caine.jpg
category: Blog
title: Making a SlackBot with Python
subTitle: Faster Horatio Caine Memes!
---
One of my favorite internet memes is the [Horatio Caine One-liner](http://knowyourmeme.com/memes/puts-on-sunglasses-yeeeeaaahhh).
The standard joke consists of a setup, Horatio putting on his sunglasses, and then finally
finishing with a one-liner consistiing typically of a dad-joke-level pun.

For Example:

    The victim was found at the end of the road. It looks like.. 
    ( •__•)    ( •__•)>⌐■--■    (⌐■__■) 
    ..it was a dead end

My team and I love memes and such scatter through our work coordination so I thought it
might be fun to make a Horatio Caine joke generator, in the form of a SlackBot.

We'll being using the Slack API and Python to create a "horatiobot".
 
## Setting up a Proper Work Environment

We'll want to create a project folder, I called mine `/horatiobot/`. Within this will be our python file,`horatiobot.py`.
As always, we should set up a clean working environment, using virtualenv.

    $ virtualenv venv
    $ source venv/bin/activate

Installing the Slack API client is easy with pip. We can also `pip freeze` our requirements into a text file, in case
we need to recreate our environment

    $ pip install slackclient
    $ pip freeze > requirements.txt

__Note: At this point, my directory looked something like this:__

    horatiobot/
        venv/
        horatiobot.py
        requirements.txt
        
## Creating a Bot Integration

Lorem Ipsum

## Time to code our program

First, we need to use the slack client API to retrieve our bot's ID from it's username along with a call to retrieve 
usernames within your channel.

    from slackclient import SlackClient
    
    
    SLACK_BOT_TOKEN = os.environ.get('SLACK_BOT_TOKEN')
  
    BOT_NAME = 'horatiobot'
    
    
    def get_bot_id(name, call):
        """Retrieves a bot's ID string."""
        if call.get('ok'):
            users = call.get('members')
            for user in users:
                if 'name' in user and user.get('name') == name:
                    return user.get('id')
            raise Exception('Could not find bot user with the name ' + name)
        raise Exception('API call is invalid')
        
    if __name__ == '__main__':
        api_call = slack_client.api_call('users.list')
        bid = get_bot_id(BOT_NAME, api_call)

The next step is to create our functions that will parse and handle to excution of horatiobot's commands. We've set
up the logic to account for the format `@horatiobot [input 1] [input 2]` in several ways:

1. If two inputs of strings follow horatiobot, they are treated as set up, and punchline, respectively.

2. If one input string follows horatiobot, it is treated as the punchline with the set up "I guess you could say".

3. For any other case, horatiobot produces a random (but awesome) joke.


Another

    import os, time, shlex, random
    
    from slackclient import SlackClient
    
    
    BOT_NAME = 'horatiobot'
    SLACK_BOT_TOKEN = os.environ.get('SLACK_BOT_TOKEN')
    READ_WEBSOCKET_DELAY = 1  # 1 second delay between reading from firehose
    
    
    def at_bot(bot_id):
        """Retrieves a bot's @ string."""
        return '<@' + bot_id + '>'
    
    
    def parse_slack_output(slack_rtm_output, bot_id):
        """This parsing function returns None, None unless a message is directed at the Bot."""
        if slack_rtm_output:
            for output in slack_rtm_output:
                if 'text' in output and at_bot(bot_id) in output['text']:
                    command = output['text'].split(at_bot(bot_id))[1].strip()
                    channel = output['channel']
                    return command, channel
        return None, None
    
    
    def handle_command(command, channel):
        """Receives commands directed at the bot and determines if they are valid commands.
    
        It so, then acts on the command. If not, returns a random message.
    
        """
        command_list = shlex.split(command)
        if len(command_list) == 1:
            set_up = 'I guess you could say'
            punchline = command_list[0]
        elif len(command_list) == 2:
            set_up = command_list[0]
            punchline = command_list[1]
        else:
            set_up, punchline = random.choice(RANDOM_LINES)
        response = '%s.. \n ( •__•)    ( •__•)>⌐■--■    (⌐■__■) \n ..*%s*' % (set_up, punchline)
        slack_client.api_call('chat.postMessage', channel=channel, text=response, as_user='true')
    
    
    def connect_and_listen(bot_id):
        """The main loop for horatiobot to listen for command events."""
        if slack_client.rtm_connect():
            print('horatiobot connected and listening!')
            while True:
                command, channel = parse_slack_output(slack_client.rtm_read(), bot_id)
                if channel:
                    handle_command(command, channel)
                time.sleep(READ_WEBSOCKET_DELAY)
        else:
            raise Exception('Connection fails. Invalid Slack token or bot ID?')
    
    
    if __name__ == '__main__':
        api_call = slack_client.api_call('users.list')
        bid = get_bot_id(BOT_NAME, api_call)
        connect_and_listen(bid)