# SteelSeries Game Event Support #

Most modern SteelSeries gaming devices have RGB illumination. All of our new and recent mice have two illumination zones, and many of our headsets have one. We've had backlit keyboards at times in the past, but the APEX was the first to offer multi-zone (5) RGB. The APEX M800 goes further, offering individual control of the color of each of over 100 keys.

To better make use of these capabilities we've built a framework for receiving events from games and using our devices' illumination (for now, with other forms of output possible in future products) capabilities as indicators of game state.

## Server discovery ##

Before you can use the GameSense™ SDK, your game will need to find out where to send handlers and events. To do this it will need to read the `coreProps.json` file that Engine creates when it starts. This file contains a json object. You are interested in the `address` top level key. The corresponding value is the host and port that Engine is listening on. This is a string in `"host:port"` format. E.g.

    {
      "address": "127.0.0.1:51248"
    }

This address value can then be used to create the URL used to post game handler specifications and events, by appending `"/game_event"`.

This file can be found in one of these locations, depending on OS:

**OSX**     | `/Library/Application Support/SteelSeries Engine 3/coreProps.json`

**Windows** | `%PROGRAMDATA%/SteelSeries/SteelSeries Engine 3/coreProps.json`

If this file is not present, then SteelSeries Engine 3 is not running and you should not attempt to send events.

## Game Events ##

Games communicate with SteelSeries Engine 3 by posting a specifically formatted JSON object to Engine's endpoint.  The properties of this object specify the game it is coming from, the event it corresponds to, and a data payload including a `"value"` property with an arbitrary value that is used by the handler. For example:

    {
      game: "MY_GAME",
      event: "HEALTH",
      data: {
          "value": 75
      }
    }

Notes about the data:
* The values for `game` and `event` are limited to uppercase A-Z, 0-9, hyphen, and underscore characters.
* All three of the keys are mandatory for the event to be processed.
* The value for `data` can be either a JSON object or a string containing the stringified form of a JSON object.  Any events that will be handled by the JSON API *must* contain the key `value`.  For simplicity and greatest compatibility with user configurability of these events in SteelSeries Engine, it is recommended that the event data be a single `value` key with numeric values.

The events must be sent as a POST request to the address `<SSE3 url>/game_event`, with a content type of `application/json`.

It is generally recommended to encapsulate the creation and POST of the JSON within a function that takes the event name and value to send, so that it can be re-used with each event type sent.

The event framework will work with games written in any language. All you need is to be able to create a JSON formatted string and POST it to a local URL. We've used it so far with languages as varied as C++, Java, Swift, Go, and Javascript.

## Heartbeat/Keepalive Events ##

GameSense™ is initialized on devices when the first event for a game is recieved.  It is deactivated when no events have been received within its timeout period of 15 seconds.  This means that your game should send at least one event every 15 seconds if you want the game state to continue to be fully represented on the user's devices.

An additional endpoint, `game_heartbeat`, is available to simplify this process.  The data payload sent to this endpoint only needs to include the name of the game:

    {
      game: "MY_GAME"
    }

This endpoint does not affect any state on the user devices, but resets the GameSense™ deactivation timer.  Use of this endpoint is completely optional, as you can also send real event data to keep GameSense™ alive.
