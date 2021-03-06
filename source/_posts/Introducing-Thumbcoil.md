---
title: Introducing Thumbcoil
tags:
  - thumbcoil
  - mse
  - hls
author:
  name: Jon-Carlos Rivera
  github: imbcmdth

date: 2016-10-19 16:47:37
---


{% pullquote right %}
A **transmuxer** takes media contained in some file format, extracts the raw compressed video and audio from inside (a process called demuxing) and repackages the compressed data  into another format (termed remuxing) without performing any re-compression.
{% endpullquote %}

## In the Beginning

While building [Mux.js](https://github.com/videojs/mux.js) - the transmuxer at the heart of [videojs-contrib-hls](https://github.com/videojs/videojs-contrib-hls) - we faced a problem: How do we determine if the output from Mux.js is correct?

Early on we managed to figure out how to coax FFmpeg into creating [MP4s](https://en.wikipedia.org/wiki/MPEG-4_Part_14) from [MPEG2-TS](https://en.wikipedia.org/wiki/MPEG_transport_stream) segments that would play back in a browser with [Media Source Extensions](https://en.wikipedia.org/wiki/Media_Source_Extensions) (MSE) which at the time meant only Chrome. However, we needed a simple way to compare the output of our transmuxer with what was produced by FFmpeg. The comparison had to be aware of the MP4 format since the two outputs are extremely unlikely to be byte-identical.

{% pullquote left %}
MP4 files are composed of **boxes** - hierarchical logical units that, conveniently, all start with a 32-bit length and a 32-bit box-type. Boxes will often contain other sub-boxes.
{% endpullquote %}

The answer to that problem was to build an "mp4-inspector" - a tool that would parse MP4 and display a sort of JSON-like dump of any relevant boxes and their contents. By generating a dump of the output from Mux.js and comparing it to a known-good fragment generated with FFmpeg, we could see where our transmuxer's output differed.

The "mp4-inspector" was built as a web page so that we can have a graphical color-coded diff of the two segments. Over time the page gained a video element and we started appending the results of transmuxing segments directly into the video element's MediaSource to aid in instant feedback and validation of changes to Mux.js.

## A Brave New World
{% pullquote right %}
A **media container** such as MP4 encapsulates the video and audio stream. It has metadata describing the streams, timing information for each frame, and the stream data itself.
{% endpullquote %}

As development continued, we would sometimes encounter streams that would fail in new and interesting ways. Some of these failures were, admittedly, due to bugs in Mux.js. As Mux.js itself became more robust, failures were increasingly caused by problems with the streams or issues with a particular implementation of the [MSE specification](http://www.w3.org/TR/2014/CR-media-source-20140717/).

It eventually dawned on us that we really needed to learn more about what was happening inside of those videos. We needed to see not just what was happening at the media container level but we had to go deeper - we needed to peek into the video data itself. For that purpose we created [Thumbcoil](http://thumb.co.il).

{% pullquote left %}
Inside of a container, video and audio are contained in data called **bitstreams**. Bitstreams are the data produced by encoders to represent the audio signals or video frames. Some common bitstreams are [AAC](https://en.wikipedia.org/wiki/Advanced_Audio_Coding) for audio and [H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC) for video.
{% endpullquote %}

## Thumbcoil

[Thumbcoil](http://thumb.co.il) is a suite of tools designed to give you a peek into the internals of H.264 video bitstreams contained inside either an MP4 or MPEG2-TS container file. Using the tools in [Thumbcoil](http://thumb.co.il) you can get a detailed view of the internal structure of the two supported media container formats.

In addition, the tools have the ability to show you the information contained within the most important NAL-units that make up the H.264 bitstream. Ever wonder what kind of secret information the video encoder has squirreled away for decoders to use? Now, with [Thumbcoil](http://thumb.co.il), you can finally see for yourself!

## Motivation
{% pullquote right %}
An H.264 encoded bitstream is composed of what are called **NAL**, or network abstraction layer, units. NALs are a simple packet format designed to use bits as efficiently as possible.
{% endpullquote %}

Believe it or not, there are very few good tools to generate a somewhat graphical display of the structure of media containers and the data that they contain. Debugging problems with video playback is usually a tedious task involving various esoteric *FFmpeg* and *FFprobe* incantations. Unfortunately at it's best, *FFprobe* is only able to print out a small portion of the data we were interested in.

The exact data inside of the various parameter sets for instance is not available via the command-line. Inside of FFprobe, that data is parsed and stored but there is no easy way to dump that information in a human readable form.

{% pullquote left %}
In H.264, there are two special types of NAL-units - the SPS or **seq_parameter_set** and the PPS or **pic_parameter_set**. These two NAL units contain a lot of information. The decoders require this information to reconstruct the video.
{% endpullquote %}

[Thumbcoil](http://thumb.co.il) not only provides parameter set information in excruciating detail but also keeps the information with its surrounding context - the boxes it was contained by or the frame it was specified along with. This context is often very important to understanding issues or peculiarities in streams.

## Built Upon Fancy Stuff

One of the more interesting things about how [Thumbcoil](http://thumb.co.il) parses parameter sets is that is builds what is internally called a "codec" for each NAL unit type. These codecs are specified using what is essentially a fancy [parser combinator](https://en.wikipedia.org/wiki/Parser_combinator)-type setup.

{% pullquote right %}
Much of the data in the two parameter sets are stored using a method called [**exponential-golomb encoding.**](https://en.wikipedia.org/wiki/Exponential-Golomb_coding) This method uses a variable number of bits to store numbers and is particularly suited to values that tends to be small.
{% endpullquote %}

Each function used to build the codec returns an object with two functions: *decode* and *encode*. This means that we can specify the format of, say, a seq_parameter_set NAL unit just once and then we can both parse from and write to the bitstream for that particular NAL unit.

The "grammar" used to specify NAL unit codecs is very similar to the grammar used by the H.264 specification (ISO/IEC 14496-10). The data-types that the codecs in [Thumbcoil](http://thumb.co.il) understand are, with some extensions, merely the same types defined in the specification such as signed- and unsigned- exponential golomb encoded integers.

In addition to the parameter sets, [Thumbcoil](http://thumb.co.il) provides insight into the structure of the slice layers themselves by parsing the slice_header data though we stop short of parsing any of the actual slice_data because things quickly become more difficult and less useful as you descend into that madness.

## But what is the deal with the name?

"[Thumbcoil](http://thumb.co.il)" doesn't mean anything, really. It's an inside joke that is funny to exactly 3 people in the world - myself included. The odd name does have one benefit in that it makes for a short and easy to remember domain-name: [thumb.co.il](http://thumb.co.il).

As with all Video.js projects, [Thumbcoil](http://thumb.co.il) is open-source software and we welcome suggestions, issues, and contributions at [https://github.com/videojs/thumbcoil](https://github.com/videojs/thumbcoil).
