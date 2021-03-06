[![Build Status](https://travis-ci.com/AccelByte/eventstream-go-sdk.svg?branch=master)](https://travis-ci.com/AccelByte/eventstream-go-sdk)

# eventstream-go-sdk
Go SDK for integrating with AccelByte's event stream

## Usage

### Install

```
go get -u github.com/AccelByte/eventstream-go-sdk
```

### Importing

```go
eventstream "github.com/AccelByte/eventstream-go-sdk"
```

To create a new event stream client, use this function:

```go
client, err := eventstream.NewClient(prefix, stream, brokers, config)
``` 
``NewClient`` requires 4 parameters :
 * prefix : Topic prefix from client (string)
 * stream : Stream name. e.g. kafka, stdout, none (string)
 * brokers : List of kafka broker (array of string)
 * config : Custom broker configuration from client. 
 This is optional and only uses the first arguments. (variadic *BrokerConfig)   


## Supported Stream
Currently event stream are supported by these stream:

### Kafka Stream
Publish and subscribe an event to / from Kafka stream. 

currently compatible with golang version from 1.12+ and Kafka versions from 0.10.1.0 to 2.1.0.

To create a kafka stream client, just pass the stream parameter with `kafka`.

#### Custom Configuration
SDK support with custom configuration for kafka stream, that is :

* DialTimeout : Timeout duration during connecting to kafka broker. Default: 10 Seconds (time.Duration)
* ReadTimeout : Timeout duration during consume topic from kafka broker. Default: 10 Seconds (time.Duration)
* WriteTimeout : Timeout duration during publish event to kafka broker. Default: 10 Seconds (time.Duration) 
* LogMode : eventstream will print log based on following levels: info, warn, debug, error and off. Default: off (string) 
* StrictValidation : If it set true, eventstream will enable strict validation for event fields, Default: False (boolean) 

```go
    config := &eventstream.BrokerConfig{
		LogMode:          eventstream.InfoLevel,
		StrictValidation: true,
		DialTimeout:      1 * time.Second,
		ReadTimeout:      1 * time.Second,
		WriteTimeout:     1 * time.Second,
	}
```

### Stdout Stream
This stream is for testing purpose. This will print the event in stdout. It should not be used in production since this 
will print unnecessary log.

To create a stdout stream client, just pass the stream parameter with `stdout`.

### Blackhole
This is used when client don't want the service to send event data to anywhere.

To create a blackhole client, just pass the stream parameter with `none`.

## Publish Subscribe Event

### Publish
Publish or sent an event into stream. Client able to publish an event into single or multiple topic.
Publish to `kafka` stream support with exponential backoff retry. (max 3x)

To publish an event, use this function:
```go
err := client.Publish(
		NewPublish().
			Topic(TopicName).
			EventName(EventName).
			Namespace(Namespace).
			ClientID(ClientID).
			UserID(UserID).
			SessionID(SessionID).
			TraceID(TraceID).
			Context(Context).
			Version(Version).
			Payload(Payload).
			ErrorCallback(func(event *Event, err error) {}))
```

#### Parameter 
* Topic : List of topic / channel. (variadic string - alphaNumeric(256) - Required)
* EventName : Event name. (string - alphaNumeric(256) - Required) * Namespace : Event namespace. (string - alphaNumeric(256) - Required)
* ClientID : Publisher client ID. (string - UUID v4 without Hyphens)
* UserID : Publisher user ID. (string - UUID v4 without Hyphens)
* SessionID : Publisher session ID. (string - UUID v4 without Hyphens)
* TraceID : Trace ID. (string - UUID v4 without Hyphens)
* Context : Golang context. (context - default: context.background)
* Version : Version of schema. (integer - default: `1`)
* Payload : Additional attribute. (map[string]interface{})
* ErrorCallback : Callback function when event failed to publish. (func(event *Event, err error){})

### Subscribe
To subscribe an event from specific topic in stream, client should be register a callback function that executed once event received.
A callback aimed towards specific topic and event name.

To subscribe an event, use this function:
```go
err := client.Register(
		NewSubscribe().
			Topic(topicName).
			EventName(mockEvent.EventName).
			GroupID(groupID).
			Context(Context).
			Callback(func(event *Event, err error) {}))
```

#### Parameter 
* Topic : Subscribed topic. (string - alphaNumeric(256) - Required)
* EventName : Event name. (string - alphaNumeric(256) - Required)
* Namespace : Event namespace. (string - alphaNumeric(256) - Required)
* GroupID : Message broker group / queue ID. (string - alphaNumeric(256) - default: `*`)
* Context : Golang context. (context - default: context.background)
* Callback : Callback function when receive event. (func(event *Event, err error){} - required)

Callback function passing 2 parameters:
* ``event`` is object that store event message. 
* ``err`` is an error that happen when consume the message.

## Event Message
Event message is a set of event information that would be publish or consume by client.

Event message format :
* id : Event ID (string - UUID v4 without Hyphens)
* name : Event name (string)
* namespace : Event namespace (string)
* traceId : Trace ID (string - UUID v4 without Hyphens)
* clientId : Publisher client ID (string - UUID v4 without Hyphens)
* userId : Publisher user ID (string - UUID v4 without Hyphens)
* sessionId : Publisher session ID (string - UUID v4 without Hyphens)
* timestamp : Event time (time.Time)
* version : Event schema version (integer)
* payload : Set of data / object that given by producer. Each data have own key for specific purpose. (map[string]interface{})
