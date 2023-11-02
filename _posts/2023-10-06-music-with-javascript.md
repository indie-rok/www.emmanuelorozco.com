---
layout: post
title: Creating Music with Javascript and React
date: 2023-10-06 18:30:00 +0000
tags:
  - tech
  - javascript
  - music
---

Last week, I went to buildspace’s hackathon and I set a goal:

> To work on something that I really really like. Music.

So, for 3 days, I worked an app that generates chord progressions and mix them with peoples voice.

The interesting part:

> Chord progressions were generated using code.

Particulary a javascript framework called _Tone.js_

The documentation it’s pretty bad, so I’ve decided to make this post to quickly show people how to start very easly.

So first: install the [npm library](https://www.npmjs.com/package/tone) or use the javascript file.

> Warning:
> You can’t play any sounds unless user makes an action. Click, hover, etc. onLoad is not supported.

Then to generate music:

- **Generating single notes**

A single note is pretty straight forward. Just use the triggerAttackRelase method with your note and you are good to go. I’ll talk about the “1n” in a minute.

```javascript
const synth = new Tone.PolySynth(Tone.Synth).toDestination();

synth.triggerAttackRelease("C4", "1n");

Tone.Transport.start();
```

- #### Chords

It’s very similar to a single note generation, but the difference it’s that you can provide an array to .triggerAttackRelase to play the whole chord. You need Tone.PolySynth (instead of Tone.Note) to be able to play multiple notes at the same time (chord).

```javascript
const synth = new Tone.PolySynth(Tone.Synth).toDestination();
const C_MAYOR = ["C4", "E4", "G4"];
synth.triggerAttackRelease(C_MAYOR, "1n");

Tone.Transport.start();
```

- #### Generating chord progressions

A chord progression has 3 or 4 chords in it, so, to play them one after another:

```javascript
const synth = new Tone.PolySynth(Tone.Synth).toDestination();

const now = Tone.now();

synth.triggerAttackRelease(C_MAYOR, 2, now);
synth.triggerAttackRelease(G_MAYOR, 2, now + 2);
synth.triggerAttackRelease(F_MAYOR, 2, now + 4);
synth.triggerAttackRelease(A_MINOR, 2, now + 6);

Tone.Transport.start();
```

First param, is the chord,
Second param is the length (in seconds) of the chord.
Thrid param is the length (in seconds) of when to start playing (so this will play a chord in second 0 , 2 ,4 and 6).

- #### Looping through chord progressions

But the point of a chord progression is loop it. I tried to use Tone.Loop but I was not sucesful. Tone.Part was my best ally.

Tone.Part it’s like a javascript map that will execute an function per each element of the progression array.

I’ll explain the “0:0:0”, “0:4:0”, etc. in a minute.

```javascript
const synth = new Tone.PolySynth(Tone.Synth).toDestination();

function playProgression() {
        const progression = [
            ["0:0:0", ["C4", "E4", "G4"]],
            ["0:4:0", ["A4", "C5", "E5"]],
            ["0:8:0", ["F4", "A4", "C5"]],
            ["0:12:0", ["G4", "B4", "D5"]],
        ];

        const part = new Tone.Part((time, chord) => {
            synth.triggerAttackRelease(chord, "1n", time);
        }, progression);

        Tone.Transport.start()

        part.loop = true;
        part.loopEnd = "0:16:0";  // Loop every bar
        part.start() // Don't forget this or the progresion will never work
    }
}
```

- #### Sync visuals with chord progressions

In my app, every time I played a different chord, I turn a particular block of color to make it easier to recognize that it was a new sound. To trigger this you just need to add a callback (with Tone.Draw) like this:

```javascript
const synth = new Tone.PolySynth(Tone.Synth).toDestination();

const part = new Tone.Part((time, chord) => {
  synth.triggerAttackRelease(chord, "1n", time);

  Tone.Draw.schedule(function () {
    // you can set state or DOM call here
  }, time);
}, progression);
```

- #### Recording user input

Tone.js includes a wrapper around the javascript audio microphone api. To use it with react:

```javascript
async function startRecord() {
  // without this recording does not work
  await Tone.start();
  try {
    await navigator.mediaDevices.getUserMedia({ audio: true });
  } catch (err) {
    console.error("Microphone access denied:", err);
    return;
  }

  // this ref is optional if you are working with react,
  // it will allow you keep the Tone.UserMedia between re renders
  micRef.current = new Tone.UserMedia();

  try {
    await micRef.current.open();
    console.log("Microphone open!");

    recorderRef.current = new Tone.Recorder();

    // Mic is open but does not record. Tone Recorder does that
    micRef.current.connect(recorderRef.current);
    recorderRef.current.start();
    console.log("Recording started...");
  } catch (err) {
    console.error("Error opening microphone:", err);
  }
}

async function stopRecord() {
  const buffer = await recorderRef.current.stop();

  let blobUrl = URL.createObjectURL(buffer);

  // play ref will be used later to play the recording
  playerRef.current = new Tone.Player(blobUrl, () => {
    console.log("buffer loaded");
  }).toDestination();
}

async function playRecording() {
  await Tone.start();
  playerRef.current.start();
}
```

- #### Using other instruments (piano, guitar, etc)

I did not liked the default instruments so I changed them. I also needed them to be available while being offline, so I could not use a URL with all the audio files. Instead I just downloaded the instruments from this repo, added them to the public file of my app and then initialized:

```javascript
import C4 from "/public/piano/C4.mp3";
import D4 from "/public/piano/D4.mp3";

// instead of Tone.Polysynth
const piano = new Tone.Sampler({
  C4,
  D4,
  E4,
  F4,
  G4,
  A4,
  B4,
  C5,
  D5,
  E5,
}).toDestination();
```

Worth mentioning that Tone.Sampler **is already capable** of playing multiple notes at the same time. So no more Tone.Polysynth

So that’s it!

---

Hope it helps, and if you are working in any product about music and javascript, please tell me more:

#### schedule a meeting with me, I would love to learn more and help if I can!

[My Calendar](https://cal.com/emmanuel-orozco)
