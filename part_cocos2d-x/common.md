cocos2d-x 샘플 코드 모아두기 

UI::ImageView  사용. ( 무한루핑되는 애니메이션도 추가)
```cpp
auto pTouch = ui::ImageView::create("touchthescreen.png");
pTouch->setAnchorPoint(Vec2(0, 0));
pTouch->setPosition(Vec2(176, 409));
pTouch->setOpacity(0);
pTouch->runAction(RepeatForever::create(
Sequence::create(FadeIn::create(0.3f), DelayTime::create(1.2f), FadeOut::create(0.3f), DelayTime::create(0.2f), nullptr)));
GetBaseNode()->addChild(pTouch);
```
ui::Button 사용법 (터치 이벤트는 별도로... )
``` cpp
auto pCloseButton = ui::Button::create("pop1.png");
pCloseButton->setPosition(Vec2(m_pPopupNode->getContentSize().width, m_pPopupNode->getContentSize().height));
pCloseButton->setAnchorPoint(Vec2(0.5f, 0.5f));
pPopup->addChild(pCloseButton);
pCloseButton->setTag(11000);
pCloseButton->addTouchEventListener(funcButtonEvent);
```

해상도 문제 해결법
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

rand()를 그냥 써서 랜덤한 값이 안나올경우 써야되는 함수...
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

C++  시간 관련 처리 
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

URL 이미지 파일 이름 찾기 
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

확장 가능한 스크롤 뷰 ( 미완성 )
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
