---
title: Softbank Robotics NAOqi API Documentation *working progress*


language_tabs:
  - python
  - c++
  - android

toc_footers:
  - <a href='https://developer.softbankrobotics.com/'>Developer Portal</a>

includes:
  - errors


search: true
---

# Introduction
>In all following code examples, make sure to replace the IP address in default to your robot IP, or execute the python script with an --ip argument.
> If running on robot (Packaged and Installed) use the local Naoqi IP address indicated below

Welcome to NAOqi API! We are excited to get building awesome applications on our robots. The pre-requisite to use the APIs can be found <a href='http://doc.aldebaran.com/2-5/dev/python/install_guide.html'> here </a> and we assume you have installed the Python SDKs with paths added. If you choose a different programming language to Python, please also add the SDK for that specific language of your choice!

# Quick Start

## Make Pepper Listen and Speak
```python
import qi
import argparse
import sys
import time

def main(session):
    # Get the service ALSpeechRecognition.
    asr_service = session.service("ALSpeechRecognition")
    asr_service.setLanguage("English")
    # Example: Adds "yes", "no" and "please" to the vocabulary (without wordspotting)
    vocabulary = ["yes", "no", "please"]
    asr_service.setVocabulary(vocabulary, False)
    #You can say the above words to see reaction
    # Start the speech recognition engine with user Test_ASR
    asr_service.subscribe("Test_ASR")
    # Get the service ALTextToSpeech.
    tts = session.service("ALTextToSpeech")
    #Says a test std::string
    tts.say("Hello, speech recognition engine started")
    time.sleep(20)
    #unsubscribe
    asr_service.unsubscribe("Test_ASR")

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ip", type=str, default="10.80.129.90",
                        help="Robot IP address. On robot or Local Naoqi: use '127.0.0.1'.")
    parser.add_argument("--port", type=int, default=9559,
                        help="Naoqi port number")
    args = parser.parse_args()
    session = qi.Session()
    try:
        session.connect("tcp://" + args.ip + ":" + str(args.port))
    except RuntimeError:
        print ("Can't connect to Naoqi at ip \"" + args.ip + "\" on port " + str(args.port) +".\n"
               "Please check your script arguments. Run with -h option for help.")
        sys.exit(1)
    main(session)
```

This example uses both the ALSpeechRecognition (ASR) and ALTextToSpeech (TTS) API Modules.
It starts the speech recognition engine with a few words added to the dictionary,
and makes the robot say some text using the TTS module.

ASR module gives to the robot the ability to recognize predefined words or phrases in several languages, and relies on sophisticated speech recognition technologies provided by NUANCE.

<aside class="notice">
The parameter enableWordSpotting of ALSpeechRecognitionProxy::setVocabulary modifies the content of the returned result:
If true: Phrase_i contains <...>phrase<...> Where the markers <...> indicates garbage results of the speech recognition.
If false: Phrase_i contains the exact searched phrase.
</aside>

TTS module allows the robot to speak. It sends commands to a text-to-speech engine, and authorizes also voice customization. The result of the synthesis is sent to the robot’s loudspeakers.
<aside class="notice">
The output audio stream can be modified. For example, these effects are available:
pitch shifting effect modifies the initial pitch of the voice,
double voice effect produces a “delay/echo” effect on the main voice.
Additional parameters are available for microAITalk engine.
</aside>

 *add more technical description of the code and the two modules, etc.*

## Have a Conversation with Pepper

```python
import qi
import argparse
import sys

def main(session):
    # Getting the service ALDialog
    ALDialog = session.service("ALDialog")
    ALDialog.setLanguage("English")
    # writing topics' qichat code as text strings (end-of-line characters are important!)
    # You could put the topic in a separate file and load it using the absolute path
    topic_content_1 = ('topic: ~example_topic_content()\n'
                       'language: enu\n'
                       'concept:(food) [fruits chicken beef eggs]\n'
                       'u: (I [want "would like"] {some} _~food) Sure! You must really like $1 .\n'
                       'u: (how are you today) Hello human, I am fine thank you and you?\n'
                       'u: (Good morning Nao did you sleep well) No damn! You forgot to switch me off!\n'
                       'u: ([e:FrontTactilTouched e:MiddleTactilTouched e:RearTactilTouched]) You touched my head!\n')
    topic_content_2 = ('topic: ~dummy_topic()\n'
                       'language: enu\n'
                       'u:(test) [a b "c d" "e f g"]\n')
    # Loading the topics directly as text strings
    topic_name_1 = ALDialog.loadTopicContent(topic_content_1)
    topic_name_2 = ALDialog.loadTopicContent(topic_content_2)
    # Activating the loaded topics
    ALDialog.activateTopic(topic_name_1)
    ALDialog.activateTopic(topic_name_2)
    # Starting the dialog engine - we need to type an arbitrary string as the identifier
    # We subscribe only ONCE, regardless of the number of topics we have activated
    ALDialog.subscribe('my_dialog_example')
    try:
        raw_input("\nSpeak to the robot using rules from both the activated topics. Press Enter when finished:")
    finally:
        # stopping the dialog engine
        ALDialog.unsubscribe('my_dialog_example')
        # Deactivating all topics
        ALDialog.deactivateTopic(topic_name_1)
        ALDialog.deactivateTopic(topic_name_2)
        # now that the dialog engine is stopped and there are no more activated topics,
        # we can unload all topics and free the associated memory
        ALDialog.unloadTopic(topic_name_1)
        ALDialog.unloadTopic(topic_name_2)

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ip", type=str, default="10.80.129.90",
                        help="Robot's IP address. If on a robot or a local Naoqi - use '127.0.0.1' (this is the default value).")
    parser.add_argument("--port", type=int, default=9559,
                        help="port number, the default value is OK in most cases")
    args = parser.parse_args()
    session = qi.Session()
    try:
        session.connect("tcp://{}:{}".format(args.ip, args.port))
    except RuntimeError:
        print ("\nCan't connect to Naoqi at IP {} (port {}).\nPlease check your script's arguments."
               " Run with -h option for help.\n".format(args.ip, args.port))
        sys.exit(1)
    main(session)
```

This example sets up a topic following the qichat syntax, you could also load the topic in a .top file on the robot.

The ALDialog module allows you to endow your robot with conversational skills by using a list of “rules” written and categorized in an appropriate way.

###Rules
ALDialog uses a list of written rules in order to manage the flow of the conversation between the human and the robot.

These rules are of two types: User rules and Proposal rules.

A User rule links a specific user input to possible robot output.
<aside class="notice">
u: (Hello Nao how are you today) Hello human, I am fine thank you and you?
</aside>

A Proposal rule triggers a specific robot output without any user output beforehand.
<aside class="notice">
proposal: Have you seen that guy on the TV yesterday?
   u1: (yes) He was crazy, no?
   u1: (no) Really, I need to tell you.
</aside>

###Grouped by Topics
In order to properly manage the conversation between the human and the robot, the rules are grouped by Topics.

<aside class="notice">
topic: ~greetings
language: enu

u: (Hello Nao how are you today) Hello human, I am fine thank you and you?
u: ({"Good morning"} {Nao} did you sleep * well) No damn! You forgot to switch me off!
proposal: human, are you going well ?
   u1: (yes) I'm so happy!
   u1: (no) I'm so sad
</aside>

*give some examples of topic files with rules*

## Make Pepper Play an Animation

```python

import qi
import argparse
import sys

def main(session):
    # Get the service ALAnimationPlayer.
    animation_player_service = session.service("ALAnimationPlayer")
    # play an animation, this will return when the animation is finished
    animation_player_service.run("animations/Stand/Gestures/Hey_1")
    # play an animation, this will return right away
    future = animation_player_service.run("animations/Stand/Gestures/Hey_1", _async=True)
    # wait the end of the animation
    future.value()
    # play an animation, this will return right away
    future = animation_player_service.run("animations/Stand/Gestures/Hey_1", _async=True)
    # stop the animation
    future.cancel()

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ip", type=str, default="10.80.129.90",
                        help="Robot IP address. On robot or Local Naoqi: use '127.0.0.1'.")
    parser.add_argument("--port", type=int, default=9559,
                        help="Naoqi port number")
    args = parser.parse_args()
    session = qi.Session()
    try:
        session.connect("tcp://" + args.ip + ":" + str(args.port))
    except RuntimeError:
        print ("Can't connect to Naoqi at ip \"" + args.ip + "\" on port " + str(args.port) +".\n"
               "Please check your script arguments. Run with -h option for help.")
        sys.exit(1)
    main(session)
```
ALAnimationPlayer allows you to start animations. This module is a wrapper of ALBehaviorManager, its goal is to have an easy way to start animations according to animation tags or the current posture of the robot.

###Animation Paths
To run an application stored in a package, use: package_name/path_of_animation

<aside class = "notice">
run("animations/Stand/Gestures/Hey_1")
run("animations/[posture]/Gestures/Hey_1")
run("my_dances/[robot]/salsa")
</aside>

Start, stop or wait an animation. You can check out more prebuilt animations
<a href='http://doc.aldebaran.com/2-5/naoqi/motion/alanimationplayer-advanced.html#animationplayer-list-behaviors-pepper'> </a> here

Or, You can also use your own behaviors provided that you follow these requirements:

Limit the content of the behavior to an animation (no speech, no complex behaviors), in order to guaranty a good compatibility with ALTextToSpeech.
Lock the resources as follow: Wait 1 second at box startup, and lock resources during box execution - in order to secure a full compatibility with the automatic body language process.

###Tags

You can use any tag supplied by default. For further details, see <a href='http://doc.aldebaran.com/2-5/naoqi/motion/alanimationplayer-advanced.html#animationplayer-tags-pepper'>
</a> here

Additionally, you can declare new tagged animations using: ALAnimationPlayer.declarePathForTags

You can also declare new tags for some specific animations (without modifying the animations) using: ALAnimationPlayer.addTagForAnimations

## Make Pepper Recognize Emotion

```python
import qi
import argparse
import sys
import time


def main(session):
    """
    This example uses the getEmotionalReaction method.
    """
    # Get the services ALMood and ALTextToSpeech.
    moodService = session.service("ALMood")
    tts = session.service("ALTextToSpeech")

    # The robot tries to provocate an emotion by greeting you
    tts.say("Hey, you! Look at me. I'll tell you what your emotion is!")
    # The robot will try to analysis your reaction during the next 3 seconds
    emotion = moodService.getEmotionalReaction()
    tts.say("You look" + emotion)
    print emotion

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--ip", type=str, default="10.80.129.90",
                        help="Robot IP address. On robot or Local Naoqi: use '127.0.0.1'.")
    parser.add_argument("--port", type=int, default=9559,
                        help="Naoqi port number")

    args = parser.parse_args()
    session = qi.Session()
    try:
        session.connect("tcp://" + args.ip + ":" + str(args.port))
    except RuntimeError:
        print ("Can't connect to Naoqi at ip \"" + args.ip + "\" on port " + str(args.port) +".\n"
               "Please check your script arguments. Run with -h option for help.")
        sys.exit(1)
    main(session)
```

ALMood works by estimating the mood of the focused user, provided by ALUserSession.

ALMood’s estimation of valence (i.e. positivity or negativity) builds upon various extractors and memory keys; in particular, it currently uses head angles from ALGazeAnalysis, expressions properties and smile information from ALFaceCharacteristics, acoustic voice emotion analysis from ALVoiceEmotionAnalysis and semantic information from the words spoken by the user.

ALMood’s estimation of attention (i.e. unengaged, semiEngaged or fullyEngaged) takes into account both the head orientation and eye gaze direction to calculate where the user is looking. For example, attention will be high if the user’s head is tilted upwards but their eyes are looking downwards at the robot.

ALMood’s estimation of ambiance (i.e. calm or excited) takes into account the general sound level from ALAudioDevice and the amount of movement in front of the robot.

ALMood retrieves information from the above extractors to combine them into high and low level extractors. Users can access the underlying representation of the emotional perception through a two-level key space: a consolidated information key (e.g. valence, attention) and more intermediate information key (e.g. smile, laugh).

The calculation of high-level keys takes into account the observation at a given moment, the previous emotional state and the ambient and social context (e.g. noisy environment, smile too long, user profile).

All emotional key values are associated with a confidence score between 0 and 1 to indicate how likely an estimation is.

###Basic Emotions
You can query for a basic emotional reaction over 3 seconds.

The analysis starts when the method ALMood::getEmotionalReaction is called.
<aside class = "notice">
An emotional reaction value can be:
“positive”
“neutral”
“negative”
“unknown”
</aside>

###Emotional descriptors
Person emotion:
The following descriptors provide mood data about the focused user.
<aside class = "notice">
Valence: whether the person’s mood is positive or negative.
Attention: the amount of attention the focused person gives to the robot.
Ambiance:
</aside>

Excitement/Agitation: indicates the activity level of the environment, whether it’s calm or excited.

###Perceived stimuli

In its present state, the module reacts to the following stimuli:

Person emotion:
<aside class = "notice">
Smile degree
Facial expressions (neutral, happy, angry, sad)
Head attitude (angles), relative to a robot
Gaze patterns (evasion, attention, diversion)
Utterance accoustic tone
Linguistic semantics of speech
Sensor touch
</aside>

Ambiance:
<aside class = "notice">
Energy level of noise
Movement detection
</aside>

## Advanced Topics

# Running a Service
```python
python service_sample.py --qi-url 10.80.127.123
```
> To access the robot with SSH, use this code. You'll need to authenticate with your password.
> Then you can use a nice tool called qicli to access the existing API modules available on the robot

```shell
ssh nao@10.80.127.123
qicli info
qicli info ALSystem
qicli call ALTextToSpeech.say Hello
```

You can either run a service remotely from your computer, or you can run it on the robot locally.

> To launch the service remotely, use this code.
> Make sure to replace the IP address with your robot IP.
