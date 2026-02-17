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

^a[0-5](\{[0-9a-fA-F]{1,16}\})?.{0,80}$

The hexadecimal part of that is a hexadecimal code that should be interpreted as a
"message id", which may be a serial number of a hash code. This is optional.
A message like that looks like:

> "a4{37F4A}My hair is on fire."

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
