---
order: 5
---
# Stream Connect (Beta)

<Aside>

Note: Stream Connect is currently in closed beta and will be available to all users in coming weeks. To request an invite, [submit a request for an invite](https://docs.google.com/forms/d/1rpRFKDTZTnh0LxysM2_Rky5pYt_rs9pUdSoJr_Ix2pk/edit).

</Aside>

Stream Connect allows you to retransmit your RTMP(S) feed to one or more outputs that support RTMP. To learn more about the vision and benefits, checkout the [Stream Connect blog post](https://blog.cloudflare.com/restream-with-stream-connect/). 

Stream Connect does not output HLS/DASH at the moment. This will be supported at a later date.

## Quick Start

There are four steps to start using Stream Connect

1. Create a live input that you will transmit to. Upon creating an input, you will receive an RTMP endpoint and stream key to use in step 3.
2. Add an output to the live input
3. Configure your streaming software with the RTMP endpoint and stream key from Step 1
4. Start streaming! Stream Connect will automatically ingest the video and push it to the configured outputs

You can do steps 1 and 2 using the API or the [Stream Connect UI](https://dash.cloudflare.com/?to=/:account/stream/inputs)

### Create a live input

This call allocates configuration for your RTMP(S) feed to be terminated at.

The response from this endpoint contains the uid to identify the source, this is referred to as `$INPUT_UID` later on. It also contains the rtmps information required for broadcasting to this input.

You may include additional metadata, such as the input’s name, they may do so by including a body on this request:
`--data '{"meta": {"name": "test stream 1"}}'`

Note that this metadata has no impact on Stream Connect's behavior, it is entirely for your bookkeeping.


#### cURL example
```bash
curl -X POST \
-H "Authorization: Bearer $TOKEN" \
https://api.cloudflare.com/client/v4/accounts/$ACCOUNT/stream/live_inputs
```

##### Example response

```bash
{
  "success": true,
  "errors": [],
  "messages": [],
  "result": {
    "uid": "66be4bf738797e01e1fca35a7bdecdcd",
    "meta": {
      "name": "test stream 1"
    },
    "created": "2014-01-02T02:20:00Z",
    "modified": "2014-01-02T02:20:00Z",
    "rtmps": {
      "url": "rtmps://live.cloudflare.com:443/live/",
      "streamKey": "<redacted>"
    }
  }
}
```

### Add an output

This call configures a live input to rebroadcast data sent to it to a third party.

As with the input, the response body's result object contains the uid as well as the data you sent. This uid will be referred to as `$OUTPUT_UID` elsewhere.

An output is associated with an input, to use it on another input you must call this endpoint against that input.

#### cURL example
```bash
curl -X POST \
--data '{"url": "rtmp://a.rtmp.youtube.com/live2","streamKey": "redacted"}' \
-H "Authorization: Bearer $TOKEN" \
https://api.cloudflare.com/client/v4/accounts/$ACCOUNT/stream/live_inputs/$INPUT_UID/outputs
```

##### Example response

```bash
{
  "result": {
    "uid": "6f8339ed45fe87daa8e7f0fe4e4ef776",
    "url": "rtmp://a.rtmp.youtube.com/live2",
    "streamKey": "<redacted>"
  },
  "success": true,
  "errors": [],
  "messages": []
}
```

### Start streaming

Use the url and streamKey returned from input creation in your streaming software.

TODO(zaid): obs example screenshots for a step-by-step?

## Limits

Some limits apply to the Stream Connect Beta(TODO(zaid): not sure if we want to specify beta or just in general)

- Up to 1000 live inputs per-account.
- Up to 10 outputs may be configured per-live input.
- A maximum bitrate of 12000 kbps TODO(zaid): this is 2x twitch's max bitrate, not sure what we want here or if we even want to specify a maximum; main intent with declaring a limit is to prevent someone from jamming 4k at 240fps through connect.

TODO(zaid): how should the customer reach out if they have issues with these limits?


## Other endpoints

Along with the baseline endpoints, we also provide endpoints for managing inputs and their outputs.

### Get details on a live input

This returns all information associated with a specific input.

#### cURL example

```bash
curl -H "Authorization: Bearer $TOKEN" \
https://api.cloudflare.com/client/v4/accounts/$ACCOUNT/stream/live_inputs/$INPUT_UID
```

##### Example response

```bash
{
  "success": true,
  "errors": [],
  "messages": [],
  "result": {
    "uid": "66be4bf738797e01e1fca35a7bdecdcd",
    "meta": {
      "name": "test stream 1"
    },
    "created": "2014-01-02T02:20:00Z",
    "modified": "2014-01-02T02:20:00Z",
    "rtmps": {
      "url": "rtmps://live.cloudflare.com:443/live/",
      "streamKey": "<redacted>"
    }
  }
}
```

### List live inputs

This displays all inputs on your account.

To access rtmps information, request the details of the specific input.

Note that this api may be paginated in the future during the beta.

#### cURL example

```bash
curl -H "Authorization: Bearer $TOKEN" \
https://api.cloudflare.com/client/v4/accounts/$ACCOUNT/stream/live_inputs
```

##### Example response

```bash
{
  "result": [
    {
      "id": "01ad856eb76821fe32749626a4b39edf",
      "created": "2021-04-23T18:57:51.425387Z",
      "modified": "2021-04-23T18:57:51.425387Z",
      "meta": {}
    },
    {
      "id": "45a52d7847c95e8cd3de797f5da77afe",
      "created": "2021-04-23T19:02:22.828507Z",
      "modified": "2021-04-23T19:02:22.828507Z",
      "meta": {}
    },
    {
      "id": "e952c51fdcea70847e728f6145727bb6",
      "created": "2021-05-05T15:02:12.175868Z",
      "modified": "2021-05-05T15:02:12.175868Z",
      "meta": {}
    },
    {
      "id": "66be4bf738797e01e1fca35a7bdecdcd",
      "created": "2021-05-05T20:46:37.056086Z",
      "modified": "2021-05-05T20:46:37.056086Z",
      "meta": {}
    }
  ],
  "success": true,
  "errors": [],
  "messages": []
}
```

### List outputs for an input

Return all outputs configured to be retransmitted to by a specific live input

#### cURL example

```bash
curl -H "Authorization: Bearer $TOKEN" \
https://api.cloudflare.com/client/v4/accounts/$ACCOUNT/stream/live_inputs/$INPUT_UID/outputs
```

##### Example response
```bash
{
  "result": [
    {
      "uid": "6f8339ed45fe87daa8e7f0fe4e4ef776",
      "url": "rtmp://a.rtmp.youtube.com/live2",
      "streamKey": "<redacted>"
    },
    {
      "uid": "baea4d9c515887b80289d5c33cf01145",
      "url": "rtmp://a.rtmp.youtube.com/live2",
      "streamKey": "<redacted>"
    }
  ],
  "success": true,
  "errors": [],
  "messages": []
}
```

### Delete an output

This prevents the live input from retransmitting to the output identified by `$OUTPUT_UID`.

Note that if the associated live input is already retransmitting to this output when delete is called, that output will be disconnected within 2 minutes..

#### cURL example

```bash
curl -X DELETE \
-H "Authorization: Bearer $TOKEN" \
https://api.cloudflare.com/client/v4/accounts/$ACCOUNT/stream/live_inputs/$INPUT_UID/outputs/$OUTPUT_UID
```

### Delete a live input 

This deletes a live input and prevents any further operations against it.

Note that if an input is already receiving data to retransmit, the connection will be cut within 2 minutes.

#### cURL example

```bash
curl -X DELETE \
-H "Authorization: Bearer $TOKEN" \
https://api.cloudflare.com/client/v4/accounts/$ACCOUNT/stream/live_inputs/$INPUT_UID
```

