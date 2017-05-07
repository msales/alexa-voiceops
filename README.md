# VoiceOps with Alexa - Start/Stop EC2 Instances on AWS

## Introduction

This sample project demonstrates how to use [Amazon Alexa](https://developer.amazon.com/alexa) to do various tasks on
your [AWS Account](https://aws.amazon.com/)  using your Amazon Echo (dot).

This example and code is brought to you by [msales - convert everything](http://www.msales.com)


## Lambda Function

The python-based Lambda function is organized into intent handler which represent the different functions. These
 handlers are called using some boilerplate code. The first part of the template will route the incoming request based 
 on its type. The type will either be LaunchRequest, IntentRequest, or SessionEndedRequest and this function will be 
 triggered whenever we interact with Alexa. The event parameter will have all the information we need about the given 
 request.

**AWS Lambda Python Boiler Plate for Custom Alexa Skills**

The following code shows a minimal boilerplate to create your lambda-based Alexa skill:

```
def lambda_handler(event, context):
    """
    Route the incoming request based on type (LaunchRequest, IntentRequest,
    etc.) The JSON body of the request is provided in the event parameter.
    
    :param event: 
    :param context: 
    :return: 
    """
	
    if event['session']['new']:
        on_session_started({'requestId': event['request']['requestId']}, event['session'])

    if event['request']['type'] == "LaunchRequest":
        return on_launch(event['request'], event['session'])

    elif event['request']['type'] == "IntentRequest":
        return on_intent(event['request'], event['session'])

    elif event['request']['type'] == "SessionEndedRequest":
        return on_session_ended(event['request'], event['session'])
	
	
def on_session_started(session_started_request, session):
    """
    Called when the session starts
    
    :param session_started_request: 
    :param session: 
    :return: 
    """

    print("on_session_started requestId=" + session_started_request['requestId']
          + ", sessionId=" + session['sessionId'])
		  

def on_launch(launch_request, session):
    """
    Called when the user launches the skill without specifying what they want
    
    :param launch_request: 
    :param session: 
    :return: 
    """
	
	...
	
	
def on_intent(intent_request, session):
    """
    Called when the user specifies an intent for this skill 
    
    :param intent_request: 
    :param session: 
    :return: 
    """

    print("on_intent requestId=" + intent_request['requestId'] +
          ", sessionId=" + session['sessionId'])

    intent = intent_request['intent']
    intent_name = intent_request['intent']['name']

    # Dispatch to your skill's intent handlers
    if intent_name == "StopTaggedInstances":
        return function_for_handling_the_intent(intent, session)
		
    elif intent_name == "AMAZON.HelpIntent":
        return intent_help_function(intent, session)
		
	elif intent_name == "AMAZON.StopIntent":
		return intent_stop_function(intent, session)
	
    else:
        raise ValueError("Invalid intent")

def on_session_ended(session_ended_request, session):
    """
    Called when the user ends the session.
    Is not called when the skill returns should_end_session=true
    
    :param session_ended_request: 
    :param session: 
    :return: 
    """
    print("on_session_ended requestId=" + session_ended_request['requestId'] +
          ", sessionId=" + session['sessionId'])

	  
def build_speechlet_response(title, output, reprompt_text, should_end_session):
    """
    Build Speechlet Response
    
    :param title: 
    :param output: 
    :param reprompt_text: 
    :param should_end_session: 
    :return: 
    """
    return {
        'outputSpeech': {
            'type': 'PlainText',
            'text': output
        },
        'card': {
            'type': 'Simple',
            'title': 'SessionSpeechlet - ' + title,
            'content': 'SessionSpeechlet - ' + output
        },
        'reprompt': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': reprompt_text
            }
        },
        'shouldEndSession': should_end_session
    }


def build_response(session_attributes, speechlet_response):
    """
    Build the Response
    
    :param session_attributes: 
    :param speechlet_response: 
    :return: 
    """
    return {
        'version': '1.0',
        'sessionAttributes': session_attributes,
        'response': speechlet_response
    }
```

As you can see every intent will be mapped to a python function call. The response at the end is simply a dictionary 
that tells Alexa with which text to response and if the session should be ended. 


### Our Lambda function

The code of the Lambda function for the skill can be found in *src/aws_alexa_handler.py*. The single function will be 
explained briefly in the following.

The function contains two constants that are used within the intents:

* **ALEXA_AWS_DEFAULT_TAG**: The tag of the EC2 Instances that should be filtered.  
* **AWS_DEFAULT_REGION**: The default AWS Region which is used within the skill.


#### StartTaggedInstances

Starts all stopped EC2 instances in the current region containing the tag *ALEXA_AWS_DEFAULT_TAG* with the value *True*.


#### StopTaggedInstances

Stops all started EC2 instances in the current region containing the tag *ALEXA_AWS_DEFAULT_TAG* with the value *True*.


#### GetRunningTaggedInstances

Returns the number of running instances in the current region containing the tag *ALEXA_AWS_DEFAULT_TAG* with the 
value *True*.


#### SetCurrentRegion

Sets the AWS region of the current session to interact with.


#### GetCurrentRegion

Returns the current AWS region of the session.


#### AMAZON.HelpIntent

Returns some help for the VoiceOps Skill.


#### GetSomeFun

Just a fun output function which simply returns speech only without any further functionality.


## How-to setup the Lambda function to be used as an Alexa Skill

### Intent Schema
Alexa needs to know what intents your skill should contain. This done by providing a JSON document that describes your 
intents. Further you are able to define so called slots. Slots are variables with a list of values that can be
 referenced in the interaction with the user. In our cases we define the *AWS_REGION_LIST* slot which contains a lit of
 all supported AWS regions within this skill. Our intent schema can be found in *src/intent.json*.


### Custom Slot Types 
As already mentioned slots represent variables with pre-defined values the user can choose to interact with your
 intents. The values for our slot *AWS_REGION_LIST* can be found in *src/AWS_REGION_LIST.txt*.
 

### Sample Utterances 

Sample utterances provide Alexa some examples to find the right intent when a user is using your skill. You can use the 
defined slots within these utterances. In our case we use {Region} to reference a value of our slot *AWS_REGION_LIST*.
Our utterances can be found in *src/sample_utterances.txt*


## So what? Let us tell you how you can test te skill

Since you already have an AWS account ;) you should get an [Amazon Developer](https://developer.amazon.com/alexa) 
Account too to be able to create your skill. Please click after the loging on "Get Started" at the Alexa Skills Kit.


![Alexa Skill Welcome Page](https://github.com/msales/alexa-voiceops/raw/master/screenshots/0_screenshot_alexa_skill.png)


On the Alexa Skills Kit overview just click "Add a New Skill" to get started. On the following screenshot we already 
filled all necessary things for our "VoiceOps" Skill. Please an appropriate invocation name which will be used to 
activate your skill later. The form offers you a link to some help how to choose your right invocation name. We tried 
"AWS" at the beginning which does not work maybe because it's a brand or reserved name. That is the reason why we have 
chosen "cloud" as our invocation name.


![Skill information](https://github.com/msales/alexa-voiceops/raw/master/screenshots/0_screenshot_skill_information.png)


Next you have to configure the interaction model of your skill. As mentioned before a skill contains a JSON-based intent 
definition, optionally some custom slots with values and some sample utterances.
 

![Interaction model](https://github.com/msales/alexa-voiceops/raw/master/screenshots/1_screenshot_interaction_model.png)


The next step contains the configuration of your skill. You can setup here some endpoints or the Amazon Resource Names 
(ARNs) in order to call your lambda function. The ARN of your Lambda function can be found at the right side of the 
detail overview of your function. 


![Lambda ARN Location](https://github.com/msales/alexa-voiceops/raw/master/screenshots/2a_screenshot_ARN_from_lambda.png)


Just copy the ARN and select the nearest location of your audience - or just you ;). One note to the Lambda function, 
currently the Alexa Tools Kit Gateway is only supported in the following regions:


* us-east-1 US East (N. Virginia)
* eu-west-1 EU (Ireland)


![Skill Configuration](https://github.com/msales/alexa-voiceops/raw/master/screenshots/2_screenshot_configuration.png)


Having the skill configured you need to provide Amazon some publishing information - in case you wanna make your skill 
public available. In our case we filled everything but at the end we just test the skill using the beta test feature.


![Publishing Information](https://github.com/msales/alexa-voiceops/raw/master/screenshots/3_screenshot_publishing_information.png)


Before your gonna be able to test your skill on your Echo using the beta test feature, you have provide some privacy 
information too. In our case we set everything to "no" since we are not going to collect some data or address people 
below the age of 13 years.


![Privacy and Compliance](https://github.com/msales/alexa-voiceops/raw/master/screenshots/4_screenshot_privacy_and_compliance.png)


Now you will be able to activate the "Beta Test" in order to test your skill on your Echo. Just choose the email of your 
Account that is setup with your Echo. Your will get an email or simply call the Skill Url directly to add it into your 
account.


![Privacy and Compliance](https://github.com/msales/alexa-voiceops/raw/master/screenshots/5_screenshot_beta_test.png)


The skill will then appear in your Skill overview.


![Skill Overview](https://github.com/msales/alexa-voiceops/raw/master/screenshots/6_screenshot_skill_overview.png)


Now lets have some fun with saying "Alexa, start cloud" ;). Feel free to ask Robert Hoppe <robert.hoppe@msales.com> of 
msales if you have any questions.


## Licence / Copyright

The code of this Alexa Skill / AWS Lambda function is brought to you by msales / Robert Hoppe and released und the 
MIT Licence.