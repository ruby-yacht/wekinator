
//////////////////////////SIMPLE SENDER//////////////////////////
/////
///// Use this block to send data (inputs) to Wekinator
///// Run Wekinator with 2 inputs to receive this data
/////
/////////////////////////////////////////////////////////////////
(

//place cursor here and use shift cmd + return to execute

//variables declared here
var sender, sl, ix, iy;

//set up the sender
sender = NetAddr.new("127.0.0.1", 6448);

w = Window.new("Wekinator test", 400@400).front;

sl = Slider2D.new(w, Rect(25, 25, 350,350));
sl.action =
{
	|val|

	ix = val.x;
	iy = 1.0 - val.y;

	sender.sendMsg("/wek/inputs", ix, iy); //send the inputs
}


)


///////////////////////SIMPLE RECIEVER///////////////////
//
// Use this block to get synth control from Wekinator
// Place cursor in ( ) below and use cmd + return to execute the synth
// make sure the server is booted before running
// use cmd + . to stop the synth
//
// Run wekinator up with five continuous outputs and sending to port 57120
//
////////////////////////////////////////////////////////////////
(

var synth, responder;

synth = SynthDef("WekiSynth",

	{

	|carFreq = 440, carAmp = 0.2, modFreq = 20, modAmp = 50, panFreq = 0.5|

	var sig, mod, pan;

	modAmp = min(carFreq * 0.95, modAmp);
	mod = Saw.kr(modFreq, modAmp);
	sig = SinOsc.ar(carFreq + mod,0,carAmp);
	pan = SinOsc.kr(panFreq, 0,0.75);
	sig = Pan2.ar(sig, pan);
	Out.ar(0, sig);

	}

).play;

~receiver.free; //free the old reciever if it already exists
~receiver = NetAddr.new("127.0.0.1", 57120);    // create the NetAddr ... only the language port 57120 actually works
~responder.free; //if there is an old responder ... then free it first
~responder = OSCFunc(
	{ | msg |

		var specs = Array.newClear(5);

		msg.postln;

		specs[0] =  ControlSpec(100, 1000,\exp);
		specs[1] =  ControlSpec(0, 0.5,\lin);
		specs[2] = ControlSpec(0.2, 200,\lin);
		specs[3] = ControlSpec(20, 1000,\lin);
		specs[4] = ControlSpec(0.1, 20,\exp);

		synth.set(\carFreq, specs[0].map(msg[1]));
		synth.set(\carAmp, specs[1].map(msg[2]));
		synth.set(\modFreq, specs[2].map(msg[3]));
		synth.set(\modAmp, specs[3].map(msg[4]));
		synth.set(\panFreq, specs[4].map(msg[5]));

	},
'/wek/outputs');

)

