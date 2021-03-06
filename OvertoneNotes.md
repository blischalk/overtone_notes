# Overtone

### Install
- Create Leiningen project
- Add overtone to project.clj

		(defproject tutorial "0.1.0-SNAPSHOT"
  			:description "FIXME: write description"
  			:url "http://example.com/FIXME"
  			:license {:name "Eclipse Public License"
            		  :url "http://www.eclipse.org/legal/epl-v10.html"}
            :dependencies [[org.clojure/clojure "1.5.1"]
                           [overtone "0.9.1"] ])
                           
### Connect to an internal server:

`(use 'overtone.live)`

### Define a Synth
`(definst foo [] (saw 220))`

### Start a Synth
`(foo)`

### Kill a Synth
`(kill num-of-synth)` e.g 4 or some number

`(kill foo)` kill all instances of foo

### Unit-generator or ugen
The saw function represents a unit-generator or ugen.  These are the basic building blocks for creating synthesizers, and they can generate or process both audio and control signals.

### Check the documentation
`(odoc saw)`

### Trigger multiple synths
	> (definst baz [freq 440] (* 0.3 (saw freq)))
	> (baz 220)
	> (baz 660)
	> (kill baz) ; stop all running synths
	
	
## (stop) KILL ALL RUNNING SYNTHS!

## Adjust Synth Volume
When the synth is running the ugen is outputing an audio signal. This signal is represented as a continuous stream of floating point values between -1 and 1, so by multiplying by 0.3 we are just lowering the amplitude of this signal, adjusting the volume.

## Change Parameters of a Running Synth on the Fly

	> (definst quux [freq 440] (* 0.3 (saw freq)))
	> (quux)
	> (ctl quux :freq 660)
	
## Input Signals

So far we've just used a single ugen and passed it arguments, but ugens can be plugged into each other in arbitrary ways too. Basically anywhere a ugen takes an argument value it can also take an input signal that will control that value. Lets put a tremolo on our saw wave:

	> (definst trem [freq 440 depth 10 rate 6 length 3]
    	(* 0.3
      	 (line:kr 0 1 length FREE)
      	 (saw (+ freq (* depth (sin-osc:kr rate))))))
	> (trem)
	4
	> (kill trem)
	> (trem 200 60 0.8)
	4
	> (kill trem)
	> (trem 60 30 0.2)
	
## `line` ugen

The line ugen outputs a value from the start value to the end value over a specific length of time. In this example we use it as a simple way to stop the synth after a few seconds. The last argument, FREE, is a typical argument to this kind of control ugen that tells the ugen to free the whole synth instance once it completes. That way you don't have to kill it by hand anymore.

Also note that the line and sin oscillator ugens have the :kr suffix. Many of the ugens can operate at different rates, shown at the bottom of the doc string. The two primary rates are audio rate and control rate, :ar and :kr. Audio rate runs at the rate of your audio card, and control rate runs at about 1/60 of that speed, so for signals that are just controlling a value rather than outputting audio you can save wasted CPU by using :kr.

## Playing Samples

Most of the time you'll want to load a sample and return a function that can be used to trigger the sample. This can be done with (sample <path>).

	(def flute (sample "/home/rosejn/studio/samples/flutes/flutey-echo-intro-blast.wav"))

	(flute)
	
## Loading a Sample Into a Buffer

You can load a wav file into an audio buffer using (load-sample <path>):

	(def flute-buf (load-sample "/home/rosejn/studio/samples/flutes/flutey-echo-intro-blast.wav"))
	
Buffers are in-memory storage spots for audio files, and they can be either read from disk or generated by synths. You can view the contents of an audio buffer using the (scope :buf <buffer>) function:

	(scope :buf flute-buf)
	
## Chords and Scales

	;; We use a saw-wave that we defined in the oscillators tutorial
	(definst saw-wave [freq 440 attack 0.01 sustain 0.4 release 0.1 vol 0.4] 
	  (* (env-gen (env-lin attack sustain release) 1 1 0 1 FREE)
	     (saw freq)
	     vol))
	
	;; We can play notes using frequency in Hz
	(saw-wave 440)
	(saw-wave 523.25)
	(saw-wave 261.62) ; This is C4
	
	;; We can also play notes using MIDI note values
	(saw-wave (midi->hz 69))
	(saw-wave (midi->hz 72))
	(saw-wave (midi->hz 60)) ; This is C4
	
	;; We can play notes using standard music notes as well
	(saw-wave (midi->hz (note :A4)))
	(saw-wave (midi->hz (note :C5)))
	(saw-wave (midi->hz (note :C4))) ; This is C4! Surprised?
	
	;; Define a function for convenience
	(defn note->hz [music-note]
	    (midi->hz (note music-note)))
	
	; Slightly less to type 
	(saw-wave (note->hz :C5))
	
	;; Let's make it even easier
	(defn saw2 [music-note]
	    (saw-wave (midi->hz (note music-note))))
	
	;; Great!
	(saw2 :A4)
	(saw2 :C5)
	(saw2 :C4)
	
	;; Let's play some chords
	
	
	;; this is one possible implementation of play-chord
	(defn play-chord [a-chord]
	  (doseq [note a-chord] (saw2 note)))
	
	;; We can play many types of chords.
	;; For the complete list, visit https://github.com/overtone/overtone/blob/master/src/overtone/music/pitch.clj and search for "def CHORD"
	(play-chord (chord :C4 :major))
	
	;; We can play a chord progression on the synth
	;; using times:
	(defn chord-progression-time []
	  (let [time (now)]
	    (at time (play-chord (chord :C4 :major)))
	    (at (+ 2000 time) (play-chord (chord :G3 :major)))
	    (at (+ 3000 time) (play-chord (chord :F3 :sus4)))
	    (at (+ 4300 time) (play-chord (chord :F3 :major)))
	    (at (+ 5000 time) (play-chord (chord :G3 :major)))))
	
	(chord-progression-time)
	
	;; or beats:
	(defonce metro (metronome 120))
	(metro)
	(defn chord-progression-beat [m beat-num]
	  (at (m (+ 0 beat-num)) (play-chord (chord :C4 :major)))
	  (at (m (+ 4 beat-num)) (play-chord (chord :G3 :major)))
	  (at (m (+ 8 beat-num)) (play-chord (chord :A3 :minor)))
	  (at (m (+ 14 beat-num)) (play-chord (chord :F3 :major)))  
	)
	
	(chord-progression-beat metro (metro))
	
	;; We can use recursion to keep playing the chord progression
	(defn chord-progression-beat [m beat-num]
	  (at (m (+ 0 beat-num)) (play-chord (chord :C4 :major)))
	  (at (m (+ 4 beat-num)) (play-chord (chord :G3 :major)))
	  (at (m (+ 8 beat-num)) (play-chord (chord :A3 :minor)))
	  (at (m (+ 12 beat-num)) (play-chord (chord :F3 :major)))
	  (apply-at (m (+ 16 beat-num)) chord-progression-beat m (+ 16 beat-num) [])
	)
	(chord-progression-beat metro (metro))
	
### Alter Running Ugen Params

	; Define an instrument with params
	(definst foo [vol 1 freq 220] (* vol (saw freq)))
	; Start it running
	(foo)
	; In 1 sec, change all foos frequency
	(at (+ (now) 1000) (ctl foo :freq 60))

### Filters

Overtone comes with a number of standard linear filters: lpf, hpf, and bpf are low-pass, high-pass and band-pass filters respectively.

	(demo 10 (lpf (saw 100) (mouse-x 40 5000 EXP)))
	;; low-pass; move the mouse left and right to change the threshold frequency
	
	(demo 10 (hpf (saw 100) (mouse-x 40 5000 EXP)))
	;; high-pass; move the mouse left and right to change the threshold frequency
	
	(demo 30 (bpf (saw 100) (mouse-x 40 5000 EXP) (mouse-y 0.01 1 LIN)))
	;; band-pass; move mouse left/right to change threshold frequency; up/down to change bandwidth (top is narrowest)


You can do Karplus-Strong string synthesis with the pluck filter. Karplus-Strong works by taking a signal, filtering it and feeding it back into itself after a delay, so that the output eventually becomes periodic.

	;; here we generate a pulse of white noise, and pass it through a pluck filter
	;; with a delay based on the given frequency
	(let [freq 220]
	   (demo (pluck (* (white-noise) (env-gen (perc 0.001 2) :action FREE)) 1 3 (/ 1 freq))))
	   
### Multi-channel Expansion
- Use out to send audio to a channel
- Multiply by a value when layering multiple oscillators so they add up to 1.


		(defsynth sin-square [freq 440] 
	  		(out 0 (* 0.5 (+ (square (* 0.5 freq)) (sin-osc freq))))
	  		(out 1 (* 0.5 (+ (square (* 0.5 freq)) (sin-osc freq)))))
	  		
This can be come cumbersome.  Supercollider provides Multi-channel Expansion:

	(sin-osc [440 443])
	
	; Expanded into two oscs with two filters
	(lpf (sin-osc [440 443]) 600)
	
The above synth can be re-written as:

    ; Plays both oscs out of both channels
	(defsynth sin-square2 [freq 440] 
  		(out 0 (* [0.5 0.5] (+ (square (* 0.5 freq)) (sin-osc freq)))))
  		
Or:

    ; Plays square on left and sin on right
	(defsynth sin-square2 [freq 440] 
  		(out 0 (* 0.5 [(square (* 0.5 freq)) (sin-osc freq)])))
  		
### Oscillators

	(definst sin-wave [freq 440 attack 0.01 sustain 0.4 release 0.1 vol 0.4] 
	  (* (env-gen (lin attack sustain release) 1 1 0 1 FREE)
	     (sin-osc freq)
	     vol))
	(sin-wave)

	(definst saw-wave [freq 440 attack 0.01 sustain 0.4 release 0.1 vol 0.4] 
	  (* (env-gen (lin attack sustain release) 1 1 0 1 FREE)
	     (saw freq)
	     vol))
	
	(definst square-wave [freq 440 attack 0.01 sustain 0.4 release 0.1 vol 0.4] 
	  (* (env-gen (lin attack sustain release) 1 1 0 1 FREE)
	     (lf-pulse:ar freq)
	     vol))
	
	(definst noisey [freq 440 attack 0.01 sustain 0.4 release 0.1 vol 0.4] 
	  (* (env-gen (lin attack sustain release) 1 1 0 1 FREE)
	     (pink-noise) ; also have (white-noise) and others...
	     vol))
	
	(definst triangle-wave [freq 440 attack 0.01 sustain 0.1 release 0.4 vol 0.4] 
	  (* (env-gen (lin attack sustain release) 1 1 0 1 FREE)
	     (lf-tri freq)
	     vol))

	(definst spooky-house [freq 440 width 0.2 
	                         attack 0.3 sustain 4 release 0.3 
	                         vol 0.4] 
	  (* (env-gen (lin attack sustain release) 1 1 0 1 FREE)
	     (sin-osc (+ freq (* 20 (lf-pulse:kr 0.5 0 width))))
	     vol))
	     
#### Synth Forumla
lin, env-gen, osc is basic asdr synth.


### Recording

	(recording-start "~/Desktop/foo.wav")
	;; make some noise. i.e.
	(demo (pan2 (sin-osc)))
	;; stop recording
	(recording-stop)
	
### Saving Buffer to Disk

	(buffer-save b "~/Desktop/bong.wav") 
	
### definst vs defsynth

definst is higher level than defsynth.  definst automatically takes care of routing the output to l and r channels.  Instruments will automatically wrap the body of code given in an out ugen.