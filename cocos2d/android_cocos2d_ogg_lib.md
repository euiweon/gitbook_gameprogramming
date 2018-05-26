#### Cocos2D-x 안드로이드에서 ogg/vorbis 관련 라이브러리 교체하기

왜 이런짓을 해야 하는건가?

- android에서 audioengine의 ogg부분이 decorder만 탑재되어 있음
- 내부에서 인코딩이 필요할 경우, 다른 라이브러리가 필요함.
- 다른 라이브러리를 추가 할 경우에 ov_read같은 함수가 잘못 동작해 에러가 남. 
- 그래서 기존 라이브러리 링크를 지우고 새로운 코드를 추가 시켜줘야 함. 

이렇게 함으로써 얻을수 있는 장점은?

- 인코딩 기능도 동작하고 에러 처리도 일어나지 않는다. 

방법

- cocos2d/cocos/audio/ 에 ogg 폴더 만들어서 관련 라이브러리 복사해서 넣어주자
-  cocos2d/cocos/audio/android/android.mk 파일도 ogg 파일 빌드 하도록 경로 넣어주고, 기존에 링크 되었던 라이브러리는 삭제 해준다
-  cocos2d/cocos/audio/Android/AudioDecoderOGG.h 헤더 링크를 수정해주고, AudioDecoderOGG.cpp의 ov_read 인자값을 적절하게 수정해주자. 




#### 