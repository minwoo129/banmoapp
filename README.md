# 구르미랑 반모 설계내용 정리

# 담당영역

### 마이페이지

- ui 설계
- 리덕스 액션, 액션함수, 리듀서 설계
- ui 이미지

![KakaoTalk_Photo_2021-10-29-23-43-35 010](https://user-images.githubusercontent.com/73742422/139526657-89ba0db2-502b-46a2-a320-ede21724904b.jpeg)

![KakaoTalk_Photo_2021-10-29-23-43-34 001](https://user-images.githubusercontent.com/73742422/139526682-f48be041-9747-4260-bfac-a8852d81f991.jpeg)


![KakaoTalk_Photo_2021-10-29-23-43-35 003](https://user-images.githubusercontent.com/73742422/139526686-4bbc5f6a-fc8a-4be1-a6e6-9e0208e2a2cd.jpeg)

![KakaoTalk_Photo_2021-10-29-23-43-35 004](https://user-images.githubusercontent.com/73742422/139526690-72b14220-d2d4-42f9-b50c-e054c154f811.jpeg)

### 반려생활

- ui 설계
- 리덕스 액션, 액션함수, 리듀서 설계
- 위치기반 서비스 개발(네이버 지도, reverse geocoding, gps)
    - 네이버 지도 연결
        - 연결 설정
            - 안드로이드
                - 네이버 인증키 연결(경로: "./android/app/src/main/AndroidManifest.xml")
                - sdk 레포지토리 경로 설정
                - 네이버 sdk 버전 싱크
            - ios
                - 네이버 인증키 연결(경로: "./ios/{프로젝트 명}/info.plist")
                - git-lfs 설치
                    
                    <aside>
                    💡 ios 환경에서는 네이버 맵의 사이즈가 너무커져서 100MB이상의 고용량 파일을 받기위해 git-lfs를 설치해야 사용이 가능하다(네이버 공식문서에도 나와있음)
                    
                    </aside>
                    
                - 네이버 sdk 버전 싱크
        - 실행 코드
            
            ```jsx
            const NaverMap = ({
                list, 
                center, 
                pinClick, 
                showMyLoc,
                isInit,
                setInit, 
                viewOpen, 
                setViewOpen, 
                pinIdx, 
                setShowMyLoc
            }) => {
                const [zoomLevel, setZoomLevel] = useState(16);
                const [smallRndSize, setSmallRndSize] = useState(10);
                const [smallRndOutSize, setSmallRndOutSize] = useState(5);
                const [bigRndSize, setBigRndSize] = useState(50);
                const [currentCenter, setCurrentCenter] = useState({...center});
                const [cnt, setCnt] = useState(0);
            
                const mapRef = useRef();
            
            	// 현재위치 조회시 현재위치 좌표 변경(자체기능 구현, 네이버 제공기능 X)
                useEffect(() => {
                    if(showMyLoc) {
                        console.log('--------------------useEffect---------------------------------');
                        mapRef.current.animateToCoordinate(center);
                        setCnt(0);
                        setCurrentCenter({
                            ...center
                        });
                    }
                }, [showMyLoc]);
            
            		// 지도 축척 변경시 현재위치 아이콘 사이즈 계산
                const sizeCalculate = (zoom) => {
                    console.log('----------------------------sizeCalculate--------------------------');
                    setZoomLevel(zoom);
                    if(zoom > 19) {
                        setSmallRndSize(0.5);
                        setSmallRndOutSize(2.5);
                        setBigRndSize(5);
                    }
                    else {
                        const compZoom = Math.floor(zoom);
                        switch(compZoom) {
                            case 19:
                                setSmallRndSize(1);
                                setSmallRndOutSize(1);
                                setBigRndSize(10);
                                break;
                            case 18:
                                setSmallRndSize(3);
                                setSmallRndOutSize(Platform.OS === 'android' ? 7 : 3);
                                setBigRndSize(20);
                                break;
                            case 17:
                                setSmallRndSize(5);
                                setSmallRndOutSize(Platform.OS === 'android' ? 7 : 3);
                                setBigRndSize(40);
                                break;
                            case 16:
                                setSmallRndSize(10);
                                setSmallRndOutSize(Platform.OS === 'android' ? 7 : 3);
                                setBigRndSize(50);
                                break;
                            default:
                                setSmallRndSize((2 ** (16-compZoom))*10);
                                setSmallRndOutSize(Platform.OS === 'android' ? 7 : 3);
                                setBigRndSize((2 ** (15-compZoom))*100); 
                                break;
                        }
                    }
                }
            
                return (
                    <NaverMapView 
                        ref={mapRef}
                        style={{flex: 1}}        
                        showsMyLocationButton={false}
                        center={{...center, zoom: zoomLevel}}
                        onTouch={e => {
                            console.log('onTouch', e)
                        }}
                        onCameraChange={e => {
                            console.log('onCameraChange', e);
                            const latitude = e.latitude.toFixed(7);
                            const longitude = e.longitude.toFixed(7);
                            const lat1 = currentCenter.latitude.toFixed(7);
                            const long1 = currentCenter.longitude.toFixed(7);
            
                            if((latitude == lat1) && (longitude == long1)) sizeCalculate(e.zoom);
            
                            if(cnt >= 2) {
                                if((latitude != lat1) || (longitude != long1)) {
                                    if(isInit) setInit(false);
                                }
                            }
                            if(cnt >= 1) {
                                if((!isInit) && ((latitude != lat1) || (longitude != long1))) setShowMyLoc(false);
                            }
                            setCnt(cnt+1);
                            
                        }}
                        onMapClick={e => {
                            console.log('onMapClick', e)
                            if(viewOpen) setViewOpen(false);
                        }}
                        zoomGesturesEnabled={true}
                        zoomControl={true}
                    >
                        {/* <Marker coordinate={P0} onClick={() => console.log('onClick! p0')}/> */}
                        {
                            list.map((item) => {
                                const {idx, coordinatesX, coordinatesY, title} = item;
                                return (
                                    <Marker 
                                        coordinate={{latitude: coordinatesY, longitude: coordinatesX}}  
                                        image={ICON_MAP_MARKERPIN}
                                        onClick={() => {
                                            pinClick(item)
                                        }}
                                        width={((pinIdx === idx)&&viewOpen) ? 48 : 37}
                                        height={((pinIdx === idx)&&viewOpen) ? 57 : 45}
                                        key={idx}
                                        caption={{
                                            text: title,
                                            textSize: 13,
                                            color: '#2C2C2E',
                                        }}
                                    />
                                );
                            })
                        }
            
                        { (showMyLoc || isInit) && <Circle coordinate={{...center}} radius={smallRndSize} color={'#FF3B30'} outlineColor={'#fff'} outlineWidth={smallRndOutSize} />}
                        { (showMyLoc || isInit) && <Circle coordinate={{...center}} radius={bigRndSize} color={'#FF3B301A'} outlineWidth={1} outlineColor={'#FF3B30'} />}
                    </NaverMapView>
                );
            }
            ```
            
        - 실행 결과
        
        ![KakaoTalk_Photo_2021-10-29-23-43-35 009](https://user-images.githubusercontent.com/73742422/139526766-d97ceb6e-d6b0-4840-bf5a-b801710cfabb.jpeg)
        
    - 네이버 reverse geocoding 연결(현재위치 주소 조회 관련)
        
        <aside>
        💡 reverse geocoding: 위도와 경도 정보를 이용하여 주소로 역변환해주는 기능
        → 이것과 반대되는 기능: geocoding(주소정보를 이용해 좌표를 포함한 상세정보로 변환해주는 기능)
        → 이러한 기능들은 지도 api 서비스를 제공하는 업체에서는 거의 전부 다 제공함
        
        </aside>
        
        - 실행코드
            
            ```jsx
            export const NaverReverseGeocoding = (coords) => {
            		// coords 파라메터 자료형은 String
            		// 형식은 '${경도},${위도}' 형식으로 들어가야 함
                return new Promise((resolve) => {
                    fetch(`https://naveropenapi.apigw.ntruss.com/map-reversegeocode/v2/gc?coords=${coords}&orders=legalcode&output=json`, {
                        headers: {
                            'X-NCP-APIGW-API-KEY-ID': CLIENT_ID,  // 네이버 client ID
                            'X-NCP-APIGW-API-KEY': CLIENT_SECRET  // 네이버 client secret
                        }
                    })
                    .then((res) => {
            						// json 결과값을 js 객체로 역변환해서 반환
                        resolve(res.json());
                    })
                });
            }
            ```
            
    - gps 기능
        - 안드로이드 권한 설정
            - AndroidManifest.xml 파일에 아래의 코드 추가
            ```xml
            <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
            ```
        - ios 권한 설정
            - info.plist 파일에 아래의 코드 추가
                
                ```jsx
                <key>NSLocationWhenInUseUsageDescription</key>
                <string>{위치 권한과 관련해 사용자에게 보여질 문장}</string>
                ```
                
        - gps 모듈 설치
            
            <aside>
            💡 모듈명: react-native-geolocation-service
            참고) 이 모듈을 통해 ios의 위치 설정 권한을 요청할 수 있음(안드로이드는 react native에서 제공하는 별도 컴포넌트가 있음)
            
            </aside>
            
        - 권한 요청 메서드 작성
            
            ```jsx
            const requestPermission = async () => {
                try {
                  if(Platform.OS === 'android') return await PermissionsAndroid.request(PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION);
                  else return await Geolocation.requestAuthorization('whenInUse');
            
            			//return 값: granted(허용), denied(거부), never_ask_again(다시 묻지 않음(거부)), disabled(사용불가), restricted(사용제한)
                }
                catch(e) {
                  console.log('HomeContainer requestPermission: ', e);
                }
              }
            ```
            
        - 위치정보 요청 메서드 작성
            
            ```jsx
            const getCoordinate = (res, catArr) => {
                if(res === 'granted') {
                  Geolocation.getCurrentPosition((pos) => {
                    const {longitude, latitude} = pos.coords;
                    console.log('pos: ', latitude, longitude);
                    
                  },
                  error => {
                    console.log('===================================================');
                    console.log('getCoordinate error: ', error);
                  },
                  {
                    timeout: 1000, // 타임아웃(ms) => ios에선 타임아웃을 안걸어주면 위치 반환이 안됨: 타임아웃 default가 INFINITY(무한대)이기 때문
                    enableHighAccuracy: true // 위치 고정밀 조회 사용여부
                  })
                }
                else { }
              }
            ```
            
        

### 앱 메인화면

- 메인화면 ui 전면 재설계
    - **변경 전**
        
        <img width="377" alt="스크린샷 2021-10-30 오후 5 22 26" src="https://user-images.githubusercontent.com/73742422/139526789-e9a5a4fe-0cf3-4817-8997-dae45597455c.png">
        
    - **변경 후**
        
        ![KakaoTalk_Photo_2021-10-29-23-43-35 014](https://user-images.githubusercontent.com/73742422/139526798-b676b983-3add-475a-8af4-bfc2a6fb0e82.jpeg)

        
        - ui 구조 변경
        - 캐러셀 **자체 제작** 후 연결
        - 메인화면 노출 날짜별 추천 1개 노출 → 반려공감 리스트 형으로 변경

### 반려공감

- 반려공감 전면 재설계
    - **변경 전 ui**
        
        <img width="376" alt="스크린샷 2021-10-30 오후 5 30 17" src="https://user-images.githubusercontent.com/73742422/139526811-9be308ae-e651-4c46-9b62-6a7a5f27a4c2.png">
        
    - **변경 후 ui**
        
        ![KakaoTalk_Photo_2021-10-29-23-43-35 012](https://user-images.githubusercontent.com/73742422/139526819-5a20bd93-856e-4a97-baee-77634dd70f37.jpeg)
        
        - 카테고리 구조: 1 depth → 3depth 구조로 변경
        - 반려공감 자체 검색기능 추가
        - 아이템 컴포넌트 ui 수정

### 공통 사용가능 컴포넌트/메서드 제작

- SelectMenu
    
    ![KakaoTalk_Photo_2021-11-06-21-37-51 006](https://user-images.githubusercontent.com/73742422/140611631-9d613cfe-ca48-4dde-b1bb-8253cc514c2a.jpeg)
    
    - 영역별로 공통적으로 적용가능한 menu select 컴포넌트
    - 형식에 맞춰서 버튼 title과 실행 함수를 선언
    
    **코드**
    
    ```jsx
    export const SelectMenu = ({visible, closeAction, actionList, isCamera=false}) => {
        return (
            <Modal animationType='fade' transparent={true} visible={visible} onRequestClose={() => closeAction()}>
                <SafeAreaView style={styles.container}>
                    <Pressable
                        style={{
                            flex: 1,
                            justifyContent: 'center',
                            alignItems: 'center'
                        }}
                        onPress={() => closeAction()}
                    >
                        <View style={styles.selectBox}>
                            {
                                actionList.map((item, index) => {
                                    const {title, action} = item;
                                    return (
                                        <TouchableOpacity 
                                            style={[styles.button, {borderBottomWidth: (index === actionList.length-1) ? 0 : Platform.select({android: 0.5, ios: 0.25})}]}
                                            key={index}
                                            onPress={() => {
                                                if(!isCamera) closeAction();
                                                else {
                                                    if(Platform.OS === 'android') closeAction();
                                                }
                                                action();
                                            }}
                                        >
                                            <Text style={styles.buttonTxt}>{title}</Text>
                                        </TouchableOpacity>
                                    );
                                })
                            }
                        </View>
                    </Pressable>
                </SafeAreaView>
            </Modal>
        );
    }
    ```
    
    - 코드 설명
        - parameter
            - visible: modal 노출,비노출 여부
            - closeAction: modal 비노출 함수
            - actionList: 메뉴로 표시할 버튼과 그에 따른 액션 함수(title(버튼 title), action(해당 버튼 클릭시 실행 함수))
            - isCamera: 카메라 관련 SelectMenu인 경우
        - 버튼들이 표시될 grid view 위에 파라미터로 받아온 actionList를 map으로 출력시켜 주는 구조
    
- TabButton
    
    ![KakaoTalk_Photo_2021-11-06-21-37-51 004](https://user-images.githubusercontent.com/73742422/140611640-c6074be6-9a71-457e-acfb-dbc644d979e1.jpeg)
    
    ![KakaoTalk_Photo_2021-11-06-21-37-51 005](https://user-images.githubusercontent.com/73742422/140611645-4507b56c-01a7-4952-81d6-1750454fb9ec.jpeg)
    
    - 영역별로 공통적인 디자인 속성이 적용되는 탭바 컴포넌트 제작
    
    **코드**
    
    ```jsx
    const TabButton = ({selected, onPress, titles}) => {
        return (
            <SafeAreaView style={styles.container}>
                {
                    titles.map((item, index) => {
                        return (
                            <TouchableOpacity
                                style={[styles.button, (selected === (index+1)) ? styles.selected : styles.unselected]}
                                onPress={() => onPress(index+1)}
                                key={index}
                            >
                                <Text style={[styles.title, (selected === (index+1)) ? styles.selectedTitle : styles.unselectedTitle]}>{item}</Text>
                            </TouchableOpacity>
                        );
                    })
                }
    
            </SafeAreaView>
        );
    }
    
    const styles = StyleSheet.create({
        container: {
            width: '100%',
            height: 48,
            borderBottomWidth: 1,
            borderBottomColor: '#E9E9EB',
            paddingHorizontal: 16,
            flexDirection: 'row',
            backgroundColor: '#fff'
        },
        button: {
            flex: 1,
            justifyContent: 'center',
            alignItems: 'center'
        },
        selected: {
            borderBottomWidth: 2,
            borderBottomColor: '#3A86F6',
        },
        unselected: {
            borderBottomWidth: 0
        },
        title: {
            fontSize: 16,
            lineHeight: 23
        },
        selectedTitle: {
            color: '#3A86F6',
            fontFamily: fontConvert('SCDream5')
        },
        unselectedTitle: {
            color: '#616161',
            fontFamily: fontConvert('SCDream4')
        },
    })
    ```
    
    - ScrollView 위에 표시하는 구조가 아니기 때문에 ui상 표시할 수 있는 탭버튼 수의 한계가 있음(최대 4개까지)
- Banner(배너)
    
    ![KakaoTalk_Photo_2021-11-06-21-37-51 003](https://user-images.githubusercontent.com/73742422/140611654-21d18598-8272-43f5-be68-dc9043fd2445.jpeg)
    
    - 3D 효과가 적용된 캐러셀 배너 컴포넌트 제작
    - 기존 외부 모듈 컴포넌트는 원하는 효과를 쉽게 가져올 수 있는 컴포넌트가 없어 자체 제작 결정
    
    **코드**
    
    ```jsx
    import React, { useEffect, useRef, useState } from 'react';
    import { 
        FlatList,
        View, ViewPropTypes 
    } from 'react-native';
    import PropTypes from 'prop-types';
    import CONF from '../../../config';
    import { Component } from 'react';
    
    const SCREENWIDTH = CONF.WINDOW_WIDTH;
    const SCREENHEIGHT = CONF.WINDOW_HEIGHT;
    const PAGEWIDTH = SCREENWIDTH-32;
    const PAGEHEIGHT = ((180/343)*PAGEWIDTH);
    const GAP = 8;
    const OFFSET = 8;
    
    class BannerSlider extends Component {
        constructor(props) {
            super(props);
            
            this.state = {
                page: 0,
                pagesIndexs: [ ...this.setPagesIndexs(props.pages) ],
                newPages: [ ...this.setFirstPages(props.pages) ],
                firstPagesLength: props.pages.length,
                autoScroll: ('autoScroll' in props) ? props.autoScroll : false,
                autoScrollInterval: ('autoScrollInterval' in props) ? props.autoScrollInterval : 5000,
                circleLoop: ('circleLoop' in props) ? props.circleLoop : false,
                sliderHeight: ('sliderHeight' in props) ? props.sliderHeight : 180,
                nextLoc: 0,
            } 
        }
    		// 페이지 별 고유 index 키 값 목록화
        setPagesIndexs(arr) {
            return arr.map((item) => item.key);
        }
    
    		// 넘어온 페이지 목록 -> 배너 view 목록
    		// 처음 넘어온 목록의 수가 배너를 적용하기 위해 필요한 최소한의 목록보다 작을 경우 그 목록의 개수를 늘려줌
        setFirstPages(arr) {
            const {circleLoop} = this.props;
    
            let newArr = [];
            let idx = 0;
            const firstLength = arr.length;
    
            for(let i of arr) {
                const {key, comp} = i;
                newArr.push({
                    num: idx++,
                    key,
                    comp
                })
            }
    
            if((newArr.length <= 3)&&circleLoop) {
                while(newArr.length !== 4) {
                    const firstIdx = newArr.length-firstLength;
                    const firstData = newArr[firstIdx];
    
                    const {key, comp} = firstData;
                    newArr.push({
                        num: idx++,
                        key,
                        comp
                    })
                }
            }
    
            return newArr;
        }
    
    		// 클래스 컴포넌트의 마운트가 완료됬을 때(타이머 효과 구현을 위해 적용)
        componentDidMount() {
            const {autoScroll, autoScrollInterval} = this.state;
            if(autoScroll) {
                this.timeout = setInterval(this.autoScrollAction, autoScrollInterval);
            }
        }
    
    		// 클래스 컴포넌트가 언마운트 될 때(타이머 interval 초기화)
        componentWillUnmount() {
            const {autoScroll, autoScrollInterval} = this.state;
            if(autoScroll) clearInterval(this.timeout);
        }
    
    		// 자동 스크롤 실행
        autoScrollAction = () => {
            const {page} = this.state;
    
            const offset = (PAGEWIDTH+GAP+0.0001)*(page+1);
            this.scrollRef.scrollToOffset({offset, animated: true});
    
            this.setState({
                ...this.state,
                page: page+1,
                nextLoc: offset
            })
        }
    
    		// 현재 페이지 index 반환
        returnPageIndex = (page, pagesArr) => {
            const {onPageChanged} = this.props;
            const {pagesIndexs} = this.state;
    
            const data = pagesArr[page];
            (typeof onPageChanged === 'function') && onPageChanged(pagesIndexs.indexOf(data.key));
        }
    
        render() {
            const {page, newPages, firstPagesLength, autoScroll, autoScrollInterval, circleLoop, sliderHeight, nextLoc} = this.state;
    
            return (
                <View style={{width: '100%', height: sliderHeight}}>
                    <FlatList 
                        ref={(ref) => this.scrollRef = ref}
                        automaticallyAdjustContentInsets={false}
                        contentContainerStyle={{
                            paddingHorizontal: (OFFSET + (GAP/2))
                        }}
                        onScrollBeginDrag={(e) => {
                            
                        }}
                        data={newPages}
                        decelerationRate='fast'
                        horizontal
                        keyExtractor={(item) => `${item.num}`}
                        onScroll={(e) => {
                            const newPage = Math.round(e.nativeEvent.contentOffset.x / (PAGEWIDTH+GAP))
                            let newState = {
                                ...this.state,
                                page: newPage
                            };
                            if(circleLoop&&(page !== newPage)&&(page < newPage)&&(newPage === newPages.length-2)) {
                                let newArr = newPages;
                                const newLength = newPages.length+1;
        
                                const firstDataIdx = (newLength-firstPagesLength)-1;
                                const firstData = newPages[firstDataIdx];
                                const lastData = newPages[newPages.length-1];
                                    
                                newArr.push({
                                    num: lastData.num+1,
                                    key: firstData.key,
                                    comp: firstData.comp
                                })
                                newState = {
                                    ...newState,
                                    newPages: newArr
                                }
                            }
        
                                if(page !== newPage) this.returnPageIndex(newPage, newState.newPages);
    
                            this.setState({
                                ...newState
                            })
                        }}
                        pagingEnabled
                        renderItem={(item) => {
                            const {num, key, comp} = item.item;
                            return (
                                <View style={{width: PAGEWIDTH, height: '100%', marginHorizontal: (GAP/2), borderRadius: 10}} key={`${num}`}>
                                    {comp()}
                                </View>
                            );
                        }}
                        snapToInterval={PAGEWIDTH+GAP+0.0001}
                        snapToAlignment='start'
                        showsHorizontalScrollIndicator={false}
                    />
                </View>
            );
        }
        
    }
    
    BannerSlider.propTypes = {
        pages: PropTypes.array.isRequired,
        autoScroll: PropTypes.bool,
        autoScrollInterval: PropTypes.number,
        circleLoop: PropTypes.bool,
        sliderHeight: PropTypes.number,
        onPageChanged: PropTypes.func
    }
    
    BannerSlider.defaultProps = {
        autoScroll: false,
        autoScrollInterval: 5000,
        circleLoop: false,
        sliderHeight: 180,
    }
    
    export default React.memo(BannerSlider);
    ```
    
    - 코드 설명
        - parameter
            - pages: 배너로 표시하게 될 view list
            - autoScroll: 자동 스크롤 활성화 여부(boolean)
            - autoScrollInterval: 자동 스크롤 간격(defalut: 5000ms(5초))
            - circleLoop: 무한 루프 활성화 여부(boolean)
            - sliderHeight: 페이지 높이
            - onPageChanged: 페이지가 바뀌었을 때 페이지별 고유 index 반환
        - state
            - pages: 현재 page number index
            - pagesIndexs: 페이지별 고유 index,
            - newPages: 배너로 표시하게 될 뷰 list,
            - firstPagesLength: parameter로 받아온 pages의 길이
            - autoScroll: 자동 스크롤 활성화 여부(boolean)
            - autoScrollInterval: 자동 스크롤 간격(defalut: 5000ms(5초))
            - circleLoop: 무한 루프 활성화 여부(boolean)
            - sliderHeight: 페이지 높이
            - nextLoc: 다음 페이지로 이동했을 시 이동해야 하는 offset 값
    - 장점
        - 단순 이미지만 표시할 수도 있고 view 태그 안에 특정한 컴포넌트(버튼, image 등)들을 표시할 수 있음 ⇒ 페이지별 customizing 가능
        - 3D효과 구현 가능
    - 단점
        - 안드로이드의 경우 페이지를 이동할 수록 제 위치에 안착이 안되는 경우가 있음
            - 확인 결과
                - 배너 scroll에 지표가 될 화면의 너비가 안드로이드에서 소수점 단위로 나오고 있음(ios는 정수단위로 출력되고 있음)
                - 너비가 소수점 단위로 나오면 어떤 경우로든 아주 미세한 오차가 발생하게 됨
                - 이 경우가 누적되면 페이지가 제 위치에 안착하지 못하게 됨
                - 추가 연구 필요
