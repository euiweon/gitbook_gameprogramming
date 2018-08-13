# Cocos2D-x 안드로이드에서 ogg/vorbis 관련 라이브러리 교체하기

왜 이런짓을 해야 하는건가?

* android에서 audioengine의 ogg부분이 tremolo라는 decorder만 탑재되어 있음
* 내부에서 인코딩이 필요할 경우, 다른 라이브러리가 필요함.
* 다른 라이브러리를 추가 할 경우에 ov\_read같은 함수가 잘못 동작해 에러가 남. 
* 그래서 기존 라이브러리 링크를 지우고 새로운 코드를 추가 시켜줘야 함. 

이렇게 함으로써 얻을수 있는 장점은?

* 인코딩 기능도 동작하고 에러 처리도 일어나지 않는다. 

방법

* cocos2d/cocos/audio/ 에 ogg 폴더 만들어서 관련 라이브러리 복사해서 넣어주자
* cocos2d/cocos/audio/android/android.mk 파일도 ogg 파일 빌드 하도록 경로 넣어주고, 기존에 링크 되었던 라이브러리는 삭제 해준다

```text
LOCAL_PATH := $(call my-dir)

#New AudioEngine
include $(CLEAR_VARS)

LOCAL_MODULE := audioengine_static

LOCAL_MODULE_FILENAME := libaudioengine

OGG_SOURCES := $(wildcard $(LOCAL_PATH)/../ogg/libogg/src/*.c)
VORBIS_SOURCES := $(wildcard $(LOCAL_PATH)/../ogg/libvorbis/src/*.c)

LOCAL_SRC_FILES :=  $(OGG_SOURCES:$(LOCAL_PATH)/%=%) \
                   $(VORBIS_SOURCES:$(LOCAL_PATH)/%=%) \
                    AudioEngine-inl.cpp \
                   ../AudioEngine.cpp \
                   CCThreadPool.cpp \
                   AssetFd.cpp \
                   AudioDecoder.cpp \
                   AudioDecoderProvider.cpp \
                   AudioDecoderSLES.cpp \
                   AudioDecoderOgg.cpp \
                   AudioDecoderMp3.cpp \
                   AudioDecoderWav.cpp \
                   AudioPlayerProvider.cpp \
                   AudioResampler.cpp \
                   AudioResamplerCubic.cpp \
                   PcmBufferProvider.cpp \
                   PcmAudioPlayer.cpp \
                   UrlAudioPlayer.cpp \
                   PcmData.cpp \
                   AudioMixerController.cpp \
                   AudioMixer.cpp \
                   PcmAudioService.cpp \
                   Track.cpp \
                   audio_utils/format.c \
                   audio_utils/minifloat.cpp \
                   audio_utils/primitives.c \
                   utils/Utils.cpp \
                   mp3reader.cpp \
                   tinysndfile.cpp


LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/../include

LOCAL_EXPORT_LDLIBS := -lOpenSLES

LOCAL_C_INCLUDES := $(LOCAL_PATH)/../include \
                    $(LOCAL_PATH)/../.. \
                    $(LOCAL_PATH)/../../platform/android \
                    $(LOCAL_PATH)/../ogg/libogg/include \
                    $(LOCAL_PATH)/../ogg/libvorbis/include \
                    $(LOCAL_PATH)/../../../external/android-specific

LOCAL_STATIC_LIBRARIES += libpvmp3dec
include $(BUILD_STATIC_LIBRARY)

#SimpleAudioEngine
include $(CLEAR_VARS)

LOCAL_MODULE := cocosdenshion_static

LOCAL_MODULE_FILENAME := libcocosdenshion

LOCAL_SRC_FILES := cddSimpleAudioEngine.cpp \
                   ccdandroidUtils.cpp \
                   jni/cddandroidAndroidJavaEngine.cpp

LOCAL_STATIC_LIBRARIES := audioengine_static
LOCAL_EXPORT_C_INCLUDES := $(LOCAL_PATH)/../include

LOCAL_C_INCLUDES := $(LOCAL_PATH)/../include \
                    $(LOCAL_PATH)/../.. \
                    $(LOCAL_PATH)/../../platform/android

include $(BUILD_STATIC_LIBRARY)

#$(call import-module,android-specific/tremolo)
$(call import-module,android-specific/pvmp3dec)
```

* cocos2d/cocos/audio/Android/AudioDecoderOGG.h 헤더 링크를 수정해주고, AudioDecoderOGG.cpp의 ov\_read 인자값을 적절하게 수정해주자. 

```cpp
#pragma once

#include "audio/android/AudioDecoder.h"

#include "vorbis/vorbisfile.h"
#include "vorbis/codec.h"

namespace cocos2d { namespace experimental {

class AudioDecoderOgg : public AudioDecoder
{
protected:
    AudioDecoderOgg();
    virtual ~AudioDecoderOgg();

    static int fseek64Wrap(void* datasource, ogg_int64_t off, int whence);
    virtual bool decodeToPcm() override;

    friend class AudioDecoderProvider;
};

}} // namespace cocos2d { namespace experimental {
```

```cpp
bool AudioDecoderOgg::decodeToPcm()
{
    _fileData = FileUtils::getInstance()->getDataFromFile(_url);
    if (_fileData.isNull())
    {
        return false;
    }

    ov_callbacks callbacks;
    callbacks.read_func = AudioDecoder::fileRead;
    callbacks.seek_func = AudioDecoderOgg::fseek64Wrap;
    callbacks.close_func = AudioDecoder::fileClose;
    callbacks.tell_func = AudioDecoder::fileTell;

    _fileCurrPos = 0;

    OggVorbis_File vf;
    int ret = ov_open_callbacks(this, &vf, NULL, 0, callbacks);
    if (ret != 0)
    {
        ALOGE("Open file error, file: %s, ov_open_callbacks return %d", _url.c_str(), ret);
        return false;
    }
    // header
    auto vi = ov_info(&vf, -1);

    uint32_t pcmSamples = (uint32_t) ov_pcm_total(&vf, -1);

    uint32_t bufferSize = pcmSamples * vi->channels * sizeof(short);
    char* pcmBuffer = (char*)malloc(bufferSize);
    memset(pcmBuffer, 0, bufferSize);

    int currentSection = 0;
    long curPos = 0;
    long readBytes = 0;

    do
    {
        //readBytes = ov_read(&vf, pcmBuffer + curPos, 4096, &currentSection);
        readBytes = ov_read(&vf, pcmBuffer + curPos, 4096, 0, 2, 1, &currentSection);
        curPos += readBytes;
    } while (readBytes > 0);

    if (curPos > 0)
    {
        _result.pcmBuffer->insert(_result.pcmBuffer->end(), pcmBuffer, pcmBuffer + bufferSize);
        _result.numChannels = vi->channels;
        _result.sampleRate = vi->rate;
        _result.bitsPerSample = SL_PCMSAMPLEFORMAT_FIXED_16;
        _result.containerSize = SL_PCMSAMPLEFORMAT_FIXED_16;
        _result.channelMask = vi->channels == 1 ? SL_SPEAKER_FRONT_CENTER : (SL_SPEAKER_FRONT_LEFT | SL_SPEAKER_FRONT_RIGHT);
        _result.endianness = SL_BYTEORDER_LITTLEENDIAN;
        _result.numFrames = pcmSamples;
        _result.duration = 1.0f * pcmSamples / vi->rate;
    }
    else
    {
        ALOGE("ov_read returns 0 byte!");
    }

    ov_clear(&vf);
    free(pcmBuffer);

    return (curPos > 0);
}
```

