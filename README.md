# ERC-Q3Signal Recovery Report
Electronics and Robotics Club Technical Assignment

 This document explains what I found at each stage, what I think was done to the signal, and what I did to fix it.

Tools I used
-	Python 3
-	NumPy, specifically np.fft for computing the FFT
-	SciPy, for reading the wav file, designing filters and applying them
-	Matplotlib for plotting graphs
-	Google Colab as the environment to run everything
-	IPython.display.Audio to listen to the audio at each stage

Stage 1 - Understanding what I received
What I did
I loaded the wav file and plotted two things. First the time domain waveform which shows amplitude over time, and second the FFT which shows where the energy of the signal is sitting in terms of frequency.
What I found
The time domain plot looked like complete noise. There was no visible structure or pattern. The FFT is where I actually got useful information. Normal speech sits between 0 and 4000 Hz. But when I plotted the FFT of the corrupted signal I saw no energy there at all. Instead I found these sharp spikes at completely wrong frequencies:
-	around 7500 Hz which was the biggest spike by far
-	around 2500 Hz
-	around 6000 Hz
-	around 9500 Hz
-	around 13500 Hz 
 
 
What I concluded
The signal had been frequency shifted. All the audio energy was pushed up to around 7500 Hz. This happens when you multiply a signal by a high frequency cosine wave, which is called AM modulation. So the carrier frequency used was 7500 Hz.

Stage 2 - Bringing the signal back to the right frequency
What I think was done to the signal
The original audio was multiplied by a cosine wave at 7500 Hz before it was sent. This is AM modulation. In simple terms:

corrupted = audio x cos(2 x pi x 7500 x t)

Why multiplying again works
There is a basic trig identity that says cos(x) times cos(x) equals half plus half of cos(2x). So when I multiply the corrupted signal by the same carrier again I get the original audio back at low frequency plus some high frequency leftovers. The low pass filter then removes those leftovers and I am left with just the audio.
What I did in code
-	Made a cosine wave at 7500 Hz
-	Multiplied the corrupted signal by this cosine
-	Applied a Butterworth low pass filter with cutoff at 4000 Hz using scipy.signal.butter and filtfilt
 
What happened after stage 2
The FFT now showed the energy had moved back to 0 to 4000 Hz which is the correct range for speech. But I could still hear a loud continuous beep in the audio and there were still some sharp spikes in the FFT. So there was more to fix.

Stage 3 - Removing the noise tones
What I found
I zoomed into the 0 to 5000 Hz region of the FFT and also ran automatic peak detection. I found four sharp spikes that do not belong in real audio:
-	200 Hz which was the strongest one and the source of the beep sound
-	1400 Hz
-	2000 Hz
-	4300 Hz
Real audio looks like a smooth spread out shape in the FFT. These thin sharp spikes were clearly artificial tones that had been added on top.
What I think was done
Pure sine wave tones were added at those specific frequencies to corrupt the signal. Like someone playing four different single note beeps on top of the actual audio.
What I did to fix it
I used notch filters. A notch filter removes energy at one specific frequency without touching anything around it. I used scipy.signal.iirnotch with a Q value of 40 which makes the cut very narrow and precise. I applied four separate notch filters one for each spike.
 
What happened after stage 3
The FFT looked clean after this. No more sharp spikes, just smooth audio energy. The loud beep was gone too. But when I listened carefully something still felt slightly off about the audio, so I moved on to stage 4.

Stage 4 - Fixing the phase
The hint
The assignment said to think beyond amplitude. A signal has two main properties which are amplitude and phase. The FFT only shows amplitude information so if something was done to the phase it would not show up in the FFT at all. That matched what I was seeing.
What I think was done
The signal was phase inverted, meaning every sample was multiplied by negative 1. This flips the entire waveform upside down. It does not change the frequency content which is why the FFT looked fine, but the signal is still technically corrupted.
How I found the fix
I tested three possibilities. Phase inversion which is multiplying by negative 1, time reversal which is flipping the array backwards, and both together. I listened to all three. The phase inverted version was clearly the correct one. The fix was just one line:

stage4 = -stage3

Result
Still the audio is not that clear it has some noise in the background after doing all this steps .
I think in stage 3 I did something wrong but that is what I thought .

Summary of what was done and how I fixed it
Stage	What was done to the signal	How I detected it	What I did to fix it
2	AM modulation at 7500 Hz	Big FFT spike at 7500 Hz, nothing in 0 to 4000 Hz	Multiply by cos(7500t) again then low pass filter at 4000 Hz
3	Sine tones added at 200, 1400, 2000 and 4300 Hz	Sharp narrow spikes in FFT and audible beeping	Applied 4 notch filters using iirnotch with Q of 40
4	Phase inversion, every sample multiplied by -1	FFT looked clean but audio still sounded wrong	Flipped the sign: stage4 = -stage3

