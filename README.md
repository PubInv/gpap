# GPAP: General Purpose Alarm Protocol

Public Invention is attempting to create an open-source alarm system ecosystem that includes the [Krake
General Purpose Alarm Device](https://github.com/PubInv/krake), an anunciator that lets an operator interact with an alarm,
and the [ADaM Alarm Dialog Manage system](https://github.com/PubInv/ADaM).
This repo defines an important part of that ecosystem, the protocol by which alarm messages and
responses to them are sent between devices and servers.

We intentionaly are not using JSON for this format so that it will be easy to parse for microcontrollers.
This allows sensor devices to generate alarms as (relatively) simple alarm formats,
and allows anunciator devices to announce alarms as simply as possible.
Although the Krake currently uses an ESP32 which is powerful enough to parse JSON,
we prefer to allow simple and cheap devices in the future.

# The Alarm Protocol

The basic alarm message looks like "a4My hair is on fire."
This means that it is an alarm of severity 4, with a message "My haris is on fire."
More precisely, this matches a regular expresson like:

^([i|u|s|a])([012345]?)(\{(.*)\})?(.{0,80})$

The first group defines the "Message Type". Is is either:
1. "i"nformation
2. "u"nmute
3. "s"ilence (mute) yourself
4. "a"alarm.

When the message type is an alarm, the message itself then has more parts.
The *severity* is a single digit (0-5) immediately follows the "a" and 
is required. All other parts are optional.  

In an alarm, if a curly-braced string occurs, it is taken to be a
*message id*. It must be a hexadecimal number (in either lower case or upper case.)
The message id may be a serial number of a hash code, but signifies the 
specific instance of this alarm occuring. A message with a message id 
looks like:

> "a4{37F4A}My hair is on fire."

If a message id occurs, it must occur immediately after the severity digit.

Similarly, a message may contain an *alarm type designator*. Best practice
is to specify all alarm types in an alarm database (see Hollifield and Habibi).
The alarm type designator is exactly 3 digits enclosed in square brackets.

For example, "a4[313]" to mean "annuciate alarm # 313 from the alarm database".)

In additions to this, we define a simple request for status as simply the character "s".

Finally, we definite a "heartbeat" message designed to allow an annunciator to
observe that the source of alarms is active and sending heartbeats.


# The Response Protocol

We have been strongly influenced by the book Alarm Managment: Hollifield, Bill R., and Eddie Habibi. Alarm management: A comprehensive guide: Practical and proven methods to optimize the performance of alarm management systems. Isa, 2011.

An anunciator device alerts human beings (called "operators") to an impportant event of situation
they need to know about. The operator may choose to interact with the alarm, creating a "dialog"
with the alarm or the server presenter. The responses begin with the letter "o" to represent
that they are initiated by the operator (as opposed to automatically).

In general, an operator may take four actions in response to an alarm:
1. Acknowledge the alarm [a] to mark the fact that they are aware of it.
2. Shelve the alarm [s] (the operator asserts that they do not intend to address it at present.)
3. Dismiss the alarm [d] (the operator asserts that the alarm condition does not accidentally exist and is likely a false alarm.)
4. Complete the alarm [c] (the operator asserts that an action has been taken to complete alarm)

These 4 actions are specified in the next character following th "o". An example is:

> "oc"

...which means that the operator has completed the current alarm.

> "od{37F4A}"

...means the the alarm identified by 37F4A is dismissed.
