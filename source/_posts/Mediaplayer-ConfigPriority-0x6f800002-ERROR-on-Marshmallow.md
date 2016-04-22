title: Mediaplayer ConfigPriority(0x6f800002)) ERROR on Marshmallow
date: 2016-04-22 15:59:30
tags:
---
#Mediaplayer ConfigPriority(0x6f800002)) ERROR on Marshmallow

Android 6.0 使用源生MediaPlayer播放小于15s的音频文件时,有可能会抛出此错误 ConfigPriority(0x6f800002))
不会crash,但是没有声音

log:

	04-21 16:42:11.787 745-21846/? I/QComExtractorFactory: Sniff: Set key to use qti parser
	04-21 16:42:11.787 745-21846/? I/FFmpegExtractor: android-source:0xee977200
	04-21 16:42:11.790 745-21846/? E/FFmpegExtractor: android-source:0xee977200: avformat_open_input failed, err:Invalid data found when processing input
	04-21 16:42:11.790 745-21846/? W/FFmpegExtractor: sniff through BetterSniffFFMPEG failed, try LegacySniffFFMPEG
	04-21 16:42:11.793 745-21846/? E/FFmpegExtractor: android-source:0xee977200|file:: avformat_open_input failed, err:Invalid data found when processing input
	04-21 16:42:11.793 745-21846/? D/FFmpegExtractor: SniffFFMPEG failed to sniff this source
	04-21 16:42:11.793 745-21846/? I/ExtendedExtractor: QTIParser is prefered
	04-21 16:42:11.793 745-21846/? D/MMParserExtractor: setExtraFlags called with flags 3 for paser instance ee977340
	04-21 16:42:11.793 745-21846/? D/MMParserExtractor: using bigger parser buffers
	04-21 16:42:11.793 745-21846/? I/ExtendedExtractor: ExtendedExtractor::create 0xee977340
	04-21 16:42:11.793 745-21846/? I/ExtendedExtractor: ExtendedExtractor::updateExtractor: Use default QTI parser 
	04-21 16:42:11.793 745-21846/? D/MMParserExtractor: Overiding Parser out buffer size  = 262144 
	04-21 16:42:11.794 745-21845/? D/NuPlayerDriver: notifyListener_l(0xefee20a0), (1, 0, 0)
	04-21 16:42:11.796 745-3258/? D/NuPlayerDriver: start(0xefee20a0), state is 4, eos is 0
	04-21 16:42:11.796 745-21845/? I/GenericSource: start
	04-21 16:42:11.797 745-21845/? I/AudioPolicyManagerCustom: Offload min duration is 15s
	04-21 16:42:11.797 745-21845/? I/AudioPolicyManagerCustom: PCM offload property is enabled
	04-21 16:42:11.797 745-21845/? I/AudioPolicyManagerCustom: Offload min duration is 15s
	04-21 16:42:11.798 745-21845/? I/AudioPolicyManagerCustom: Offload min duration is 15s
	04-21 16:42:11.798 745-21845/? I/AudioPolicyManagerCustom: PCM offload property is enabled
	04-21 16:42:11.798 745-21845/? I/AudioPolicyManagerCustom: Offload min duration is 15s
	04-21 16:42:11.812 745-21850/? E/OMXNodeInstance: setConfig(21f:google.aac.decoder, ConfigPriority(0x6f800002)) ERROR: Undefined(0x80001001)
	04-21 16:42:11.812 745-21850/? I/ACodec: codec does not support config priority (err -2147483648)
	04-21 16:42:11.813 745-21850/? I/MediaCodec: MediaCodec will operate in async mode
	04-21 16:42:11.814 745-21851/? I/SoftAAC2: Reconfiguring decoder: 0->44100 Hz, 0->0 channels
	04-21 16:42:11.815 745-21851/? W/SoftAAC2: aacDecoder_DecodeFrame decoderErr = 0x4007
	04-21 16:42:11.815 745-21851/? W/SoftAAC2: AAC decoder returned error 0x4007, substituting silence
	04-21 16:42:11.816 745-21849/? I/NuPlayerDecoder: [audio] saw output EOS
	04-21 16:42:11.816 745-21845/? I/ExtendedNuPlayer: notifyListener 2 0 0
	04-21 16:42:11.816 745-21845/? D/NuPlayerDriver: notifyListener_l(0xefee20a0), (2, 0, 0)
	04-21 16:42:11.816 745-21848/? W/NuPlayerRenderer: onDrainAudioQueue(): audio sink is not ready


google issue已经确认了此Bug,目前处在assign状态,不过在6.0.1上测试时已经正常.

ref [google issue](https://code.google.com/p/android/issues/detail?id=189051) [需翻墙]

从log和更新记录上看,猜测似乎是offload的影响

	av.offload.enable=true 
    av.streaming.offload.enable=true 
    audio.offload.buffer.size.kb=64 
    audio.offload.min.duration.secs=30 
    audio.offload.gapless.enabled=true 
    audio.offload.pcm.16bit.enable=true 
    audio.offload.pcm.24bit.enable=true

从源码上看 似乎针对不同长度的音频文件 ,根据offload.enable和offload.min.duration,会有不同的处理方式

Audio Offload 音频分载, 具体代码逻辑没有细看,也不大清楚具体错误原因

目前唯一已知解决方案是,降级到5.0或升级到6.0.1+
