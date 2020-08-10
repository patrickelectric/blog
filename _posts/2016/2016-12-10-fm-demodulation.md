---
title: "FM with IQ Demodulation"
header:
    teaser: /assets/fm_demodulation/smiley1.jpg
tags: [ "Radio", "FM", "Math", "Demodulation" ]
---

After some time reading about hacking RF signals:

<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

*   [Using RTL-SDR to automatically receive weather satellite
images. ](http://www.asciimation.co.nz/bb/2014/03/19/using-rtl-sdr-to-automatically-receive-weather-satellite-images)
*   [Reverse engineering traffic lights with software defined radio.](https://hackaday.com/2015/09/20/reverse-engineering-traffic-lights-with-software-defined-radio/)
*   [Spectrum painting on 2.4 GHz.](https://hackaday.com/2015/08/22/spectrum-painting-on-2-4-ghz/)
*   [RTL-SDR Tutorial: Cheap ADS-B aircraft radar](http://www.rtl-sdr.com/adsb-aircraft-radar-with-rtl-sdr/)

I bought a RTL2832U receiver, that can operate on a range of 70-1700 Mhz.

![RTL](/assets/fm_demodulation/rtl-1024x576.jpg)

[Aliexpress link.](http://pt.aliexpress.com/item/USB2-0-Digital-DVB-T-SDR-DAB-FM-HDTV-TV-Tuner-Receiver-Stick-HE-RTL2832U-R820T/32357851768.html)

# Libraries and Programs

After some time (6 months, hell yeah Brazil...) to receive the package, I begun to install all the programs and libraries to use the RTL.

    $ sudo apt-get install gnuradio rtl-sdr gqrx-sdr

*   gnuradio : GNU Radio is a free software development toolkit that provides signal processing blocks to implement software-defined radios and signal processing systems. It
can be used with external RF hardware to create software-defined radios, or without hardware in a simulation-like environment. It is widely used in hobbyist, academic, and
commercial environments to support both wireless communications research and real-world radio systems. (ty wikipedia).

    ![gnuradio interface. (like simulink but isn't java)](/assets/fm_demodulation/gnuradio-1024x576.png)

*   rtl-sdr have:

`

     * rtl_adsb: a simple ADS-B decoder for RTL2832 based DVB-T receivers
     * rtl_eeprom: an EEPROM programming tool for RTL2832 based DVB-T receivers
     * rtl_fm: a narrow band FM demodulator for RTL2832 based DVB-T receivers
     * rtl_sdr: an I/Q recorder for RTL2832 based DVB-T receivers
     * rtl_tcp: an I/Q spectrum server for RTL2832 based DVB-T receivers
     * rtl_test: a benchmark tool for RTL2832 based DVB-T receivers

`

*   gqrx-sdr is a RF receive interface that have a displays FFT plot and spectrum waterfall very helpful to detect signals,  the software includes AM, SSB, FM-N, FM-W (mono
and stereo) demodulators and Special FM mode for NOAA APT.

![gqrx interface.](/assets/fm_demodulation/gqrx-1024x576.png)

After installed all the software and plugging the receiver on the computer, we can see that have a Realtek chip with the lsusb feedback.

    Bus 001 Device 004: ID 0bda:2838 Realtek Semiconductor Corp. RTL2838 DVB-T

Ok, now with everything done,  we can capture some data ! We can use the rtl-sdr toolkit.

<pre class="wiki">rtl_sdr capture.bin -s 1.8e6 -f 100.9e6</pre>

The capture.bin is the file that will have the IQ data, 1.8 MHz is the sample frequency and the frequency that we will look for is 100.9MHz (Rádio Atlântida), the radio
station.

Warning !! 1.8e6 samples for second, so take 3~10 seconds because the fast growing of the file.

# To demodulate we need do understand the modulation.

The FM is a method to modulate a message $m(t)$ with the change of the carrier <span class="MathJax_Preview">![cos(2\pi f_c t)]</span> <script type="math/tex">cos(2\pi f_c
t)</script> phase's <span
class="MathJax_Preview">![\phi(t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_56bc496f2eefa3002ee6d223c29c9fde.gif?w=474)</span><script
type="math/tex">\phi(t)</script>. [Wikipedia is your friend.](https://en.wikipedia.org/wiki/Frequency_modulation)

The phase have a relationship with the message, that is <span class="MathJax_Preview">![\phi(t) = 2\pi K_f \int^t_0
m(u)du](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_6b83d3eb197efdbec2148d2f5cc7f357.gif?w=474)</span><script type="math/tex">\phi(t) = 2\pi K_f
\int^t_0 m(u)du</script>. So the simple FM modulator will look like:

![FM modulator](/assets/fm_demodulation/fm_modulator-1024x434.png)

The <span class="MathJax_Preview">![s(t)](http://i1.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_4f37783823fc5bcb18cec232f7563904.gif?w=474)</span><script
type="math/tex">s(t)</script> data will be transmitted after the modulation.

# Demodulate FM, IQ Decomposition.

The RTL driver give to us the result of the QAM demodulator.

![QAM Demodulator](/assets/fm_demodulation/qam_demodulator-1024x411.png) Where:

<span class="MathJax_Preview">![s(t) = Acos(2\pi
f_ct+\phi(t))](http://i2.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_16356e909e03cbcbc0836ab3fd11eab1.gif?w=474)</span><script type="math/tex">s(t) = Acos(2\pi
f_ct+\phi(t))</script>
<span
class="MathJax_Preview">![cos(a+b)=cos(a)cos(b)-sin(a)*sin(b)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_3118bc215041d051756f88e2d18bf8ef.gif?w=474)</span><script
type="math/tex">cos(a+b)=cos(a)cos(b)-sin(a)*sin(b)</script>
<span class="MathJax_Preview">![s(t) = Acos(\phi(t))cos(2\pi f_ct) - Asin(\phi(t))sin(2\pi
f_ct)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_58416d3915c13b55a2d0210753885309.gif?w=474)</span><script type="math/tex">s(t) =
Acos(\phi(t))cos(2\pi f_ct) - Asin(\phi(t))sin(2\pi f_ct)</script>
<span class="MathJax_Preview">![s_I(t) =
Acos(\phi(t))](http://i2.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_d276b83fa58624d9e680cbebf1d1b3c6.gif?w=474)</span><script type="math/tex">s_I(t) =
Acos(\phi(t))</script>
<span class="MathJax_Preview">![s_Q(t) =
Asin(\phi(t))](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_7343408eb92dc0de86d8772fe6d67d52.gif?w=474)</span><script type="math/tex">s_Q(t) =
Asin(\phi(t))</script>
<span class="MathJax_Preview">![\phi(t) = \sphericalangle
(s_I(t)+js_Q(y))](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_db677c59f7c0d6b4974c961e1e755a81.gif?w=474)</span><script type="math/tex">\phi(t) =
\sphericalangle (s_I(t)+js_Q(y))</script>

You can read more about IQ data <span class="MathJax_Preview">![s_I(t)
s_Q(t)](http://i1.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_611c418992f5abe9f86e14debafd41b7.gif?w=474)</span><script type="math/tex">s_I(t)
s_Q(t)</script> [here](http://www.ni.com/tutorial/4805/en/).

Now we know how to demodulate the FM, with the equations above, the result is:
<span class="MathJax_Preview">![m(t) = \frac{1}{(2\pi K_f)} \frac{d}{dt}(\sphericalangle
(s_I(t)+js_Q(y))](http://i2.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_9747dccd22de368a2debc5ee4c346120.gif?w=474)</span><script type="math/tex">m(t) =
\frac{1}{(2\pi K_f)} \frac{d}{dt}(\sphericalangle (s_I(t)+js_Q(y))</script>

# Some code.

Now, with the knowledge of how the FM modulation and demodulation work, we can write a octave code that can show to us how the theory work in the real world.

```matlab
%read file and convert data
fid = fopen('./capture2.bin','rb');
y = fread(fid,'uint8=&amp;amp;amp;amp;amp;amp;gt;double');
y = y-127;
% transform IQ data to an imaginary number
yi = y(1:2:end)+i*y(2:2:end);

% the radio frequency and the sample frequency
freq = 100.9e6;
fs = 1.8e6;

% some math
yang = angle(yi);     % take the angle -pi to pi
yrap = unwrap(yang);  % make the correction to continue the angle after pi and -pi
tdev = diff(yrap);    % the derivative of theta
sound(tdev,fs)        % give us some music
```

You can download the capture.bin that I have used and the .m code to octave (open source rules) here:
[fm_demodulator_mono](/assets/fm_demodulation/fm_demodulator_mono.zip).

<audio class="wp-audio-shortcode" id="audio-120-1" preload="none" style="width: 100%;" controls="controls"><source type="audio/wav"
src="/assets/fm_demodulation/mono_fm.wav?_=1">[/assets/fm_demodulation/mono_fm.wav](/assets/fm_demodulation/mono_fm.wav)</audio>

# Stereo demodulation.

Ok, we have done the simpliest demodulation possible, without filters and a lot of things, so now we will use more energy to obtain a <del>very good </del> <del>good</del>
cool sound.

##### The FM spectrum

The FM spectrum have more things that I thought.

1.  30~15kHz L+R (mono), that we have already demodulated.
2.  19kHz, the omnipotent carrier.
3.  23~53kHz, a kind of dsb-sc (double side band suppressed-carrier) with L-R.
4.  55.35~58.65kHz, the RBDS (<del>Rebeldes</del> Radio Data system),  a kind of digital data system.
5.  58.65~76.65kHz, FTF first [read this](https://en.wikipedia.org/wiki/DirectBand), now ask yourself wth Microsoft is here ??
6.  92+<span class="MathJax_Preview">![\Delta
f_{as}](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_8ed7235e4e012bce81ca345a6a3ea653.gif?w=474)</span><script type="math/tex">\Delta
f_{as}</script>, [IDKWII](https://en.wikipedia.org/wiki/Subcarrier).

![tys wikipedia](/assets/fm_demodulation/FM-band.png)

<figcaption class="wp-caption-text">tys wikipedia</figcaption>

###### What is important ?

The FM modulation appears to be something like this:

[![](/assets/fm_demodulation/modulador_FM_tudo1.png?resize=474%2C199)](/assets/fm_demodulation/modulador_FM_tudo1.png)

Mathematically:

<span class="MathJax_Preview">![s(t) = cos(2\pi
f_c+\phi(t))](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_6c836c3ce0a026c8f7af559b3307bd25.gif?w=474)</span><script type="math/tex">s(t) = cos(2\pi
f_c+\phi(t))</script>
Where <span class="MathJax_Preview">![\phi(t) = 2 \pi K_f\int[m_R(t)+m_L(t)+ (m_R(t)-m_L(t))cos(4\pi f_c t)+RBDS(t)cos(6\pi f_c
t)]](http://i2.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_4809f8f5cd44e1b8d6ad5ea6ae932e07.gif?w=474)</span><script type="math/tex">\phi(t) = 2 \pi
K_f\int[m_R(t)+m_L(t)+ (m_R(t)-m_L(t))cos(4\pi f_c t)+RBDS(t)cos(6\pi f_c t)]</script>.

Yeah, this is a kind of crazy. The IQ data give to us the imaginary part that compose the <span
class="MathJax_Preview">![\phi(t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_56bc496f2eefa3002ee6d223c29c9fde.gif?w=474)</span><script
type="math/tex">\phi(t)</script>, theoretically making the FFT of <span
class="MathJax_Preview">![\frac{d}{dt}\phi(t)](http://i2.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_b7893f79356a6305f50f23d9aa9194bb.gif?w=474)</span><script
type="math/tex">\frac{d}{dt}\phi(t)</script> we can see something like the FM spectrum image.

```matlab
% fft
 tdfft = fft(tdev);
 P2=abs(tdfft/ysize);
 P1=P2(1:ysize/2+1);
 P1(2:end-1)=2*P1(2:end-1);
 f=fs*(0:(ysize/2))/ysize;
 % plot the fft until 59kHz
 freqe=round(2*59e3*size(f)/fs);
 freqe=freqe(2);
 plot(f(1:freqe),(P1(1:freqe)))
 grid on
 xlabel 'freq Hz',ylabel 'dB'
 hold on
 axis([30 59e3 0 0.02])

 Gab1 = [30 0; 30 0.02; 15e3 0.02; 15e3 0]; % R+L
 plot(Gab1(:,1),Gab1(:,2),'g');

 Gab1 = [18.5e3 0; 18.5e3 0.02; 19.5e3 0.02; 19.5e3 0]; % carrier
 plot(Gab1(:,1),Gab1(:,2),'b');

 Gab1 = [23e3 0; 23e3 0.02; 53e3 0.02; 53e3 0]; % R-L
 plot(Gab1(:,1),Gab1(:,2),'r');

 Gab1 = [55.35e3 0; 55.35e3 0.02; 58.65e3 0.02; 58.65e3 0]; % RBDS
 plot(Gab1(:,1),Gab1(:,2),'c');
```

And the result:

![fm_demo](/assets/fm_demodulation/spectrum_FM.png)

All good, all good sir !

##### The carrier and his friends.

Like said before, we have the IQ data that can give to us the <span
class="MathJax_Preview">![\phi(t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_56bc496f2eefa3002ee6d223c29c9fde.gif?w=474)</span><script
type="math/tex">\phi(t)</script> and with a derivative we can have <span class="MathJax_Preview">![m_R(t)+m_L(t)+ (m_R(t)-m_L(t))cos(4\pi f_c t)+RBDS(t)cos(6\pi f_c
t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_9dbb40352d1c6dc23d22845e9dc63490.gif?w=474)</span><script type="math/tex">m_R(t)+m_L(t)+
(m_R(t)-m_L(t))cos(4\pi f_c t)+RBDS(t)cos(6\pi f_c t)</script>. Applying a low-pass filter on the L+R, a band-pass filter on the carrier, L-R and RBDS we can have a clear
signal of each data.

With the frequencies specifications, the filter was done:

```matlab
%R+L
[b,a] = butter(10,2*16e3/fs,'low');
R_p_L = filter(b,a,tdev);
hhhh=R_p_L;
plote

%carrier
[b,a] = butter(4,[2*18.5e3,2*19.5e3]/fs,'bandpass');
carrier = filter(b,a,tdev);
hhhh=carrier;
plote

% R-L
[b,a] = butter(4,[2*23e3,2*53e3]/fs,'bandpass');
R_l_L = filter(b,a,tdev);
hhhh=R_l_L;
plote

%RBDS
[b,a] = butter(4,[2*55.35e3,2*58.65e3]/fs,'bandpass');
rbds = filter(b,a,tdev);
hhhh=rbds;
plote
```

The 'plote' code plot the fft of the signal.

<span class="MathJax_Preview">![m_R(t)+m_L(t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_26fa20951173011eb56e82a0520beefd.gif?w=474)</span><script type="math/tex">m_R(t)+m_L(t)</script>

[![rpl](/assets/fm_demodulation/rpl.png?resize=474%2C356)](/assets/fm_demodulation/rpl.png)

<span class="MathJax_Preview">![cos(2\pi f_c t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_fbe7de253c63357b060a5ba2c244065d.gif?w=474)</span><script type="math/tex">cos(2\pi f_c t)</script>

[![carrier](/assets/fm_demodulation/carrier.png?resize=474%2C356)](/assets/fm_demodulation/carrier.png)<span class="MathJax_Preview">![(m_R(t)-m_L(t))cos(4\pi f_c t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_02ecbfc1758b061f8ba5a2532b1cf418.gif?w=474)</span><script type="math/tex">(m_R(t)-m_L(t))cos(4\pi f_c t)</script>

[![rll](/assets/fm_demodulation/rll.png?resize=474%2C356)](/assets/fm_demodulation/rll.png)

<span class="MathJax_Preview">![RBDS(t)cos(6\pi f_c t)](http://i0.wp.com/patrickjp.com/wp-content/plugins/latex/cache/tex_6147ad2cc82784588b521e1a4a7514e4.gif?w=474)</span><script type="math/tex">RBDS(t)cos(6\pi f_c t)</script>

[![rbds](/assets/fm_demodulation/rbds.png?resize=474%2C356)](/assets/fm_demodulation/rbds.png)

To be continue..

