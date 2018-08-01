### 유니티 여러가지 사용법

##### iPad Pro 의 120hz 대응 
- Player Setting 에서 enable Promotion 옵션을 체크
- App 의 타겟 프레임에 60으로 설정되어 있기에 따로 디바이스 이름을 체크해서 해당 모델일 경우만 120프레임으로 올려주는 기능의 필요
- SystemInfo.deviceModel 로 디바이스 이름 체크 
- IOS.DeviceGeneration 를 사용하지 않는 이유는 enum 값으로 해당 디바이스가 정의되어 있어 디바이스가 추가 될 경우에 대한 대응이 힘듬. 

