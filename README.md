# GPAP: General Purpose Alarm Protocol

**Current Version:** 0.1.1

Public Invention is attempting to create an open source alarm system ecosystem that includes the Krake General Purpose Alarm Device, an annunciator that lets an operator interact with an alarm, and the ADaM Alarm Dialog Manager system. This repo defines the protocol by which alarm messages and responses to them are sent between devices and servers.

[Krake General Purpose Alarm Device](https://github.com/PubInv/krake), an anunciator that lets an operator interact with an alarm,
and the [ADaM Alarm Dialog Manage system](https://github.com/PubInv/ADaM).
This repo defines an important part of that ecosystem, the protocol by which alarm messages and
responses to them are sent between devices and servers.

We intentionally are not using JSON for this format so that it will be easy to parse on microcontrollers. This allows sensor devices to generate alarms in a simple format, and allows annunciator devices to announce alarms as simply as possible. Although the Krake currently uses an ESP32 which is powerful enough to parse JSON, we prefer to allow simple and cheap devices in the future.

---

## Versioning

GPAP follows Semantic Versioning.

- **MAJOR** for incompatible protocol changes  
- **MINOR** for backward compatible additions  
- **PATCH** for backward compatible fixes and documentation corrections  

GPAP is currently in 0.y.z development, which means the protocol is still evolving and may change before 1.0.0.

---

# The Alarm Protocol

The basic alarm message looks like:

> `a4My hair is on fire.`

This means it is an alarm of severity 4 with the content `My hair is on fire.`

---

## Message Types

The first character of every message is the message type.

- `i` information  
- `u` unmute  
- `s` silence or mute yourself  
- `a` alarm  
- `b` heartbeat  

---

## Alarm Message Structure

When the message type is `a` (alarm), the message may have more parts. The severity digit is required. All other parts are optional.

Order matters:

1. Message type `a`  
2. Severity digit `0-5` (required for alarms)  
3. Optional message id `{...}` (hex)  
4. Optional alarm type designator `[...]` (3 digits)  
5. Optional content up to 80 characters  

---

### Severity

Severity is a single digit from `0` to `5` immediately after `a`.

---

### Message id

If a curly braced string occurs, it is taken to be a message id. It must be a hexadecimal number in lower case or upper case. The message id may be a serial number or a hash code, but it signifies the specific instance of this alarm occurring.

Example:

> `a4{37F4A}My hair is on fire.`

If a message id occurs, it must occur immediately after the severity digit.

The message id is meant to allow acknowledgment of a specific alarm instance.

---

### Alarm type designator

A message may contain an alarm type designator. Best practice is to specify all alarm types in an alarm database. The alarm type designator is exactly 3 digits enclosed in square brackets.

Example:

> `a4[313]`

This means "annunciate alarm type 313 from the alarm database."

---

### Content

Any remaining text that is not the message type, severity, message id, or alarm type designator is content. Content is limited to 80 characters.

---

## Grammar and Regular Expressions

This protocol is intended to be easily parsable without a full regex engine.

### Simple parsing rules

- Read the first character as message type  
- If message type is `a`, read the next character as severity digit `0-5`  
- If the next character is `{`, parse a hex message id until `}`  
- If the next character is `[`, parse exactly three digits until `]`  
- The rest is content (0 to 80 characters)  

### Helpful regular expressions (documentation/testing)

General message:

```
^(b|i|u|s|a)([0-5]?)(\{[0-9A-Fa-f]+\})?(\[\d{3}\])?(.{0,80})$
```

Alarm messages specifically:

```
^a([0-5])(\{[0-9A-Fa-f]+\})?(\[\d{3}\])?(.{0,80})$
```

---

## Examples

### Alarm examples

```
a4My hair is on fire.
a4{37F4A}My hair is on fire.
a4[313]
a4{37F4A}[313]My hair is on fire.
```

### Non-alarm examples

```
iSystem running normally
s
u
```

---

# Heartbeat

We define a heartbeat message designed to allow an annunciator to observe that the source of alarms is active and sending heartbeats.

Current placeholder form:

```
b
```

Optional content may be appended, but the exact heartbeat fields are not stable yet.

### Open questions

- Should heartbeat include a timestamp?  
- Should it include signal strength?  
- Should it include a serial number?  

---

# The Response Protocol

We have been strongly influenced by the book *Alarm Management* by Hollifield and Habibi.

An annunciator device alerts human beings, called operators, to an important event or situation they need to know about. The operator may choose to interact with the alarm, creating a dialog with the alarm or the server presenter.

Responses begin with the letter `o` to represent that they are initiated by the operator rather than automatically.

---

## Operator Actions

An operator may take four actions in response to an alarm:

- **a** — Acknowledge the alarm (mark that they are aware of it)  
- **s** — Shelve the alarm (do not intend to address it at present)  
- **d** — Dismiss the alarm (likely a false alarm)  
- **c** — Complete the alarm (action has been taken to resolve it)  

These four actions are specified by the next character following the `o`.

---

### Examples

```
oc
```

This means the operator has completed the current alarm.

```
od{37F4A}
```

This means the alarm instance identified by message id `37F4A` is dismissed.

---

## Notes

- If a message id is present in a response, it must be enclosed in curly braces and be hex.  
- If no message id is present, the response applies to the currently presented alarm as defined by the server or annunciator session.  

---

# Compatibility Guidance

Because GPAP, GPAD_API, and ADaM may be slightly out of sync while the ecosystem evolves, implementations should be liberal in what they accept.

### Recommended behavior

- Accept alarm messages with or without message id  
- Accept alarm messages with or without alarm type designator  
- If alarm type designator is missing, fall back to default handling and log that the alarm type is unknown  
- Ignore unsupported message types while logging the raw message  

---

# TODO

- Decide stable heartbeat fields and format  
- Consider whether control of backlight should be added to the protocol  
- Add releases and tags for protocol versions  