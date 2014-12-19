#Send a song to your friend through a phone call

Last week we sponsored [The Next Web hackathon](http://thenextweb.com/conference/usa/hack-battle) in NYC. We met a developer, Herb, who had an idea to use the Deezer API to send a song to a friend over the phone. We couldn't get it to work that weekend, but now we figured it out and I want to share with you how to manipulate the AudioStreams in the Sinch JS SDK. So, here you go [Ivo](https://twitter.com/ilukac)!

## Preparation
1. [Sign up](#signup) for a Sinch account
2. Download the JS SDK from [http://sinch.com/download/](https://www.sinch.com/downloads/#downloads-javascript)
3. Click 'create new app'
4. Name your app, and click 'create'
5. Take note of your app key and secret, you will need them in a few minutes

## Time to Code
First, add your favorite song to the folder. I am adding [Something New](https://www.youtube.com/watch?v=BhJSsX5AKPI) by Axwell and Ingrosso, and naming it `somethingnew.m4a`

Open up `index.html` and add an audio element to the page just under the incoming tag:

```
<audio id="song" src='somthingnew.m4a'></audio>
```

Open up the `SinchPSTNsample/PSTNSample.js`, and enter your app key on line 24:

```
sinchClient = new SinchClient({
	applicationKey: 'your appkey',
	capabilities: {calling: true},
	//supportActiveConnection: true, /* NOTE: This is only required if application is to receive calls / instant messages. */ 
	//Note: For additional logging, please uncomment the three rows below
	onLogMessage: function(message) {
		console.log(message);
	},
});
```
##Add mediastream with two channels
I want to play the song and be able to speak to my friend at the same time. The starting point for all HTML5 sound is the audio context, so let’s grab that first:

```
/*** add your own mix stream***/
var audioCtx = new AudioContext();
```

Next, we want to create a dynamics compressor; this will level the volume on the tracks so you hear the song and the speech. For more info about this see [http://www.w3.org/TR/webaudio/#DynamicsCompressorNode](http://www.w3.org/TR/webaudio/#DynamicsCompressorNode)

```
var compressor = audioCtx.createDynamicsCompressor();
```

Next, create a source for the song and the microphone and hook it up to a media destination:

```
//this will be the stream we broadcast in our call.

var destination = audioCtx.createMediaStreamDestination(); 
var song = audioCtx.createMediaElementSource($('audio#song')[0]);
song.connect(compressor);

// get access to the mic, and connect that to the compressor
navigator.getUserMedia({video: false, audio: true}, function(stream) {
	var mic = audioCtx.createMediaStreamSource(stream);
	mic.connect(compressor);
	compressor.connect(destination);
	
}, function(error) {
		console.log(error);
});
```

Next, we need to change the call function to broadcast our custom stream instead of just hooking up to the microphone.

Change `call = callClient.callPhoneNumber()` to this:

```
call = callClient.callPhoneNumber($('input#phoneNumber').val(), {}, destination.stream);
```

Now, we want to start playing the song as soon as the other party answers. In `onCallEstablished` add 

```
$('audio#song').trigger("play");
```

and in `onCallEnded` add

```
$('audio#song').trigger("stop");
```

When running this, remember to fully quit Chrome and start with `--allow-file-access-from-files` like so:


**Windows**

```
path/to/chrome yourfilename.html --allow-file-access-from-files
```


**Mac**

```
open -a path/to/chrome yourfilename.html --args --allow-file-access-from-files
```

