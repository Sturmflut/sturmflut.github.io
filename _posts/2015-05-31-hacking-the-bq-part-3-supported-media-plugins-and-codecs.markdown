---
layout: post
title:  "Hacking the bq, Part 3: Supported media plugins and codecs"
date:   2015-05-31 15:00:00
categories: Ubuntu bq
---

*UPDATE 31.05.2015: The original list was incomplete and misleading. I have thus completely rewritten this article.*

You might have wondered which media filetypes and codecs your bq phone supports. Since the list of available plugins depends on the image version and maybe also the device (not every device has the same hardware decoders), I didn't file this article under the "Hacking the Ubuntu Touch" series for the moment.


## Plugins

The backend for media playback on Ubuntu Touch is the gstreamer infrastructure. It supports a very long list of plugins for various purposes. I went and installed `gst-inspect` on my OTA-3.5 phone so you don't have to modify your image.

I made a commented list of all plugins because you might want to build an audio or video app and want to know what you can do with the existing possibilities.

The phone currently comes with the following gstreamer plugins:


* *1394:* Firewire input source (useless on the phone)

* *aasink:* ASCII art output

* *adder:* Mixes multiple streams by adding them.

* *alaw:* An "A Law" audio format decoder/encoder.

* *alpha:* Adds an alpha channel to a video.

* *alphacolor:* Converts between different colorspaces, preserving the alpha channel.

* *androidmedia:* A plugin written by Canonical, it uses libhybris to tap into the Android media decoders. This is necessary to leverage hardware decoding. Currently decodes H.264/AVC, DivX, H.263, MPEG-4, Sorenson H.263.1 (s263) and Xvid streams.

* *apetag:* Read APE audio format tags.

* *app:* Gives raw access to streams.

* *audioconvert:* Converts audio.

* *audiofx:* Applies audio effects. The following are currently available: amplification, band pass/band reject, low/high pass, dynamic range, echo, FIR, IIR, inversion, karaoke (?), stereo positioning, tempo scaling.

* *audioparsers:* Knows how to parse AAC, AC3, AMR, DTS, FLAC, MPEG1, SBC and WavPack streams.

* *audiorate:* Adjust audio rate.

* *audioresample:* Resamples audio.

* *audiotestsrc:* An audio test source.

* *auparse:* Parses the AU audio format.

* *autodetect:* Auto-detects available audio/video in- and outputs.

* *avi:* Demultiplexes AVI files into audio, video and subtitle streams.

* *cacasink:* Colored ASCII art output.

* *cairo:* Renders overlay on a video stream using Cairo.

* *camerabin:* Take still images and videos from a hardware camera.

* *cdparanoia:* Rip audio CDs. Most likely useless on a phone :)

* *coreelements:* Contains the gstreamer core elements.

* *cutter:* Cuts audio.

* *debug:* Contains multiple elements useful for debugging.

* *deinterlace:* A deinterlacer for interlaced (e.g. 576i SD) content.

* *dtmf:* Generates DTMF tones and knows how to generate DTMF packets for RTP (VoIP).

* *dv:* Demultiplexes and decodes the DV (Digital Video Camera) format.

* *effectv:* Video effects from the effectv project.

* *encoding:* Contains a convenience element for stream multiplexing/encoding.

* *equalizer:* equalizer with 3, 10 or N bands.

* *faad:* Decodes AAC audio.

* *flac:* Encodes, decodes and tags FLAC audio files.

* *flump3dec:* MP3 Decoder by Fluendo.

* *flv:* Demultiplexes and Multiplexes the FLV (Flash Video) file format.

* *flxdec:* Decodes video files in FLC/FLI/FLX formats.

* *gdkpixbuf:* Decodes, overlays and outputs images using GdkPixbuf.

* *gio:* Reads and writes streams using GIO.

* *goom:* Audio visualisation using the GOOM filter.

* *goom2k1:* Audio visualisation using the GOOM 2k1 filter.

* *icydemux:* Reads and outputs ICY tags (often present in HTTP audio streams) and outputs them while demuxing the content.

* *id3demux:* Reads and outputs ID3 tags (usually present in MP3 streams) and outputs them while demuxing the content.

* *imagefreeze:* Generates a stream of still images from an existing video stream.

* *interleave:* Folds many mono channels into one interleaved audio stream.

* *isomp4:* Demultiplexes and multiplexes 3GPP, ISML, MJ2, MP4 and Quicktime streams.

* *jack:* Allows audio input and output using the JACK audio daemon.

* *jpeg:* Decodes and encodes JPEG images.

* *jpegformat:* Demultiplexes and parses JPEG streams.

* *level:* Measures the peak level of an audio stream.

* *matroska:* Demultiplexes Matroska and WebM streams, parses and multiplexes Matroska streams.

* *mirsink:* Outputs video via the Mir video server.

* *monoscope:* Visualises audio streams like an oscilloscope.

* *mulaw:* A "Mu Law" audio format decoder/encoder.

* *multifile:* Reads and writes from/to sequentially named files.

* *multipart:* Demultiplexes and multiplexes multipart streams.

* *navigationtest:* Tests navigation events by drawing a black square that follows the mouse pointer.

* *ogg:* Demultiplexes, multiplexes and parses OGG streams.

* *oss4:* Inputs and outputs audio using the legacy OSS v4 audio system.

* *ossaudio:* Inputs and outputs audio using the legacy OSS audio system.

* *playback:* Collects various playback elements.

* *png:* Decodes and encodes PNG images.

* *pulseaudio:* Inputs and outputs audio using the pulseaudio daemon.

* *replaygain:* Analyses, compresses and volume adjusts an audio stream using ReplayGain.

* *rtp:* Knows how to handle RTP streams. Contains a very large number of elements for specific audio and video sub-streams and other functions.

* *rtpmanager:* Handles RTP session management.

* *rtsp:* Handles encrypted RTP streams.

* *shapewipe:* Adds a "shape wipe" transition effect to a video stream.

* *shout2send:* Sends data to an icecast server using libshout2.

* *smpte:* Applies SMPTE video transitions.

* *soup:* Handles HTTP streams.

* *spectrum:* Runs a Fast Fourier Transformation on an audio stream and outputs spectrum data.

* *speex:* Decodes and encodes audio in the Speex format.

* *staticelements:* Couldn't find out what this one does.

* *subparse:* Parses subtitles in SRT, SUB, MPSUB, MDVD, SMI, TXT and DKS formats.

* *taglib:* Writes APEv2 or ID3v2 tags using TagLib.

* *tcp:* Handles raw TCP/IP streams.

* *theora:* Encodes, parses and decodes streams in the Theora video format.

* *typefindfunctions:* Identifies stream types.

* *udp:* Handles raw UDP/IP streams.

* *video4linux2:* Handles Video4Linux2 tuners and input devices.

* *videobox:* Resizes a video stream by adding borders or cropping.

* *videoconvert:* Converts video streams from one colorspace to another.

* *videocrop:* Crops video streams.

* *videofilter:* Offers different video stream filters, currently: Gamma correction, brightness/contrast/hue/saturation, flipping and rotation, median filter.

* *videomixer:* Mixes several video streams into a combined one.

* *videoparsersbad:* Contains parsers for Dirac, H.263, H.264, MPEG-4, MPEG, PNG and VC1 stream formats.

* *videorate:* Adjusts the rate of a video stream.

* *videoscale:* Scales a video stream.

* *videotestsrc:* Creates an user-defined video test stream.

* *volume:* Changes the volume of the audio data.

* *vorbis:* Encodes, parses and decodes OGG Vorbis audio streams.

* *vpx:* Encodes and decodes On2 VP8/VP9 video streams.

* *wavenc:* Multiplexes WAV audio streams.

* *wavpack:* Encodes and decodes WavPack audio streams.

* *wavparse:* Parses WAV audio streams.

* *ximagesrc:* Creates a screenshot video stream from an X server.

* *y4menc:* Encodes video streams in the YUV4MPEG format.



## Codecs

This is the complete list of currently supported audio, image and video codecs on the bq:

{% highlight text %}
audio/ac3;
audio/mpeg, mpegversion=(int)1;
audio/mpeg, mpegversion=(int)1, layer=(int)[ 1, 3 ];
audio/mpeg, mpegversion=(int)2;
audio/mpeg, mpegversion=(int){ 2, 4 };
audio/mpeg, mpegversion=(int)4, stream-format=(string){ raw, adts };
audio/ogg;
audio/webm;
audio/x-ac3;
audio/x-alaw;
audio/x-amr-nb-sh;
audio/x-amr-wb-sh;
audio/x-au;
audio/x-dts;
audio/x-eac3;
audio/x-flac;
audio/x-flac, framed=(boolean)true;
audio/x-m4a;
audio/x-matroska;
audio/x-mulaw;
audio/x-private1-ac3;
audio/x-private1-dts;
audio/x-sbc;
audio/x-speex;
audio/x-vorbis;
audio/x-wav;
audio/x-wavpack;
audio/x-wavpack, framed=(boolean)true;
image/bmp;
image/gif;
image/jpeg;
image/png;
image/svg;
image/svg+xml;
image/tiff;
image/vnd.wap.wbmp;
image/x-bitmap;
image/x-bmp;
image/x-cmu-raster;
image/x-icon;
image/x-MS-bmp;
image/x-pcx;
image/x-pixmap;
image/x-portable-anymap;
image/x-portable-bitmap;
image/x-portable-graymap;
image/x-portable-pixmap;
image/x-sun-raster;
image/x-tga;
video/mj2;
video/mpeg, mpegversion=(int)[ 1, 2 ], systemstream=(boolean)false;
video/mpeg, mpegversion=(int)4, systemstream=(boolean)false;
video/ogg;
video/quicktime;
video/webm;
video/x-dirac;
video/x-divx, divxversion=(int)[ 3, 5 ];
video/x-divx, divxversion=(int)[ 4, 5 ];
video/x-dv, systemstream=(boolean)false;
video/x-dv, systemstream=(boolean)true;
video/x-fli;
video/x-flv;
video/x-h263, variant=(string)itu;
video/x-h264;
video/x-h264, stream-format=(string)byte-stream, alignment=(string)au;
video/x-matroska;
video/x-matroska-3d;
video/x-msvideo;
video/x-theora;
video/x-vp8;
video/x-vp9;
video/x-wmv, wmvversion=(int)3, format=(string){ WVC1, WMV3 };
{% endhighlight %}
