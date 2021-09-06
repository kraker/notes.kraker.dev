---
title: Managing the Exim Queue
date: 2021-04-15 10:30
---

# Managing the Exim Queue
[Source](https://wiki.inmotionhosting.com/index.php?title=Managing_the_Exim_Queue)

## Looking at the Queue

Count how many messages are in queue:
```
exim -bp | exiqsumm | grep {domain.tld}
```

Get message ids of mail in queue.
Outgoing:
```
exiqgrep -if {domain.tld} | head -5
```

Incoming:
```
exiqgrep -ir domain.com | head -5
```

## Running the Queue

Start a queue run:
```
exim -q -v
```

Start a queue run for just local deliveries:
```
exim -ql -v
```

Start queue run for _non-frozen_ messages:
```
exim -qf -v
```

_Unthaw_ frozen messages and start queue run:
```
exim -qff -v
```

## Handling Individual Messages

### Table of Commands

| Flag    | Description                           | Usage                                                |
|---------|---------------------------------------|------------------------------------------------------|
| `-M`    | Force delivery                        | `exim -M <message-id> [ <message-id> ... ]`          |
| `-Mc`   | Deliver message if retry time reached | `exim -Mc <message-id> [ <message-id> ... ]`         |
| `-Mar`  | Add recipient                         | `exim -Mar <message-id> <address> [ <address> ... ]` |
| `-Mes`  | Edit sender                           | `exim -Mes <message-id> <address>`                   |
| `-Mf`   | Freeze message                        | `exim -Mf <message-id> [ <message-id> ... ]`         |
| `-Mg`   | Give up (bounce message)              | `exim -Mg <message-id> [ <message-id> ... ]`         |
| `-Mmad` | Mark all recipients as delivered      | `exim -Mmad <message-id> [ <message-id> ... ]`       |
| `-Mmd`  | Mark recipient as delivered           | `exim -Mmd <message-id> <address> [ <address> ... ]` |
| `-Mrm`  | Remove message (no bounce)            | `exim -Mrm <message-id> [ <message-id> ... ]`        |
| `-Mt`   | Thaw message                          | `exim -Mt <message-id> [ <message-id> ... ]`         |
| `-Mvb`  | View message body                     | `exim -Mvb <message-id>`                             |
| `-Mvh`  | View message header                   | `exim -Mvh <message-id>`                             |
| `-Mvl`  | View message log                      | `exim -Mvl <message-id>`                             |

### Examples

_STUB_
