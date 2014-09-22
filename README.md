Pi-FM-DMA
=========


## FM transmitter using the Raspberry Pi

This program generates an FM modulation. It can transmit monophonic or stereophonic audio.

It is based on the FM transmitter created by [Oliver Mattos and Oskar Weigl](http://www.icrobotics.co.uk/wiki/index.php/Turning_the_Raspberry_Pi_Into_an_FM_Transmitter), and later adapted to using DMA by [Richard Hirst](https://github.com/richardghirst). [Christophe Jacquet](https://github.com/ChristopheJacquet) adapted it and added the RDS data generator and modulator. The transmitter uses the Raspberry Pi's PWM generator to produce VHF signals.

Supports any input sample rate and is able to produce stereo FM signal.

## How to use it?

Pi-FM-DMA, depends on the `sndfile` library. To install this library on Debian-like distributions, for instance Raspbian, run `sudo apt-get install libsndfile1-dev`.

Then clone the source repository and run `make` in the `src` directory:

```bash
git clone https://github.com/CRC-Mismatch/PiFmDma.git
cd PiFmDma/src
make
```

Then you can just run:

```
sudo ./pi_fm_dma
```

This will generate an FM transmission on 107.9 MHz without audio or radio-text. The radiofrequency signal is emitted through GPIO 4 (pin 7 on header P1).


You can add monophonic or stereophonic audio by referencing an audio file as follows:

```
sudo ./pi_fm_dma -audio sound.wav
```

To test stereophonic audio, you can try the file `stereo_44100.wav` provided.

The more general syntax for running Pi-FM-DMA is as follows:

```
pi_fm_dma [-freq freq] [-audio file] [-ppm ppm_error]
```

All arguments are optional:

* `-freq` specifies the carrier frequency (in MHz). Example: `-freq 107.9`.
* `-audio` specifies an audio file to play as audio. The sample rate does not matter: Pi-FM-DMA will resample and filter it. If a stereo file is provided, Pi-FM-DMA will produce an FM-Stereo signal. Example: `-audio sound.wav`. The supported formats depend on `libsndfile`. This includes WAV and Ogg/Vorbis (among others) but not MP3. Specify `-` as the file name to read audio data on standard input (useful for piping audio into Pi-FM-DMA, see below).
* `-ppm` specifies your Raspberry Pi's oscillator error in parts per million (ppm), see below.

### Clock calibration (only if experiencing difficulties)

**Technically, this is almost useless without RDS support, but since it still affects the sound signal, I have decided to maintain it from Jacquet's original implementation.**

The RDS standards states that the error for the 57 kHz subcarrier must be less than ± 6 Hz, i.e. less than 105 ppm (parts per million). The Raspberry Pi's oscillator error may be above this figure. That is where the `-ppm` parameter comes into play: you specify your Pi's error and Pi-FM-DMA adjusts the clock dividers accordingly.

In practice, I found that Pi-FM-DMA works okay even without using the `-ppm` parameter. I suppose the receivers are more tolerant than stated in the DMA spec.

One way to measure the ppm error is to play the `pulses.wav` file: it will play a pulse for precisely 1 second, then play a 1-second silence, and so on. Record the audio output from a radio with a good audio card. Say you sample at 44.1 kHz. Measure 10 intervals. Using [Audacity](http://audacity.sourceforge.net/) for example determine the number of samples of these 10 intervals: in the absence of clock error, it should be 441,000 samples. With my Pi, I found 441,132 samples. Therefore, my ppm error is (441132-441000)/441000 * 1e6 = 299 ppm, **assuming that my sampling device (audio card) has no clock error...**


### Piping audio into Pi-FM-DMA

If you use the argument `-audio -`, Pi-FM-DMA reads audio data on standard input. This allows you to pipe the output of a program into Pi-FM-DMA. For instance, this can be used to read MP3 files using Sox:

```
sox -t mp3 http://www.linuxvoice.com/episodes/lv_s02e01.mp3 -t wav -  | sudo ./pi_fm_dma -audio -
```


## Warning and Diclaimer

Never use this program to transmit VHF-FM data through an antenna, as it is
illegal in most countries. This code is for experimental purposes only.
Always connect a shielded transmission line from the RaspberryPi directly
to a radio receiver, so as **not** to emit radio waves.

I could not be held liable for any misuse of your own Raspberry Pi. Any experiment is made under your own responsibility.

Finally, I'm not to be held responsible for any damages caused (hardware or software-wise) to your Raspberry Pi. All code available here was lightly tested and deemed harmless until proven otherwise.


## Tests

Pi-FM-DMA was successfully tested with some FM receiving devices, namely:

* a RadioShack DX-375 portable receiver,
* a Samsung Galaxy S3 Mini using Spirit FM,
* a General Motors RDS-capable stock radio system.

Reception works perfectly with all the devices above, with only an almost unaudible hiss probably accountable to how the samples are being handled (upsampled, FIR filtered and such).
For now, the only major drawbacks are this hissing (light but still perceptible), some tiny clicking noises, and sub-par output volume if compared to official broadcast radio stations.

### CPU Usage

CPU usage is as follows:

* without audio: 9%
* with mono audio: 33%
* with stereo audio: 40%

CPU usage increases dramatically when adding audio because the program has to upsample the (unspecified) sample rate of the input audio file to 228 kHz, its internal operating sample rate. Doing so, it has to apply an FIR filter, which is costly.

## Design

The FM multiplex signal (baseband signal) is generated by `fm_mpx.c`. This file handles the upsampling of the input audio file to 228 kHz, and the generation of the multiplex: unmodulated left+right signal (limited to 15 kHz), possibly the stereo pilot at 19 kHz, possibly the left-right signal, amplitude-modulated on 38 kHz (suppressed carrier). Upsampling is performed using a zero-order hold followed by an FIR low-pass filter of order 60. The filter is a sampled sinc windowed by a Hamming window. The filter coefficients are generated at startup so that the filter cuts frequencies above the minimum of:
* the Nyquist frequency of the input audio file (half the sample rate) to avoid aliasing,
* 15 kHz, the bandpass of the left+right and left-right channels, as per the FM broadcasting standards.

The samples are played by `pi_fm_rds.c` that is adapted from Richard Hirst's [PiFmDma](https://github.com/richardghirst/PiBits/tree/master/PiFmDma). The program was changed to support a sample rate of precisely 228 kHz.

## History

* 2014-09-21: Initial version, simplified from Christophe Jacquet's [PiFmRds](https://github.com/richardghirst/PiBits/tree/master/PiFmDma).

--------

© CRC-Mismatch, 2014. Released under the GNU GPL v3.
