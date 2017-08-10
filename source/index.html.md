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
> If running on robot (Packaged and Installed) use the local Naoqi IP address indicated below.

Welcome to NAOqi API Tutorial. We are excited to start building awesome applications on our robots. The pre-requisite to use the APIs can be found <a href='http://doc.aldebaran.com/2-5/dev/python/install_guide.html'> here </a> and we assume you have installed the Python SDKs with paths added. If you choose a different programming language to Python, please also add the SDK for that specific language of your choice.

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

Before starting, ALSpeechRecognition needs to be fed by the list of phrases that should be recognized. Once started, ALSpeechRecognition places in the key SpeechDetected, a boolean that specifies if a speaker is currently heard or not. If a speaker is heard, the element of the list that best matches what is heard by the robot is placed in the key WordRecognized. If a speaker is heard, the element of the list that best matches what is heard by the robot is placed in the key WordRecognizedAndGrammar.

TTS module allows the robot to speak. It sends commands to a text-to-speech engine, and authorizes also voice customization. The result of the synthesis is sent to the robot’s loudspeakers.

In this example, we have added "yes", "no" and "please" to the vocabulary so you can test the reaction by saying "yes", "no" or "please". Pepper should respond by saying "Hello, speech recognition engine started" if the speech recognition works.

You can find more about the ALTextToSpeech API Module  <a href='http://doc.aldebaran.com/2-5/naoqi/audio/altexttospeech-api.html#altexttospeech-api'>here</a> and ALSpeechRecognition API Module <a href='http://doc.aldebaran.com/2-5/naoqi/audio/alspeechrecognition-api.html?highlight=alspeechre#ALSpeechRecognitionProxy'>here</a>.


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

This example sets up a topic following the QiChat syntax, you could also load the topic in a .top file on the robot.

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

###Collaborative dialog vs applications
We can differentiate three steps in robot life: autonomous life, collaborative dialog (interactive state) and your application. Collaborative dialog allows running your application by voice. You can simply add trigger sentences in the manifest of your application or add more advanced dialog.

<aside class="notice">
topic: ~startmyweather
language: enu

include: lexicon_enu.top

u: (start weather) are you sure?
u1: (~yes) ok ^switchFocus(myweather/.)
</aside>

Find more about QiChat rules <a href='http://doc.aldebaran.com/2-5/naoqi/interaction/dialog/aldialog_syntax_toc.html'>here</a>


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

Start, stop or wait an animation. You can check out more prebuilt animations
<a href='http://doc.aldebaran.com/2-5/naoqi/motion/alanimationplayer-advanced.html#animationplayer-list-behaviors-pepper'>here </a>

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
    1st example uses the currentPersonState method.
    """
    # Get the service ALMood.
    moodService = session.service("ALMood")
    tts = session.service("ALTextToSpeech")
    valence = moodService.currentPersonState()["valence"]["value"]
    tts.say("your current state is " + valence)

    """
    2nd example uses the getEmotionalReaction method.
    """
    # Get the services ALMood and ALTextToSpeech.

    # The robot tries to provocate an emotion by greeting you
    tts.say("Hey, you! Look at me. I'll tell you what your emotion is!")
    # The robot will try to analysis your reaction during the next 3 seconds
    emotion = moodService.getEmotionalReaction()
    tts.say("You reaction is" + emotion)

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

ALMood works by estimating the mood of the focused user, provided by ALUserSession. The first example is how to get current valence and the second example is to get emotional reaction after a provocation.

ALMood’s estimation of valence (i.e. positivity or negativity) builds upon various extractors and memory keys; in particular, it currently uses head angles from ALGazeAnalysis, expressions properties and smile information from ALFaceCharacteristics, acoustic voice emotion analysis from ALVoiceEmotionAnalysis and semantic information from the words spoken by the user.

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

# Advanced Topics

## Create a Service

A service is a background process. It is always running and advertises : <br>
- Methods <br>
- Signals <br>
- Properties <br>


## qi Framework

```python
#service example
import qi
import os
import sys


class ServiceSample:
    services_connected = None
    connected_signals = []
    number_of_activities_launched = 0

    def __init__(self, application):
        # Getting a session that will be reused everywhere
        self.application = application
        self.session = application.session
        self.service_name = self.__class__.__name__

        self.activities_launched = qi.Signal()

        # Getting a logger. Logs will be in /var/log/naoqi/servicemanager/{application id}.{service name}
        self.logger = qi.Logger(self.service_name)

        # Do some initializations before the service is registered to NAOqi
        self.logger.info("Initializing...")
        self.connect_services()
        self.logger.info("Initialized!")

    @qi.nobind
    def start_service(self):
        self.logger.info("Starting...")
        self.subscribe_to_autonomous_life()
        self.logger.info("Started!")

    @qi.nobind
    def stop_service(self):
        # probably useless, unless one method needs to stop the service from inside.
        # external naoqi scripts should use ALServiceManager.stopService if they need to stop it.
        self.logger.info("Stopping service...")
        self.application.stop()
        self.logger.info("Stopped!")

    @qi.bind(methodName="getNumberActivitiesLaunched", paramsType=(), returnType=qi.Int32)
    def get_number_activities_launched(self):
        return self.number_of_activities_launched

    @qi.nobind
    def connect_services(self):
        # connect all services required by your module
        # done in async way over 30s,
        # so it works even if other services are not yet ready when you start your module
        # this is required when the service is autorun as it may start before other modules...
        self.logger.info('Connecting services...')
        self.services_connected = qi.Promise()
        services_connected_fut = self.services_connected.future()

        def get_services():
            try:
                self.memory = self.session.service('ALMemory')
                self.life = self.session.service('ALAutonomousLife')
                # connect other services if needed...
                self.logger.info('All services are now connected')
                self.services_connected.setValue(True)
            except RuntimeError as e:
                self.logger.warning('Still missing some service:\n {}'.format(e))

        get_services_task = qi.PeriodicTask()
        get_services_task.setCallback(get_services)
        get_services_task.setUsPeriod(int(2 * 1000000))  # check every 2s
        get_services_task.start(True)
        try:
            services_connected_fut.value(30 * 1000)  # timeout = 30s
            get_services_task.stop()
        except RuntimeError:
            get_services_task.stop()
            self.logger.error('Failed to reach all services after 30 seconds')
            raise RuntimeError

    @qi.nobind
    def subscribe_to_autonomous_life(self):
        self.add_memory_subscriber("AutonomousLife/FocusedActivity", self.on_focused_activity)
        self.add_memory_subscriber("AutonomousLife/State", self.on_state_changed)
        pass

    @qi.nobind
    def on_focused_activity(self, activity):
        self.logger.info("FocusedActivity signal raised: {}".format(activity))
        if activity:
            self.logger.info("New focused activity: {}".format(activity))
            self.number_of_activities_launched += 1
            self.activities_launched(self.number_of_activities_launched)
            self.logger.info("Number of activities focused since last activation: {}".format(self.number_of_activities_launched))
        else:
            self.logger.info("Just going back to IDLE, discard")
        pass

    @qi.nobind
    def on_state_changed(self, state):
        self.logger.info("State signal raised: {}".format(state))
        if state == "disabled":
            self.logger.info("AutonomousLife disabled, resetting count")
            self.number_of_activities_launched = 0
            self.activities_launched(0)
        else:
            self.logger.info("AutonomousLife is changing state but remains activated")
        pass

    @qi.nobind
    def add_memory_subscriber(self, event, callback):
        # add memory subscriber utility function
        self.logger.info("Subscribing to {}".format(event))
        try:
            sub = self.memory.subscriber(event)
            con = sub.signal.connect(callback)
            self.connected_signals.append([sub, con])
        except Exception, e:
            self.logger.info("Error while subscribing: {}".format(e))

    @qi.nobind
    def remove_memory_subscribers(self):
        # remove memory subscribers utility function
        self.logger.info("unsubscribing to all signals...")
        for sub, con in self.connected_signals:
            try:
                sub.signal.disconnect(con)
            except Exception, e:
                self.logger.info("Error while unsubscribing: {}".format(e))

    @qi.nobind
    def cleanup(self):
        # called when your module is stopped
        self.logger.info("Cleaning...")
        self.remove_memory_subscribers()
        self.logger.info("End!")

    # ################# #

if __name__ == "__main__":
    # with this you can run the script for tests on remote robots
    # run : python my_super_service.py --qi-url 123.123.123.123
    app = qi.Application(sys.argv)
    app.start()
    service_instance = ServiceSample(app)
    service_id = app.session.registerService(service_instance.service_name, service_instance)
    service_instance.start_service()
    app.run()
    service_instance.cleanup()
    app.session.unregisterService(service_id)

```



### qi.Application API

Application initializes the qi framework, it extract –qi-* commandline arguments and initialise various options accordingly. It also provides facilities to connect the main session. This ease the creation of console applications that connects to a session or want to be standalone.

You can pass the following arguments to your application to control the session:

–qi-url : address of the session to connect to
–qi-listen-url : address on which the session will listen
–qi-standalone : make a standalone session (use –qi-listen-url to set the listen url)
By default the session url is set to “tcp://127.0.0.1:9559” and the listen url to “tcp://0.0.0.0:0”.

If raw is specified there wont be a session embedded into the application and you are free to create and connect a session yourself if needed.

if autoExit is set to True, a session disconnection will quit the program. Set autoExit to False to inhibit this behavior. By default autoExit is True.

Application will parse your arguments and catch –qi-* arguments.

For a normal application you have to call start to let the Session embedded into the application connect. If you want to handle –help yourself you can do that before the application starts avoiding a useless connection to the session.

### qi.Session API

A session connect to a standalone qi.Session. Once connected the session can:

advertise new services using qi.Session.registerService()
get services using qi.Session.service()

### qi.Future API

Promise and Future are a way to synchronise data between multiples threads. The number of future associated to a promise is not limited. Promise is the setter, future is the getter. Basically, you have a task to do, that will return a value, you give it a Promise. Then you have others thread that depend on the result of that task, you give them future associated to that promise, the future will block until the value or an error is set by the promise.

### qi.Signal API
Signal allows communication between threads. One thread emits events, other threads register callback to the signal, and process the events appropriately.


### Bind API
qi.bind allow specifying types for bound methods. By default methods parameters are of type AnyValue and always return an AnyValue.

With qi.bind you can specify which types the function really accept and return. With qi.nobind you can hide methods.

### Using qicli commands

qicli commands allows you to:

1. See information
2. Use methods
3. Watch signals

You can either run a service remotely from your computer, or you can run it on the robot locally. Change the IP to your robot IP address.
<aside class = "notice">
python service_sample.py --qi-url 10.80.127.123
</aside>

To access the robot with SSH, use this code. You'll need to authenticate with your password.
Then you can use qicli commands.

<aside class = "notice"> <b>Use secure shell to access the robot </b> <br>
ssh nao@10.80.127.123 <br>
<b>Then you can access the qicli commands</b> <br>
qicli info <br>
qicli info ALSystem <br>
qicli call ALTextToSpeech.say Hello <br>
</aside>

## Find the Logs

System:
tail -f /var/log/naoqi/servicemanager/system.Naoqi.log
tail -f /var/log/naoqi/servicemanager/system.Naoqi_error.log

Services:
tail -f /var/log/naoqi/servicemanager/<app-uuid>.<service-name>.log
tail -f /var/log/naoqi/servicemanager/<app-uuid>.<service-name>_error.log

## Sample Robot Application

Controller : 	Service written in python <br>
Tablet: 		Webpage written in HTML + Javascript <br>
Dialog: 		Qichat <br>

Here is a color app which demonstrates the MVC (Model View Controller) architecture of our robot application. <a href="file:///Users/mo.zhou/Training/python-app-colors-example/working-Color-App.zip" download>Download link</a>

→ Expose a method to be called by Javascript

→ Subscribe to the events raised by the QiChat

→ Communicate back to the tablet to change the background’s color
