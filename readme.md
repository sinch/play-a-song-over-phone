#Sending a song to your friend thru a phone call

Last week we where sponsoring thenextweb hackathon in NYC, one idea from Herb from there was to user the Deezer API to send a song to a friend over the phone. We couldn't really get it to work that weekend. But now we figured it out and I wanted to share it with you guys on how you can manipulate the AudioStreams in the Sinch JS SDK. So here you go Herb.

## Preparation
1. Download the JS SDK from http://sinch.com/download/
2. Click create new app
3. Name your app, and click create
4. Take note of your app key and secret, you will need them in a few minutes

## Lets code
First add a your favorite song to the folder, I am adding something new with Axwell and Ingrosso, and naming it somethingnew.m4a 

Open up index.html and add a audio element to the page just under the incoming tag
```
<audio id="song" src='somthingnew.m4a'></audio>
```

Open up the SinchPSTNsample/PSTNSample.js

on row 24 enter your app key

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
###Add mediastream with two channels
I want to play the song and be able to speak to my friend and the same time, the starting point for all html 5 sound is the audio context so lets grab that first
/*** add your own mix stream***/
```
var audioCtx = new AudioContext();
```

Next we want to crate a dynamicscompressor, this will level the volume on the tracks so you both hear the song and and speech. For more info about this see [http://www.w3.org/TR/webaudio/#DynamicsCompressorNode]()
```
var compressor = audioCtx.createDynamicsCompressor();
Next we want to create a source for the song and the microphone and hook it up to a media destination
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

Next we need to change or call function to broadcast our customstream instead of just hooking up the microphone.

Change call = callClient.callPhoneNumber() to below
```
call = callClient.callPhoneNumber($('input#phoneNumber').val(), {}, destination.stream);
```

So the scenario we have now we want to start playing the song as soon as the other party answers in  onCallEstablished add 
```
$('audio#song').trigger("play");
```

and in onCallEnded add
```
$('audio#song').trigger("stop");
```

Remember to start chrome with the --allow-file-access-from-files
```
open /Applications/Google\ Chrome.app --args --allow-file-access-from-files "/Users/christianjensen/Downloads/sinchjssdk/samples/SinchPSTNsample/index.html"
```
