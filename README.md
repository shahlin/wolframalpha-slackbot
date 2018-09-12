# Wolfram Alpha Slackbot
A Slack bot that answers your simple or complex questions using Wolfram Alpha's API. The bot will be capable of answering questions like 'What is the current weather in New Zealand?' or even '7 = x - 5'. The limitation of the bot is that it cannot accept image inputs or retrieve visual data from the API. The example code can be customized to overcome those limitations.

## Setting up the bot
First things first, we'll need to create an app on Slack, get the bot token, install the app to the workspace, etc. It's just a one time configuration to basically allow us to install a bot to our workspace and get a reference to our bot so we can program it. Thankfully, there's a complete tutorial on that! So head over to the link below, complete the tutorial and we'll continue from there.

 **Tutorial Link : https://www.fullstackpython.com/blog/build-first-slack-bot-python.html**
 
I've gone ahead and changed the tutorial code a bit. But the change is not a requirement. You can continue with the code from the tutorial.

```python
import time
import re
from slackclient import SlackClient


# instantiate Slack client
slack_client = SlackClient("your-bot-access-token-goes-here")

# starterbot's user ID in Slack: value is assigned after the bot starts up
starterbot_id = None

# constants
RTM_READ_DELAY = 1 # 1 second delay between reading from RTM
EXAMPLE_COMMAND = "President of South Africa?"
MENTION_REGEX = "^<@(|[WU].+?)>(.*)"

def parse_bot_commands(slack_events):
    """
        Parses a list of events coming from the Slack RTM API to find bot commands.
        If a bot command is found, this function returns a tuple of command and channel.
        If its not found, then this function returns None, None.
    """
    for event in slack_events:
        if event["type"] == "message" and not "subtype" in event:
            user_id, message = parse_direct_mention(event["text"])
            if user_id == starterbot_id:
                return message, event["channel"]
    return None, None

def parse_direct_mention(message_text):
    """
        Finds a direct mention (a mention that is at the beginning) in message text
        and returns the user ID which was mentioned. If there is no direct mention, returns None
    """
    matches = re.search(MENTION_REGEX, message_text)
    # the first group contains the username, the second group contains the remaining message
    return (matches.group(1), matches.group(2).strip()) if matches else (None, None)

def handle_command(command, channel):
    """
        Executes bot command if the command is known
    """
    # Default response is help text for the user
    default_response = "Not sure what you mean. Try *{}*.".format(EXAMPLE_COMMAND)

    # Finds and executes the given command, filling in response
    response = None
    
    # This is where you start to implement more commands!
    if command.startswith(EXAMPLE_COMMAND):
        response = "Sure...write some more code then I can do that!"

    # Sends the response back to the channel
    slack_client.api_call(
        "chat.postMessage",
        channel=channel,
        text=response or default_response
    )

if __name__ == "__main__":
    if slack_client.rtm_connect(with_team_state=False):
        print("SmartBot connected and running!")
        # Read bot's user ID by calling Web API method `auth.test`
        starterbot_id = slack_client.api_call("auth.test")["user_id"]
        while True:
            command, channel = parse_bot_commands(slack_client.rtm_read())
            if command:
                handle_command(command, channel)
            time.sleep(RTM_READ_DELAY)
    else:
        print("Connection failed. Exception traceback printed above.")

```

So now, you should be having a python file with code similar to the one above. At this point, you should be able to use your Slack bot even though it is not going to reply anything useful. But make sure, it does reply something.

## Setting up Wolfram Alpha app
You'll need an account in Wolfram Alpha to be able to use the API.

**Create An Account : https://www.wolframalpha.com/pro/subscribe/signup.html**

**Open Developer Portal : http://developer.wolframalpha.com/portal/myapps/**

**Wolfram Alpha Documentation for Reference : https://products.wolframalpha.com/api/documentation**

Follow the steps below :
- Click on **Get an AppID** on top right
![Get AppID Button](https://imgur.com/eljDBvO.png "Get an AppID")

- Enter your app's name and description

 ![Get AppID Button](https://imgur.com/gSnHPK0.png "Enter App details")

- Upon success, your app will be given an **APP ID** which we will use later in the code
![Get AppID Button](https://imgur.com/sm5y2DP.png "App ID")

## Using Wolfram Alpha API
To be able to make use of the Wolfram Alpha API, we first need to install the following library `wolframalpha`. We can use **pip** to install it quickly. The following steps will guide you on how to start using the Wolfram Alpha API.

**NOTE :** All of the code below will be written in the same file containing the code for the Slack bot.

- Install the library :
`pip install wolframalpha`
[Visit the package page for reference](https://pypi.org/project/wolframalpha/)

- Once it is installed, we can just import the package in our app and make use of the API.
    ```python
    import wolframalpha
    ```

- We need to connect the app we created on the Wolfram Alpha Developer Portal to our Python application. This is where the **APPID** comes into play. Copy the APPID from the Wolfram Alpha Developer Portal website and paste it in the code below.
    ```python
    wolfram = wolframalpha.Client('your-appid-goes-here')
    ```

- With that done, we can start querying. To ask a question or to perform a query, we use the `query()` method which takes a string as the parameter. This string is the question/query input.
    ```python
    res = wolfram.query('President of USA?')
    ```
    This will return `wolframaplha.Result` object. We only need certain info from this object. We're only looking for the short answer and not details such as related topics, visualizations, etc. For example, if the query is 'President of USA?', we only want the answer to be Donald Trump.

- To get that result and output it, we use the following :
    ```python
    print(next(res.results).text)
    ```
    **NOTE :** If the input query is invalid, the StopIteration Exception will be raised and it needs to be caught.
    
## Adding Wolfram Alpha API to Slack Bot
Once the Wolfram Alpha API querying code is put in the Slack bot python file, it should look something like this :
**NOTE :** Do not forget to add you tokens accordingly
```python
import time
import re
from slackclient import SlackClient
import wolframalpha

# Connect to wolframalpha
wolfram = wolframalpha.Client('your-appid-goes-here')

# instantiate Slack client
slack_client = SlackClient("your-bot-access-token-goes-here")

# starterbot's user ID in Slack: value is assigned after the bot starts up
starterbot_id = None

# constants
RTM_READ_DELAY = 1 # 1 second delay between reading from RTM
EXAMPLE_COMMAND = "President of USA?"
MENTION_REGEX = "^<@(|[WU].+?)>(.*)"

def parse_bot_commands(slack_events):
    """
        Parses a list of events coming from the Slack RTM API to find bot commands.
        If a bot command is found, this function returns a tuple of command and channel.
        If its not found, then this function returns None, None.
    """
    for event in slack_events:
        if event["type"] == "message" and not "subtype" in event:
            user_id, message = parse_direct_mention(event["text"])
            if user_id == starterbot_id:
                return message, event["channel"]
    return None, None

def parse_direct_mention(message_text):
    """
        Finds a direct mention (a mention that is at the beginning) in message text
        and returns the user ID which was mentioned. If there is no direct mention, returns None
    """
    matches = re.search(MENTION_REGEX, message_text)
    # the first group contains the username, the second group contains the remaining message
    return (matches.group(1), matches.group(2).strip()) if matches else (None, None)

def handle_input(input, channel):
    """
        Provides an answer if the question is valid
    """
    # Default response is help text for the user
    default_response = "Not sure what you mean. Try *{}*.".format(EXAMPLE_COMMAND)

    # Variable containing response to the user's question
    response = get_answer(input) 

    # Sends the response back to the channel
    slack_client.api_call(
        "chat.postMessage",
        channel=channel,
        text=response or default_response
    )
 
def get_answer(question):
    """
        Returns the answer to the question asked
    """

    # Ask wolfram the question
    res = wolfram.query(question)

    # Return the results
    try:
        return next(res.results).text
    except StopIteration:
        return "No results found"


if __name__ == "__main__":
    if slack_client.rtm_connect(with_team_state=False):
        print("SmartBot connected and running!")
        # Read bot's user ID by calling Web API method `auth.test`
        starterbot_id = slack_client.api_call("auth.test")["user_id"]
        while True:
            input, channel = parse_bot_commands(slack_client.rtm_read())
            if input:
                handle_input(input, channel)
            time.sleep(RTM_READ_DELAY)
    else:
        print("Connection failed. Exception traceback printed above.")
```
