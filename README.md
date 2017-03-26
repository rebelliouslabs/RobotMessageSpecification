# Robot Message Protocol

The Robot Message Protocol takes a lot from Meteor's DDP. DDP has a robust spec and is powerful.

## General Message Structure

RMP currently only uses WebSockets. Clients that do not support WebSockets will not be able to connect to a RMP server.

Messages sent to the server are JSON objects. Each message has a 'msg' field that specifies the message type. An example message is below.

```
{
  msg: 'connect',
  version: 1
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    robot: {
      id: 'ROBOT-CONNECT-ID'
    }
  }
}
```

## Establishing a RMP Connection

Connecting to a RMP service requires the ability to accept a connection from two different client types.

The first is an authenticated client that allows direct messaging between the robot and the service. You can think of this as true two way communication.

The other option is an unauthenticated known client. These clients can receive messages from the service but are limited in the messages the service can receive from these clients. An example of this client would be a retail kit where the user pairs the robot based on a key. We will not detail this part of the spec here b/c it is not relievant to most connecting clients.

### Connection Procedure

Todo

### Message Examples

**Authenticated Client**

```
{
  msg: "connect",
  version: 1,
  api: {
    token: 'API-TOKEN'
  },
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    robot: {
      id: 'ROBOT-CONNECT-ID'
    }
  }
}
```

**Server Accepts Connection**

```
{
  msg: "connected",
  version: 1,
  clientId: 'CLIENT-ID'
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    session: {
      renewal: 'RENEWAL-TOKEN'
    }
  }
}
```

## Heartbeats

Heartbeats are intermittent messages between the RMP service and client. At any time the client or service can send a message 'ping' and the receiver must respond with ping. If an `id` is included as an `option` the receiver must also include the `id` in its response.

### Message Example

**Ping**

```
{
  msg: "ping",
  version: 1,
  clientId: 'CLIENT-ID'
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    id: 'OPTIONAL-ID'
  }
}
```

**Pong**

```
{
  msg: "pong",
  version: 1,
  clientId: 'CLIENT-ID'
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    id: 'OPTIONAL-ID'
  }
}
```


## Robot Control Message

A Robot Control Message is a message sent from a client or a service to a robot client. This message follows a convention so if you implement a message handler on a robot please follow the convention. It will make life easier.

We will start with a short example to give context to its implementation.

```
{
  msg: "control",
  version: 1,
  api: {
    token: 'API-TOKEN'
  },
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    robot: {
      id: 'ROBOT-CONNECT-ID'
    },
    do: 'turn-right',
    speed: 10
  }
}
```

You will notice in the `options` property a `do` option. This option is mapped to a control command instructing the robot client to do something. This is a list of standard commands: `forward`, `backward`, `stop`, `turn-right`, `turn-left`, `die`. When possible implement these on a self constructed bot.

You can also pass an optional speed with a command changing the speed the robot operates at. This value is normal normalized to a value between 0 and 100 (0<= speed <= 100).

## Agent Control Message

Agents are programs that run on a robot. They run independently of control actions allowing the robot to run autonomously. Agents that utilize the RMP protocol must implement message handlers for the following types: `start`, `stop`, `pause`, `die`.

An example start message to an agent is below.

```
{
  msg: "agent",
  version: 1,
  api: {
    token: 'API-TOKEN'
  },
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    robot: {
      id: 'ROBOT-CONNECT-ID'
    },
    do: 'start',
    agent: 'AGENT-NAME'
    params: {
      // Optional
    }
  }
}
```

The main fields of interest in this example are the `do`, `agent`, and `params` fields. These act as commands for a stated agent.

**Option Descriptions**

* `do` - The action the agent should take.
* `agent` - The name of the agent to target.
* `params` - Optional data to pass to the agent.

## Health/Status Check Message

Health or Status messages are sent from the client or RMP service requesting information from the robot. This includes what it is currently doing, if an agent is running, or anything else registered with the bot runtime object.

### Example Message

```
{
  msg: "status",
  version: 1,
  api: {
    token: 'API-TOKEN'
  },
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {

  }

}
```

## Received Messages

These messages are responses to messages received. When a client ask a robot to run a command, agent, or call a procedure a received message sent back to the client or service. This is a common example of the use case for this type of message.

### Message Example



## Errors

Error Message can be sent from any client, service, or robot. Below is an example error message.

```
{
  msg: "error",
  version: 1,
  api: {
    token: 'API-TOKEN'
  },
  session: {
    token: 'SESSION-TOKEN'
  },
  options: {
    error: 'string or number',
    reason: 'The reason we see the error',
    message: 'A longer description of what happened.',
    errorType: 'A pre-defined string from the error api.'
  }
}
```
