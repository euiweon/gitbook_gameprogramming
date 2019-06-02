### cocos2d-x 샘플 코드 모아두기 

#### UI::ImageView  사용. ( 무한루핑되는 애니메이션도 추가)
```cpp
auto pTouch = ui::ImageView::create("touchthescreen.png");
pTouch->setAnchorPoint(Vec2(0, 0));
pTouch->setPosition(Vec2(176, 409));
pTouch->setOpacity(0);
pTouch->runAction(RepeatForever::create(
Sequence::create(FadeIn::create(0.3f), DelayTime::create(1.2f), FadeOut::create(0.3f), DelayTime::create(0.2f), nullptr)));
GetBaseNode()->addChild(pTouch);
```
#### ui::Button 사용법 (터치 이벤트는 별도로... )
``` cpp
auto pCloseButton = ui::Button::create("pop1.png");
pCloseButton->setPosition(Vec2(m_pPopupNode->getContentSize().width, m_pPopupNode->getContentSize().height));
pCloseButton->setAnchorPoint(Vec2(0.5f, 0.5f));
pPopup->addChild(pCloseButton);
pCloseButton->setTag(11000);
pCloseButton->addTouchEventListener(funcButtonEvent);
```

#### 해상도 문제 해결법
``` cpp

// no Border로 해상도 설정
Director::getInstance()->getOpenGLView()->setDesignResolutionSize(CLIENT_WIDTH, CLIENT_HEIGHT , ResolutionPolicy::NO_BORDER);

// Scene에 들어갈 Node만 비율에 맞춰 스케일링 및 리포지셔닝
Node* ResizeBorderNode()
{
	auto director = Director::getInstance();
	auto glview = director->getOpenGLView();
	cocos2d::Rect rect = glview->getVisibleRect();
	auto pBaseNode = ui::Layout::create();
	((ui::Layout*)pBaseNode)->setClippingEnabled(false);
	((ui::Layout*)pBaseNode)->setAnchorPoint(Vec2(0.5f, 0.5f));
	((ui::Layout*)pBaseNode)->setTouchEnabled(false);
	pBaseNode->setContentSize(Size(CLIENT_WIDTH, CLIENT_HEIGHT));
	pBaseNode->setTag(123456);
	if ((CLIENT_HEIGHT /*+ APPMANAGER().GetADViewHeight()*/) >= rect.size.height - 3 && (CLIENT_HEIGHT /*+ APPMANAGER().GetADViewHeight()*/) <= rect.size.height + 3)
	{
		((ui::Layout*)pBaseNode)->setScale(rect.size.width / CLIENT_WIDTH);
	}
	else
	{
		((ui::Layout*)pBaseNode)->setScale(rect.size.height / (CLIENT_HEIGHT /*+ APPMANAGER().GetADViewHeight()*/));
	}
	((ui::Layout*)pBaseNode)->setPosition(Vec2(CLIENT_WIDTH / 2, CLIENT_HEIGHT / 2));
	return pBaseNode;
}
``` 

#### rand()를 그냥 써서 랜덤한 값이 안나올경우 써야되는 함수...
``` cpp
// 

inline float GetRandomMinMaxF(float min, float max) 
{
	float randNum = static_cast<float>(rand()) / RAND_MAX;
	return min + ((max - min) * randNum);
}

inline int GetRandomMinMaxI(int min, int max) 
{
	return static_cast<int>(GetRandomMinMaxF(static_cast<float>(min),
static_cast<float>(max)));
}
``` 

#### C++  시간 관련 처리 
``` cpp
inline unsigned int GetNowTimeSec()
{
	auto now = std::chrono::system_clock::now();
	auto duration = now.time_since_epoch();
	uint64_t nowsec = std::chrono::duration_cast<std::chrono::seconds> (duration).count();
	return nowsec;
}

inline uint64_t GetNowTimeMillSec()
{
	auto now = std::chrono::system_clock::now();
	auto duration = now.time_since_epoch();
	uint64_t nowMillsec = std::chrono::duration_cast<std::chrono::milliseconds> (duration).count();
	return nowMillsec;
}
``` 

#### URL 이미지 파일 이름 찾기 
``` cpp
inline void ImageUrlToFilename(const std::string strProfilePhotoURL, char *out, const int& out_size)
{
	char buffer[512];

	snprintf(buffer, sizeof(buffer), "%s", strProfilePhotoURL.c_str());

	for (int i = strProfilePhotoURL.length(); i > 0; --i)
	{
		if (strProfilePhotoURL[i] == '/') {

			snprintf(out, out_size, "%s", strProfilePhotoURL.substr(i + 1).c_str());
			break;

		}
	}
}
``` 

문자열 만들기 
``` cpp
inline const char* _format(const char* format, ...)
{
	char buf[2048];
	buf[0] = 0;
	va_list params;
	va_start(params, format);
	vsprintf(buf, format, params);
	return CCString::create(buf)->getCString();
}
``` 

#### Spine
``` cpp
// 스파인 로드 
auto pBackStage = spine::SkeletonAnimation::createWithFile("apxsoft/spine/down_light_back.json", "apxsoft/spine/down_light_back.atlas");

// 애니메이션 플레이
pBackStage->setAnimation(0, "down02", true);
pBackStage->setAnchorPoint(Vec2(0.5f, 0.0f));
pBackStage->setPosition(Vec2(204 + 110, 231));
m_pCenterNode->addChild(pBackStage, 9);

// 애니메이션 변경 
m_pCenterImage->setAnimation(0, "touch03", false);

// 애니메이션 종료후 처리 
m_pCenterImage->setCompleteListener([&](int trackIndex, int loopCount) {
	ResetCenterAnimation();
});

``` 

#### 확장 가능한 스크롤 뷰 ( 미완성 )
```cpp
class ExpandableCustomListView : public ui::ScrollView //@ MyScrollView는 그냥 사용한 것... 적합한 이름으로 바꿀 것
{
public:
    ExpandableCustomListView() : m_aMargin(nullptr), m_aItem(nullptr), m_nIndex(-1), m_nCount(0) { }
    virtual ~ExpandableCustomListView()
    {
        CC_SAFE_DELETE_ARRAY(m_aMargin);
        CC_SAFE_DELETE_ARRAY(m_aItem);
    }

    static ExpandableCustomListView * create()
    {
        ExpandableCustomListView * widget = new (std::nothrow)ExpandableCustomListView();
        if (widget && widget->init())
        {
            widget->autorelease();
            return widget;
        }
        CC_SAFE_DELETE(widget);
        return nullptr;
    }

    void setItemCount(int count)
    {
        m_nCount = count;
        m_aMargin = new float[count];
        m_aItem = new Node *[count];
        memset(m_aMargin, 0, sizeof(float) * count);
        memset(m_aItem, 0, sizeof(Node *) * count);
    }

    void setItem(int index, Node * item, float margin)
    {
        m_aItem[index] = item;

        if (index - 1 == m_nCount)
            m_aMargin[index] = 0.f;
        else m_aMargin[index] = margin;
    }

    Node * getItemShow()
    {
        if (m_nIndex >= 0)
            return m_aItem[m_nIndex];
        return nullptr;
    }

    void setItemShow(int index)
    {
        if (index < 0)
        {
            scrollVetical();
            m_nIndex = index;
        }
        else
        {
            m_nIndex = index;
            scrollVetical();
        }
    }

    void scrollVetical()
    {
        float rows = 0.f, posY;
        for (int i = 0; i < m_nCount; i++)
        {
            rows += m_aItem[i] ? m_aItem[i]->getContentSize().height : 0.f;
            rows += m_aMargin[i];
        }

        setInnerContainerSize(Size(getContentSize().width, rows));
        posY = rows - 1.f;

        for (int i = 0; i < m_nCount; i++)
        {
            if (m_aItem[i])
            {
                m_aItem[i]->setPositionY(posY);
                posY -= m_aItem[i]->getContentSize().height;
            }
            posY -= m_aMargin[i];
        }

        // 해당 아이템이 범위에 들어와 있는지를 계산해주는 부분입니다.
        if (m_nIndex >= 0 && m_nIndex < m_nCount && m_aItem[m_nIndex])
        {
            float innerHeight = getInnerContainerSize().height;
            float wholeHeight = innerHeight - getContentSize().height;

            if (wholeHeight <= 0.f) return; // 아이템이 스크롤창을 넘치지 않으면 그대로 종료합니다.

            float innerSY = wholeHeight + getInnerContainerPosition().y;
            float itemSY = innerHeight - m_aItem[m_nIndex]->getPositionY();

            startRecordSlidAction();

            if (innerSY > itemSY)
            {	// 아이템의 상단부분 100분율 계산
                scrollToPercentVertical(itemSY / wholeHeight * 100.f, 0.333f, true);
            }

            float innerEY = -getInnerContainerPosition().y;
            float itemEY = m_aItem[m_nIndex]->getPositionY() - m_aItem[m_nIndex]->getContentSize().height;

            if (innerEY > itemEY)
            {	// 아이템의 하단부분 100분율 계산
                scrollToPercentVertical(100.f - itemEY / wholeHeight * 100.f, 0.333f, true);
            }

            //			CCLOG("wholeHeight = %f, innerSY = %f, itemSY = %f, innerEY = %f, itemEY = %f, itempos=%f, itemsize=%f", wholeHeight, innerSY, itemSY, innerEY, itemEY, m_aItem[m_nIndex]->getPositionY(), m_aItem[m_nIndex]->getContentSize().height);
        }
        //		else
        //		{
        //			startRecordSlidAction();
        //			endRecordSlidAction();
        //		}
    }

    int m_nIndex;
    int m_nCount;
    float * m_aMargin;
    Node ** m_aItem;
};

#define SCROLL_ROW		102
#define SCROLL_ADD		83
#define SCROLL_GAP		5
```

#### // 웹 API 연결
```cpp
#pragma once


#if (CC_TARGET_PLATFORM == CC_PLATFORM_WIN32)
#include <windows.h>
#endif 

namespace Test
{
	// h
    class NetworkManager : public Test::Singleton<NetworkManager>
    {
    public:
        NetworkManager();
        ~NetworkManager();

        void init();

        void SendHttpPacketCore(std::string strPath, std::string strData, network::HttpRequest::Type eType,
            std::function<void(cocos2d::network::HttpClient* client, cocos2d::network::HttpResponse* response)> funcCallback, bool encrypt = false);


        void SendHttpPacket(std::string strPath, std::string strData, network::HttpRequest::Type eType,
            std::function<void(int, std::string)> callback, bool encrypt);
        // get의 경우는 mapdata를 주소로 변환 하고 나머지는 json으로 변환(자식구조는 불가능)
        void SendHttpPacket(std::string strPath, std::map<std::string, std::string> mapData, network::HttpRequest::Type eType,
            std::function<void(int, std::string)> callback, bool encrypt = true);

        /////////////////////////////////////////////////////////////////
        // Facebook Rest API 연결 함수 
        // 기존 Rest API코드와 동일하지만, 약간의 수정이 있어서 별도로 분리 해놓음. 
        // 나중에 합치기를... 
        void SendHttpPacketFB(std::string strAddpath, std::map<std::string, std::string> mapData, network::HttpRequest::Type eType,
            std::function<void(int, std::string)> callback);

        void SendHttpPacketFB(std::string strPath, std::string strData, network::HttpRequest::Type eType,
            std::function<void(int, std::string)> callback);

        void SendHttpPacketCoreFB(std::string strPath, std::string strData, network::HttpRequest::Type eType,
            std::function<void(cocos2d::network::HttpClient* client, cocos2d::network::HttpResponse* response)> funcCallback);
    private:
        void update(float dt);
    protected:
        A_SYNTHESIZE(std::string, LastErrorMessage);
        A_SYNTHESIZE(bool, Disconnect);
        A_SYNTHESIZE(int64_t, NetworkUserID);
    };

	// cpp 
	NetworkManager::NetworkManager()
    {

    }


    NetworkManager::~NetworkManager()
    {

    }


    void    NetworkManager::init()
    {
#if (CC_TARGET_PLATFORM != CC_PLATFORM_WINRT)
        network::HttpClient::getInstance()->setTimeoutForConnect(15);
#endif 
        m_LastErrorMessage = "";
        m_Disconnect = false;
        m_NetworkUserID = -1;
    }


    void NetworkManager::update(float dt)
    {
    }

    void  NetworkManager::SendHttpPacketCore(std::string strPath, std::string strData, network::HttpRequest::Type eType,
        std::function<void(cocos2d::network::HttpClient* client, cocos2d::network::HttpResponse* response)> funcCallback, bool encrypt)
    {

        
        network::HttpRequest* request = new network::HttpRequest();
        
        if (m_NetworkUserID == -1)
        {
            auto pResponse = new cocos2d::network::HttpResponse(request);
            pResponse->setResponseCode(404);
            if (encrypt)
            {
                std::string strSendData = "not found networkID";
                strSendData = encrypt_server_base64(strData.c_str());
                pResponse->setResponseDataString(strSendData.c_str(), strlen(strSendData.c_str()));
            }
            else
            {
                pResponse->setResponseDataString("not found networkID", strlen("not found networkID"));
            }
            
            funcCallback(network::HttpClient::getInstance(), pResponse);
            request->release();
            return;
        }
        else if (m_Disconnect)
        {
            auto pResponse = new cocos2d::network::HttpResponse(request);
            pResponse->setResponseCode(404);
            if (encrypt)
            {
                std::string strSendData = "Not Connect";
                strSendData = encrypt_server_base64(strData.c_str());
                pResponse->setResponseDataString(strSendData.c_str(), strlen(strSendData.c_str()));
            }
            else
            {
                pResponse->setResponseDataString("Not Connect", strlen("Not Connect"));
            }
           
            funcCallback(network::HttpClient::getInstance(), pResponse);
            request->release();
            return;
        }


        request->setUrl(strPath.c_str());
        request->setRequestType(eType);
        request->setResponseCallback(funcCallback);

        std::vector<std::string> headers;


        headers.push_back("content-type: application/json");

        if (eType != network::HttpRequest::Type::GET)
        {
            request->setRequestData(strData.c_str(), strData.length());
        }

        request->setHeaders(headers);

        network::HttpClient::getInstance()->send(request);
        request->release();
    }


    void  NetworkManager::SendHttpPacket(std::string strPath, std::string strData, network::HttpRequest::Type eType,
        std::function<void(int, std::string)> callback, bool encrypt)
    {
        
        std::string strSendData = strData;
        if (encrypt)
            strSendData = encrypt_server_base64(strData.c_str());

        SendHttpPacketCore(strPath, strSendData, eType, [&, strPath, eType, strData, callback, encrypt]
                                (cocos2d::network::HttpClient* client, cocos2d::network::HttpResponse* pResponse)
        {

            std::string strResponseEncryptData = std::string(pResponse->getResponseData()->begin(), pResponse->getResponseData()->end());

            std::string strResponseData = "";

            if (encrypt)
                strResponseData = (char *)decrypt_server_base64(strResponseEncryptData.c_str(), (int)strResponseEncryptData.length());
            else
                strResponseData = strResponseEncryptData;
            
           
            if (pResponse->getResponseCode() / 100 >= 4)
            {
                SetDisconnect(true);
                m_NetworkUserID = -1; 
            }



           
            

            // Header Test
            {
                std::string strHeader(pResponse->getResponseHeader()->begin(), pResponse->getResponseHeader()->end());
                std::unordered_map<std::string, std::string>  mapHeader;
                std::string strHeader1 = strHeader;
                int     nCutPos;
                int     nIndex = 0;
                std::vector<std::string> headers; 
                while ((nCutPos = strHeader1.find_first_of("\r\n")) != strHeader1.npos)
                {
                    if (nCutPos > 0)
                    {
                        headers.push_back(strHeader1.substr(0, nCutPos));
                    }
                    strHeader1 = strHeader1.substr(nCutPos + 1);
                }
                for (auto header : headers)
                {
                    std::string key = header.substr(0, header.find_first_of(':'));
                    std::string value = header.substr(header.find_first_of(':') + 1, header.size() - 1);

                    if (mapHeader.find(key) == mapHeader.end())
                        mapHeader.insert(std::make_pair(key, value));
                }
            }

            switch (eType)
            {
            case cocos2d::network::HttpRequest::Type::GET:
                CCLOG("ResultCode : %d\nURL : %s\nType : GET\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::POST:
                CCLOG("ResultCode : %d\nURL : %s\nType : POST\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::PUT:
                CCLOG("ResultCode : %d\nURL : %s\nData : %s\nType : PUT\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(),
                    strData.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::DELETE:
                CCLOG("ResultCode : %d\nURL : %s\nType : DELETE\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::UNKNOWN:
                CCLOG("ResultCode : %d\nURL : %s\nType : UNKNOWN\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            default:
                break;
            }

            callback(pResponse->getResponseCode(), strResponseData);
        }, encrypt);
    }


    void NetworkManager::SendHttpPacket(std::string strPath, std::map<std::string, std::string> mapData, network::HttpRequest::Type eType,
        std::function<void(int, std::string)> callback, bool encrypt)
    {
        if (eType == cocos2d::network::HttpRequest::Type::GET)
        {

            bool bFirst = true;
            std::string outData;
            for (auto& ref : mapData)
            {
                if (bFirst)
                {
                    bFirst = false;
                    outData += "?";
                }
                else
                {
                    outData += "&";
                }
                outData += ref.first + "=" + ref.second;
            }

            SendHttpPacket(strPath + outData, "", eType, callback, encrypt);

        }
        else
        {

            Json::Value root;
            Json::Value encoding;
            for (auto& ref : mapData)
            {
                root[ref.first] = ref.second;
            }

            Json::StyledWriter writer;
            std::string outputData = writer.write(root);

            SendHttpPacket(strPath, outputData.c_str(), eType, callback, encrypt);

            CCLOG("Send : %s", outputData.c_str());

        }
    }

    /////////////////////////////////////////////////////////////////

    void NetworkManager::SendHttpPacketFB(std::string strAddpath, std::map<std::string, std::string> mapData, network::HttpRequest::Type eType,
        std::function<void(int, std::string)> callback)
    {

        /////////////////////////////////////////////////////////////////
        // FB 연결 관련 내용... 
        std::string strPath = "https://graph.facebook.com/v2.8" + strAddpath;

        if (!APPMANAGER().IsFacebookLogin())
        {
            callback(1000, "Not Login or Token expired");
            return;
        }
        std::string strUserToken = "";
        strUserToken = PluginXManager::GetSingleton().sendPluginFunctionWrapString("PluginFB", "GetAccessToken");
        mapData.insert(std::make_pair("access_token", strUserToken.c_str()));
        /////////////////////////////////////////////////////////////////

        if (eType == cocos2d::network::HttpRequest::Type::GET ||
            eType == cocos2d::network::HttpRequest::Type::DELETE)
        {

            bool bFirst = true;
            std::string outData;
            for (auto& ref : mapData)
            {
                if (bFirst)
                {
                    bFirst = false;
                    outData += "?";
                }
                else
                {
                    outData += "&";
                }
                outData += ref.first + "=" + ref.second;
            }

            SendHttpPacketFB(strPath + outData, "", eType, callback);

        }
        else
        {

            Json::Value root;
            Json::Value encoding;
            for (auto& ref : mapData)
            {
                root[ref.first] = ref.second;
            }

            Json::StyledWriter writer;
            std::string outputData = writer.write(root);

            SendHttpPacketFB(strPath, outputData.c_str(), eType, callback);

            CCLOG("%s", outputData.c_str());

        }
    }



    void  NetworkManager::SendHttpPacketFB(std::string strPath, std::string strData, network::HttpRequest::Type eType,
        std::function<void(int, std::string)> callback)
    {
        SendHttpPacketCoreFB(strPath, strData, eType, [&, strPath, eType, strData, callback](cocos2d::network::HttpClient* client, cocos2d::network::HttpResponse* pResponse)
        {
            std::string strResponseData = std::string(pResponse->getResponseData()->begin(), pResponse->getResponseData()->end());


            if (pResponse->getResponseCode() / 100 >= 4)
            {
                SetDisconnect(true);
            }

            switch (eType)
            {
            case cocos2d::network::HttpRequest::Type::GET:
                CCLOG("ResultCode : %d\nURL : %s\nType : GET\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::POST:
                CCLOG("ResultCode : %d\nURL : %s\nType : POST\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::PUT:
                CCLOG("ResultCode : %d\nURL : %s\nData : %s\nType : PUT\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(),
                    strData.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::DELETE:
                CCLOG("ResultCode : %d\nURL : %s\nType : DELETE\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            case cocos2d::network::HttpRequest::Type::UNKNOWN:
                CCLOG("ResultCode : %d\nURL : %s\nType : UNKNOWN\nResult : %s", (int)pResponse->getResponseCode(), strPath.c_str(), strResponseData.c_str());
                break;
            default:
                break;
            }

            callback(pResponse->getResponseCode(), strResponseData);
        });
    }


    void  NetworkManager::SendHttpPacketCoreFB(std::string strPath, std::string strData, network::HttpRequest::Type eType,
        std::function<void(cocos2d::network::HttpClient* client, cocos2d::network::HttpResponse* response)> funcCallback)
    {
        network::HttpRequest* request = new network::HttpRequest();

        if (m_Disconnect)
        {
            auto pResponse = new cocos2d::network::HttpResponse(request);
            pResponse->setResponseCode(404);
            pResponse->setResponseDataString("Not Connect", strlen("Not Connect"));
            funcCallback(network::HttpClient::getInstance(), pResponse);
            request->release();
            return;
        }

        request->setUrl(strPath.c_str());
        request->setRequestType(eType);
        request->setResponseCallback(funcCallback);

        std::vector<std::string> headers;

        //headers.push_back("Content-Type: application/json");
        //headers.push_back("charset=utf-8");


        if (eType != network::HttpRequest::Type::GET)
        {
            request->setRequestData(strData.c_str(), strData.length());
        }

        request->setHeaders(headers);


        network::HttpClient::getInstance()->send(request);
        request->release();
    }

}
```
