# LOGI_BOOGIE 정리 블로그

## 개요
- 프로젝트명: ROGIE_BOOGIE
- 참여자: 강민지(wbclair7@konkuk.ac.kr), 권미경(kmk3942@konkuk.ac.kr) [2명]
- 프로젝트 일자: 2019년 12월 ~ 진행중
- 주제: Turtlebot3 Waffle과 Open Manipulator를 이용한 로봇 제작

현재 이 블로그는 연구상황 정리, 참고 자료 정리와 더불어 다른 사람들이 진행 시 겪을 수 있는 오류에 보탬이 되고자 개설하게 되었다. 진행 상황과 더불어 참고한 문헌 혹은 참고할만한 문헌, 에러 해결 방법 등에 대해서 올릴 예정이다.

-------------
## 2020년도 겨울방학 이전 진행상황
- 기존의 RaspberryPi 대신 Jetson TX2를 Turtlebot에 탑재하여 개발환경 구축(Ubuntu 16.04 / ROS kinetic)
- 259장의 이미지를 YOLO training 후 화살표 이미지 detect 성공

```
Jetson TX2의 환경 구축단계에 상당한 시행착오를 겪었음. 
단계는 다음과 같다 ->

- Jetson TX2에 최신 버전인 jetpack 4.3을 설치 (우분투 18.04 버전으로 ROS melodic 버전이 stable하지 않았음)

- Jetpack 다운그레이드를 위하여 초기화를 하고 우분투 16.04로 변경하였음. 
jetpack 3.3.1을 사용하였으나 설치과정에서 중간에 필요한 툴들이 깔리지 않았음.

이에 이어 OpenCV 및 CUDA와 CUdnn을 직접 설치하였으나 내장 메모리의 부족 단계로 다시 초기화를 진행

- Jetpack 3.3.1을 재설치
```

--------------
## 2020.03.05

### TIP (참고용)
-리눅스에서 압축풀기: 
https://dbrang.tistory.com/625


- **Jetpack 3.3.1로 새로 설치**
  - 이전에 설치한 Jetpack에서 OpenCV와 Cuda, Cudnn이 제대로 설치되지 않아 개별적으로 설치해서 용량이 부족하다고 판단했기 때문
  
  [참고사이트](https://www.guruhong.com/33)

  이전에 설치되어 있던것에서 재설치가 완벽하게 진행되었음.
  
  이전에는 CUDA 및 CUdnn의 경우 root단에서 확인해보면 실제로 안깔려 있었음.
  jetpack 설치를 다양한 노트북으로 진행했는데 성능의 차이인가 라는 의문이 있음.

  tip -> 재설치를 하게 되는 경우 wifi보단 랜선을 연결하여 설치를 진행하는게 편하다. 또한 반드시 허브를 준비할 것

- **500G SSD 사용하기로 결정**
  - 64G SD 사용하려고 했으나 500G가 더 안정적이라고 판단했기 때문
  - 메모리 문제를 해결하고 다양한 필요 프로그램 설치에 어려움을 없앤다.
  - 사용할 SSD: BARACUDA SATA SSD
  
---

## 2020.03.06
- **SSD로 부팅 설정**
  - Booting Rootfs off SD Card on Jetson TX1

  ssd 를 젯슨에 연결하게 되면 재부팅을 진행한다. 처음에 연결하고 ssd를 확인할 수 없었으나 재부팅 후 확인 가능했음.

  진행 방법: https://www.youtube.com/watch?v=ZpQgRdg8RmA&t=377s

  유튜브에서 시키는대로 따라하면 거의 진행은 가능하다.

  ```
  해결하지 못한 문제점:
  현재 젯팩 3.3.1을 이용하고 있는데 커널이 최신버전으로 올라와 있다.
  git에서 예전 버전을 찾았지만 sh명령어 실행도중 에러가 발생함.
  ```

---

## 2020.03.09
### Kernel과의 싸움
- **SSD를 주메모리로 설정**
참고: Youtube video `Develop on SSD - NVIDIA Jetson TX Dev Kits`
https://www.youtube.com/watch?v=ZpQgRdg8RmA 하던 중에

  `./makeKernel.sh`에서 오류남:
          ```
          recipe for target 'drivers' failed
          ```
      - 그래서 구글링
      https://devtalk.nvidia.com/default/topic/1019770/error-when-building-device-tree-in-l4t-28-1-for-jetson-tx1/
      
    위 링크에서 NVIDA Guide 링크 있어서 참고하여 시도 : `NVIDIA Tegra Linux Driver Package Development Guide`
      https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-271/index.html
      - 하다가 에러가 발생하여 중도에 멈춤

- **opencv3.4.6으로 upgrade**
    - 사용했던 Jetpack 3.3.1에 내장된 opencv3.3.1으로는 gstreamer가 작동하지 않기 때문
    - OpenCV를 다운받고 귀가

- 다음에 할 일 OpenCV 확인하기
    
---

## 2020.03.10
- 오자마자 opencv upgrade 돌려놓은거에 권한 잠겨있어서 암호 입력함

  ... 프로젝트 할 때는 무조건 화면 보호기 끄고 가기
  
- **USB Cam 실행을 위한 설정**
  - OpenCV 3.4.6으로 upgrade 성공 했는데도
      ```
      ~/project/jetson_nano/opencv$ python tegra-cam.py
      ```
      내장 Camera 실행이 안됨
      ```
      ImportError: No module named cv2
      ```
  - 서치 결과 python용 opencv가 설치되지 않아 생긴 문제로 판단
    - python opencv 설치 시도
        ```
        pip install opencv-python
        ```
        : 에러
        ```
        sudo apt install python-opencv
        ```
        : cv2 import 성공
  - 근데 다른 에러남
- **built in 카메라 실행 성공**
  - `use camera on jetson TX2` 
  https://devtalk.nvidia.com/default/topic/1022265/jetson-tx2/use-camera-on-jetson-tx2/
    - Honey_Patouceul
        ```
        Still unclear what is your use case, but I'll try to summarize:

        The onboard camera is a bayer sensor.

        If you access it through v4l2 interface, you'll get bayer format into CPU memory. Try:
        v4l2-ctl -d /dev/video0 --list-formats

        If you access it through gstreamer interface, then plugin nvcamerasrc can provide different formats (I420, NV12...) into NVMM memory. Try:
        gst-inspect-1.0 nvcamerasrc
        and see its src capabilities.
        For example:
        gst-launch-1.0 nvcamerasrc ! 'video/x-raw(memory:NVMM),width=640, height=480, framerate=30/1, format=NV12' ! nvvidconv flip-method=2 ! nvegltransform ! nveglglessink -e
        should show your camera capture in a window. Other plugins may convert into many other formats. Searching this forum you should find many examples.

        You may also have a look to Tegra MultiMedia API and Argus, depending on what you intend to do with it.
        ```
    - 핵심:
        ```
        gst-launch-1.0 nvcamerasrc ! 'video/x-raw(memory:NVMM),width=640, height=480, framerate=30/1, format=NV12' ! nvvidconv flip-method=2 ! nvegltransform ! nveglglessink -e
        ```
- **USB Cameara 실행 성공**
  참고: `How to Capture and Display Camera Video with Python on Jetson TX2`
  https://jkjung-avt.github.io/tx2-camera-with-python/
  ```
  $ python3 tegra-cam.py --usb --vid 1 --width 1280 --height 720
  ```
- 다음에 도전해볼 것
    `How to Capture Camera Video and Do Caffe Inferencing with Python on Jetson TX2`
    https://jkjung-avt.github.io/tx2-camera-caffe/

- **Darknet 설치**
  - 이전에 뜬 에러 `Video-stream stopped`로 인해 다른 다크넷(https://github.com/AlexeyAB/darknet) 설치
  참고: `Video-stream stopped! error`
  https://github.com/stereolabs/zed-yolo/issues/11
    - image_opencv.cpp 파일 수정
    참고: https://github.com/dlwnstjr2004/EskerJuneA/tree/master/src
    - makefile 실패
  - 이전에 설치했던 다크넷(https://github.com/pjreddie/darknet.git)으로 다시 설치함

- **Yolo mark 설치**
  참고: `[5] YOLO 데이터 학습`
  https://juni-94.tistory.com/10?category=802791

- **ROS 설치**
  참고: `ROBOTIS e-Manual Turtlebot3`
  http://emanual.robotis.com/docs/en/platform/turtlebot3/raspberry_pi_3_setup/#raspberry-pi-3-setup

---

## 2020.03.12.

- **OpenCR 재설치**
  - Ubuntu에서 Arduino IDE 실행 안됨
  - 민지 개인노트북에 Arduino IDE 설치하여 OpenCR board에 16.04 버전으로 재설치

---

## 2020.03.16.
- **SLAM 시도(실패)**
  - SLAM 시도해 보려고 했으나 turtlebot bringup에서 오류
    ```
    [ERROR] [1584332348.587128343]: An exception was thrown: open: No such file or directory
    ``` 
    ```
    [ERROR] [1584332750.539349148]: An exception was thrown: open: No such file or directory
    [ERROR] [1584332751.183600]: Error opening serial: [Errno 2] could not open port /dev/ttyACM0: [Errno 2] No such file or directory: '/dev/ttyACM0'
    ```
  - 계속 ttyACM0 포트 관련해서 에러메세지가 뜨는데 구글링해도 답이 안나옴
  - 문득 Jetson의 USB 포트가 3.0인데, 사용하고 있던 USB Hub의 포트가 2.0인 것을 보고 그 문제가 원인인 것으로 예상 - Hub를 바꿔보기로 결정

---

## 2020.03.17.
- **ttyACM0 문제 해결**
  - USB Hub 3.0으로 변경했더니 bringup 잘되고 포트 에러도 안남
- **SLAM 재시도**
  - launch는 다 되는데 teleop 실행 후 조작이 안됨

---

## 2020.03.21.
### OpenCR 보드에 귀신들림..
- **OpenCR 확인**
  - OpenCR 확인을 위해 로봇 분해
  - OpenCR의 PUSH Sw1을 눌러 DXL의 작동을 확인해본 결과 1번 Motor만 돌아가고 2번은 돌아가지 않음
    - 로봇케이블을 바꿔도 증상 동일
    - Turtelbot3 burger(17번)의 DXL을 연결해도 증상 동일
    - OpenCR(17번)을 변경해도 증상 동일
  - 그 와중에 waffle에 쓰던 OpenCR 전원스위치(Toggle) 고장 -> 부품함에 있던 새 OpenCR로 변경(어쨋든 작동 안함)
  - Arduino IDE로 turtlebot3_setup_motor 열고 Serial monitor 켜서 Setup right motor 해봤는데 에러 발생
    ```
    [TxRxResult] There is no status packet!
    ```
  - R+ Manager 2.0 이용해보려고 했으나 OpenCM 보드가 필요하여 못함
  - Robotis에 문의 -> Dynamixel Wizard를 이용해보고, 안되면 A/S 맡기라고 함
  
---

## 2020.03.28.
- **Dynamixel Wizard 이용**
  - Minji_UBUNTU에 Dynamixel Wizard 설치
  - OpenCR에 usb_to_dxl 업로드
  - Scan 했을 때 아무 변화 없음
  - Recovery 실패
- 다음 주 평일 중에 문의전화 예정

---

## 2020.03.31.
- **Dynamixel Wizard를 이용한 점검**
  - ROBOTIS 본사 기술지원팀에 문의한 결과 통신문제일 수도 있다는 가능성을 들었음.
  - OpenCR에 usb_to_dxl을 업로드하여 Dynamixel Wizard를 시도해보았으나 아래 링크에서 U2D2가 아닌 OpenCR을 이용하면 불안정할 수 있다고 하여 U2D2로 시도
    - 참고: `OpenCR Board no longer recognizing dynamixels after firmware update`
    https://github.com/ROBOTIS-GIT/OpenCR/issues/203
    - 참고: `DYNAMIXEL Wizard 2.0 Firmware Recovery with U2D2`
    https://youtu.be/PgbIAK2Qg1Y
    - 학교노트북(5번)의 윈도우에 Dynamixel Wizard 2.0을 설치하고 Dynamixel과 연결 성공
  - 2번 모터의 ID가 ID1으로 되어있어 ID2로 변경하였으나 OpenCR SW test 결과 같은 증상 보임(1번 모터만 작동)
  - 2번 모터를 Firmware Recovery해 보았으나 작동 안함
  - Dynamixel Wizard를 이용한 자가진단 [도구]->[자가진단]
    - 2번 모터를 자가진단해 본 결과 나머지는 정상이었으나 '속도가 사양보다 낮습니다'라는 문구가 뜸
    - 1번 모터를 자가진단 해 본 결과 나머지는 정상이었으나 '속도가 사양보다 높습니다'라는 문구가 뜸
    - 자가진단을 하면서 팩토리리셋(Factory reset)이 되었기 때문에 복구도 해 보았으나(v44, v43 버전 둘 다 해봄) 두 모터 다 작동 안함
  - A/S 요청 필요하다고 판단

---

## 2020.04.16.
- **A/S 센터로 배송 보냄**
  - Tutlebot3 Waffle Pi의 Dynamixel 2개, OpenCR 1개

---

## 2020.04.18.
- **수리 완료하여 부품 다시 받음**
  - OpenCR 스위치: 전원을 양쪽에서 공급하면 일시적으로 작동하지 않을 수 있으나 현재는 문제 없음
  - Dynamixel: setting 문제로 현재는 문제 없다고 함

---

## 2020.05.15.
- **분해한 Turtlebot 다시 조립**
- **OpenCR Dynamixel 작동 test**
  - 또 안됨!!!!! PUSH SW1, SW2 둘다 해봤으나 2번 Dynamixel이 여전히 작동하지 않음
  - teleop 안됨
- **다음 계획**
  - 준형님이 관리하던 Waffle의 Dynamixel을 분해하여 test해도 되는지 여쭤볼 것
  - ROBOTIS에 전화해서 setting후 test 완료된건지 확인
  - U2D2로 다시 Dynamixel setting

---

## 2020.05.22.
- **다른 와플로 test**
  - 우리 dynamixel을 다른 OpenCR에 연결했을 때 ID2 작동 X
  - 다른 dynamixel을 우리 OpenCR에 연결했을 때 ID2 작동 X
- **U2D2로 다시 Dynamixel setting 확인**
- **ROBOTIS에 문의전화**
  - 우리가 시도했던 모든 것들을 시켜서 다시 해봤지만 작동하지 않아서 SMPS를 확인해보라고 하심
  - Dynamixel(XM430-W210-T)이 12V를 인가해야 정상 작동하는데, 우리가 19V짜리 SMPS를 사용하고 있었음
  - *결론: 우리는 한달 넘게 멍청한 짓을 했다. SMPS 잘 확인하자.*
  `12V`

---

## 2020.07.11.
- **와플 환경 다시 test**
  - jetson pinout 참고 사이트: https://www.jetsonhacks.com/nvidia-jetson-nano-j41-header-pinout/
   
   (화살표 있는 부분이 1)
   
  - Jetson TX2와 호환이 안되는 것 같음
  - lidar 값을 받아오질 못하고 텔레옵키 또한 작동 안함 (오로지 roscore 구동 및 그 외의 launch만 진행됨)
  - 라즈베리파이에 환경 구축해둠 (원래 세팅대로 우선 진행하고 젯슨에서는 영상처리 하여 싱크 맞춰서 라즈베리에 넘겨볼 계획)
  
- **할 일**
  - 회사 문의 게시판에 세팅 문의
  - 대회 지원금 외 교내 문의
  - conda - pytorch, cv .. etc download
  - box image - annotation

---

## 2020.07.18.
- **Conda 설치**
- 상자 사진 수집
#### 할일
- 적재 알고리즘 개발
- 이미지 Annotation
  
---

## 2020.07.30.
- ROBOTIS 문의 결과 SBC setting은 도움줄 수 없다고 답변받음
  - Jetson과 RaspberryPi를 따로 구동해야 할 듯함
- **SLAM 시도**
  - 라즈베리파이로 SLAM 실행
  - 로봇팔은 인식하지 않는 것으로 보임
    - teleop로 이동시켜봐도 문제 없어 보임(라이다와의 거리가 너무 가까워서? 이유는 확실하지 않음)
- **GPU**
  - Google의 Cloud GPU를 사용하려고 했으나 페이지에서 다음 절차로 안넘어가짐

---

## 2020.07.31.
- **Box Detecting**
  - 다음 링크를 참고할 예정
  fontenay-ronan.fr/computer-vision-a-box-on-a-industrial-conveyor/
  - 수정 중

#### 내일 할 일
- Box Detecting 알고리즘 개발
- Jetson - Raspberry 간 Sync
- Google Cloud GPU

---

## 2020.08.01.
- **진행정리**
  - GCP는 대회 지급 연구비에서 해결하기 어려운 가격 / 사용하려 해도 사이트에서 가입이 안됨 -> google colab pro 사용 예정 (2달)
  - 로봇팔에 맞춰서 박스 크기 다양하게 프린트 할 예정
  - 컨베이어벨트는 구매 예정
  
- **박스 인식 알고리즘**
  - 픽셀단위로 접근하고 빛 조절 진행하면 박스 이미지 처리할 필요 없을 것으로 예상
  
- **바코드 인식**
  - 만약 박스 처리에 인공지능이 필요없다면 바코드 부분에만 사용하면 됨(:만약 바코드가 제 위치에 없을 때 알람을 주는 형태로 가야하지 않을까?)
  - 바코드 생성은 홈페이지에서 진행할 예정
  - 바코드 위치 인식은 영상처리로 진행(다음 링크를 참고할 예정)
  https://github.com/kairess/qrcode_barcode_detection
  pzbar에서 decoding도 됨
  - jetson tx2 version으로 수정하여 코드 생성 - barcode_jetson.py 참고
  ```
  $ python3 barcode_jetson.py --usb --vid 1 --width 1280 --height 720
  ```
  
- **sync or sbc problem**
  - jetson tx2 doesn't get lidar data -> maybe ttyACM0 and ttyUSB0 problem
  - 2 ways to solve the problem 
   1. change ttyUSB0 to ttyACM0
   2. use two board and sync the data
   
   ```
   tty checking command: dmesg | grep tty
   ```
   ttyACM0: https://github.com/NVIDIA-AI-IOT/turtlebot3
   maybe have to check kernel
   
- **Box Detecting**
  - 컨투어를 찾아서(초록색) 면적이 가장 큰 컨투어를 직사각형으로 표시(빨간색)
  - 조명에 따라 결과가 달라지므로 threshold trackbar를 추가하여 적절한 threshold값을 찾은 후 대입해 줌(새천년관 1006호에서는 142가 적합함)
  - 오픈소스를 수정한 코드
    ```py
    import numpy as np
    import cv2

    # # 트랙바를 위한 dummy 함수
    # def nothing(x):
    #     pass

    #Read image
    #img = cv2.imread('box-1.jpg')
    cap = cv2.VideoCapture(1)   # 내장 camera인 경우: 0 / USB camera인 경우: 1

    cv2.namedWindow('image', cv2.WINDOW_NORMAL)
    # cv2.createTrackbar('threshold', 'image', 0, 255, nothing)  # 트랙바 생성
    # cv2.setTrackbarPos('threshold', 'image', 127)  # 트랙바의 초기값 지정

    while(True):
        ret, img_color = cap.read()  # 카메라로부터 이미지를 읽어옴

        # 캡처에 실패할 경우 다시 loop의 첫 줄부터 수행하도록 함
        if ret  == False:
            continue

        #Gaussian blur
        blurred = cv2.GaussianBlur(img_color, (5, 5), 0)

        #Convert to graysscale
        gray = cv2.cvtColor(blurred, cv2.COLOR_BGR2GRAY)

        #Autocalculate the thresholding level
        threshold = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 11, 2)

        #Threshold
        # low = cv2.getTrackbarPos('threshold', 'image')  # 트랙바의 현재값을 가져옴
        # retval, bin = cv2.threshold(gray, low, 255, cv2.THRESH_BINARY)    # 트랙바의 threshold값 받아옴
        retval, bin = cv2.threshold(gray, 142, 255, cv2.THRESH_BINARY)  # 새천년관 1006호에서 threshold: 142

        #Find contours
        contours, hierarchy = cv2.findContours(bin, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)

        #Sort out the biggest contour (biggest area)
        max_area = 0
        max_index = -1
        index = -1
        for i in contours:
            area = cv2.contourArea(i)
            index = index+1
            if area > max_area:
                max_area = area
                max_index = index

        #Draw the raw contours
        cv2.drawContours(img_color, contours, max_index, (0, 255, 0), 3 )
        #cv2.imwrite("box-1-biggest-contour.png", img)

        #Draw a rotated rectangle of the minimum area enclosing our box (red)
        cnt = contours[max_index]
        rect = cv2.minAreaRect(cnt)
        box = cv2.boxPoints(rect)
        box = np.int0(box)
        img_color = cv2.drawContours(img_color, [box], 0, (0, 0, 255), 2)

        #Show original picture with contour
        cv2.imshow('image', img_color)

        if cv2.waitKey(1) & 0xFF == 27:  # 1초 단위로 update되며, esc키를 누르면 탈출하여 종료
            break
            
    cap.release()
    cv2.destroyAllWindows()
    ```
  - 컨투어와 영역 면적만으로 박스를 detecting할 수 있는가? Harris 코너검출을 함께 이용하는 방법은?

*오픈소스 활용으로 인해 상자와 바코드 인식을 위한 딥러닝 학습이 무의미해짐 → 다른 아이디어 필요*

#### 다음 주 할 일
- 강민지
  - kernel 문제 확인(ttyACM0 ↔ ttyUSB0 포트 변경)
    - ttyUSB0: LDS(LiDAR)
    - ttyACM0: OpenCR
  - open-manipulator 구동
- 권미경
  - 트럭 상황 3D로 표현하는 방법 찾기(C++ or Python)
  - 적재 알고리즘 개발
  - 바코드 인식 코드 수정
- 공통
  - 딥러닝 활용 아이디어

---

## 2020.08.03.
! ping 확인해보기 (라이더)
- **sync 계획정리**
    1. flash는 아예 처음부터 진행해야함 - jetpack 최근 버전을 고려해보기
       용준오빠네 jetson tx2로 테스트해보고 만약 된다면 flash하기 (나머지는 내재되어있으니까 ssd -> ros -> yolo) (토요일 테스트)
       kernel만 수정 가능 (documentation 보기)
    2. Rpi와 tx2의 중간 ssd 등을 거치고 거기에 공유폴더 생성해서 활용
    3. 라우터 사용
    4. TX2는 micro-B USB RPi는 USB로 연결을 하여 호스트를 이더넷 취급하여 연결 (찾아봐야함) : 
    https://forums.developer.nvidia.com/t/how-to-communicate-between-raspberry-pi-3b-and-jetson-tx2/80559
  
- **3D로 데이터 보여주기**
  - OpenGL 사용? (3d 렌더링이 잘되어있다고함)
  - Matplotlib
  - pandas

- **아이디어**
 - 파손과 관련해서 퍼센트 나타내기?
 - 이더넷을 연결하는게 대회 명목과 맞을것으로 예상됨
 
- **Barcode Detecting**
  - 코드
    ```py
    import pyzbar.pyzbar as pyzbar
    import cv2

    cap = cv2.VideoCapture(1)

    i = 0
    while (cap.isOpened()):
        ret, img = cap.read()

        if not ret:
            continue

        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

        decoded = pyzbar.decode(gray)

        for d in decoded:
            x, y, w, h = d.rect

            barcode_data = d.data.decode("utf-8")   # 바코드 인식 결과
            barcode_type = d.type

            cv2.rectangle(img, (x, y), (x + w, y + h), (0, 0, 255), 2)

            text = '%s (%s)' % (barcode_data, barcode_type)
            cv2.putText(img, text, (x, y), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 255), 2, cv2.LINE_AA)

        cv2.imshow('img', img)

        key = cv2.waitKey(1)
        if key == ord('q'):
            break
        elif key == ord('s'):
            i += 1
            cv2.imwrite('c_%03d.jpg' % i, img)

    cap.release()
    cv2.destroyAllWindows()
    ```
  - `barcode_data` 변수가 스캔한 바코드 정보를 가지고 있음. 이 데이터를 이용하여 배송지를 분류
  
 ---
 
 ## 2020.08.04
 
 - **오픈매니퓰레이터 참고**
     https://github.com/youtalk/youfork
 - **flash**
     https://forums.developer.nvidia.com/t/jetson-tx2-change-kernel-without-full-flash/74029
        https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%2520Linux%2520Driver%2520Package%2520Development%2520Guide%2Fkernel_custom.html%23

--- 
## 2020.08.06.
- **Truck Visualization**
  - 개인 Windows(Pycharm 이용)에 `pygame`과 `OpenGL` 모듈 설치
  - 참고: https://blog.naver.com/samsjang/220708189400
  - 정육면체 화면에 그리기(블로그 참고, 코드 수정함)
  - 코드
    ```py
    import pygame
    from pygame.locals import *
    from OpenGL.GL import *
    from OpenGL.GLU import *
    import time

    # 각 꼭짓점
    vertices = ((1, -1, -1), (1, 1, -1),
                (-1, 1, -1), (-1, -1, -1),
                (1, -1, 1), (1, 1, 1),
                (-1, -1, 1), (-1, 1, 1))

    # 각 모서리(꼭짓점끼리의 연결)
    edges = ((0, 1), (0, 3), (0, 4),
            (2, 1), (2, 3), (2, 7),
            (6, 3), (6, 4), (6, 7),
            (5, 1), (5, 4), (5, 7))

    pygame.init()
    display = (800, 600)
    pygame.display.set_mode(display, DOUBLEBUF | OPENGL)

    gluPerspective(45, (display[0]/display[1]), 0.1, 50.0)  # 투영 양식
    glTranslatef(2, -3, -15)      # 바라보는 위치(상하, 좌우, 전후)

    # 육면체를 그림
    glBegin(GL_LINES)       # OpenGL에게 직선을 그릴 것이라는 것을 알려줌
    for edge in edges:      # 각 꼭짓점을 직선으로 연결
        for vertex in edge:
            glVertex3fv(vertices[vertex])
    glEnd()                 # OpenGL에게 작업이 끝났음을 알려줌

    pygame.display.flip()       # 화면에 보여줌
    time.sleep(1)       # 기다림
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)  # OpenGL에 쓰인 버퍼를 비움
    ```
  - 위 코드를 활용하여 트럭 상태 시각화하는 알고리즘 개발 예정

---
## 2020.08.07.
- **Truck Visualization**
  - 참고: https://blog.naver.com/samsjang/220717571305
  - 정육면체 두 개 화면에 그리기, 육면체 하나는 표면 색깔 입히기(블로그 참고, 코드 수정함)
  - 코드
    ```py
    # https://blog.naver.com/samsjang/220708189400
    # https://blog.naver.com/samsjang/220717571305
    import pygame
    from pygame.locals import *
    from OpenGL.GL import *
    from OpenGL.GLU import *
    import time

    # 도형에 필요한 변수 선언
    ## 작은 육면체의 각 꼭짓점
    vertices = ((1, -1, -1), (1, 1, -1),
                (-1, 1, -1), (-1, -1, -1),
                (1, -1, 1), (1, 1, 1),
                (-1, -1, 1), (-1, 1, 1))
    ## 큰 육면체의 꼭짓점
    vertices2 = ((2, -2, -2), (2, 2, -2),
                (-2, 2, -2), (-2, -2, -2),
                (2, -2, 2), (2, 2, 2),
                (-2, -2, 2), (-2, 2, 2))

    ## 작은 육면체의 면을 칠할 색
    ### 문제: 4개의 튜플을 모두 같게 하면 두 육면체의 모서리까지 모두 같은색이 되는 이유?
    colors = ((1, 1, 0),
              (1, 1, 0),
              (1, 1, 0),
              (1, 1, 1))

    ## 각 면(꼭짓점끼리의 연결)
    surfaces = ((0, 1, 2, 3),
                (3, 2, 7, 6),
                (6, 7, 5, 4),
                (4, 5, 1, 0),
                (1, 5, 7, 2),
                (4, 0, 3, 6))

    ## 각 모서리(꼭짓점끼리의 연결)
    edges = ((0, 1), (0, 3), (0, 4),
            (2, 1), (2, 3), (2, 7),
            (6, 3), (6, 4), (6, 7),
            (5, 1), (5, 4), (5, 7))



    # 기본 세팅
    pygame.init()
    display = (800, 600)
    pygame.display.set_mode(display, DOUBLEBUF | OPENGL)

    gluPerspective(45, (display[0]/display[1]), 0.1, 50.0)  # 투영 양식
    glTranslatef(2, -3, -15)      # 바라보는 위치(상하, 좌우, 전후)



    # 육면체를 그림
    ## 작은 육면체
    ### 모서리 그리기
    glBegin(GL_LINES)       # OpenGL에게 직선을 그릴 것이라는 것을 알려줌
    for edge in edges:      # 각 꼭짓점을 직선으로 연결
        for vertex in edge:
            glVertex3fv(vertices[vertex])
    glEnd()

    ### 면 그리기
    glBegin(GL_QUADS)       # OpenGL에게 면을 그릴 것이라는 것을 알려줌
    for surface in surfaces:
        x = 0
        for vertex in surface:
            glColor3fv(colors[x])
            glVertex3fv(vertices[vertex])
            x += 1
    glEnd()


    ## 큰 육면체
    ### 모서리 그리기
    glBegin(GL_LINES)       # OpenGL에게 직선을 그릴 것이라는 것을 알려줌
    for edge in edges:      # 각 꼭짓점을 직선으로 연결
        for vertex in edge:
            glVertex3fv(vertices2[vertex])
    glEnd()                 # OpenGL에게 작업이 끝났음을 알려줌



    # 화면 표시
    pygame.display.flip()       # 화면에 도형을 보여줌
    time.sleep(1)       # 1초 기다림



    # 자원 해제
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)  # OpenGL에 쓰인 버퍼를 비움
    ```
---
## 2020.08.08.
- **kernel 수정관련**
  - kernel 중 cp210 과 USB0 관련 부분은 이미 다 체크되어 있었음
  - rplidar를 따로 연결했을 때 dev/tty*로 변경되는 사항이 없음
  - 연결 커넥터 문제는 아니였음
  - ping은 문제가 없음
  - dmesg를 했을 때 rplidar를 뺐다가 켜는 경우에는 USB0 connected 및 CP210x converted 가 확인되지만 여전히 dev/ttyUSB* 로는 확인되는 것이 없음
  - 현재 우선은 젯슨에서 영상을 받을 만한 것이 없으므로 binary data를 라즈베리 파이에 전달할 방법을 찾는 것이 우선

- **본체**
  - 젯슨을 굳이 영상을 라즈베리에 넘길 필요성 없어짐 (아이디어 필요함)
  - 나중에 시각화 할 때를 위해서라고 데이터 어떻게 파이에 넘길지 필요
  - catkin_make 에러 문의함 : 에러 해결사항 확인 후 수정 필요 
  - 다음주 할일:
   1. 영상 / yolo 아이디어 조금 더 생각 
   2. catkin 수정하고 기본 예제 돌려서 팔 구동 확인 
   3. 터틀봇 원하는 좌표 지정해서 그 위치에 가도록 구동

- **Loading Algorithm**
  - C로 짜고 있던 코드 Python으로 옮기는 작업 진행

- **Truck Visualization**
  - Jetson에 PyOpenGL과 pygame 모듈 설치
  - pygame 설치 오류 - 해결
    - 해결 이유 정확하지 않음, dependency 관련 문제로 추측
    - https://www.pygame.org/wiki/CompileUbuntu?parent=
    - `#install dependencies` 부분 참고
  - 아래 코드 실행 시 오류
    ```py
    pygame.display.set_mode(display, DOUBLEBUF | OPENGL)
    ```
    - 오류 내용
      ```
      Fatal Python error: (pygame parachute) Segmentation Fault

      Current thread 0x0000007fa36ff000 (most recent call first):
        File "test.py", line 45 in <module>
      Aborted (core dumped)
      ```
    - OPENGL 플래그에서 문제 발생, pygame창에 OpenGL 툴을 불러오지 못함, 원인 모름
    
---
## 2020.08.12.
- **매니퓰레이터 문의**
  - 답변 준대로 했는데 똑같은 에러남

---
## 2020.08.13.
- **Truck Visualization**
  - Jetson에 GPU 전용 VRAM이 없어서 메모리 초과 의심 → 아닌듯함
    - GPU 상태를 모니터링하는 툴을 설치하여 코드를 돌려 보았으나 특이한 변화 없음
      - https://www.jetsonhacks.com/2018/05/29/gpu-activity-monitor-nvidia-jetson-tx-dev-kit/
      - https://eungbean.github.io/2018/08/23/gpu-monitoring-tool-ubuntu/

---
## 2020.08.14.
- **Truck Visualization**
  - OpenGL과 pygame 이용하지 않는 방법 시도
  - matplotlib 이용 → Ubuntu에서 실행 되는지 확인 필요
  - 참고
    - https://codereview.stackovernet.com/ko/q/38653
    - https://stackoverflow.com/questions/18853563/how-can-i-paint-the-faces-of-a-cube
    - https://matplotlib.org/3.1.0/gallery/color/named_colors.html
  - 수정하여 작성한 테스트용 코드
    ```py
    import matplotlib.pyplot as plt
    import numpy as np
    from itertools import product
    from matplotlib.patches import Rectangle
    import mpl_toolkits.mplot3d.art3d as art3d


    fig = plt.figure()
    ax = fig.gca(projection='3d')
    ax.set_aspect('equal')


    # draw cube
    def rect_prism(x_range, y_range, z_range):

          yy, zz = np.meshgrid(y_range, z_range)
          ax.plot_wireframe(x_range[0], yy, zz, color="black")
          ax.plot_wireframe(x_range[1], yy, zz, color="black")

          xx, zz = np.meshgrid(x_range, z_range)
          ax.plot_wireframe(xx, y_range[0], zz, color="black")
          ax.plot_wireframe(xx, y_range[1], zz, color="black")


    rect_prism(np.array([0, 45]), np.array([0, 20]), np.array([0, 20]))

    colors = 'gold'
    for i, (z, zdir) in enumerate(product([0, 2], ['x', 'y', 'z'])):
        side = Rectangle((0, 0), 2, 2, fill=False, edgecolor=colors)    # https://www.delftstack.com/ko/howto/matplotlib/how-to-draw-rectangle-on-image-in-matplotlib/
        ax.add_patch(side)
        art3d.pathpatch_2d_to_3d(side, z=z, zdir=zdir)

    plt.show()
    ```
---
## 2020.08.15.
- **Manipulator**
  - catkin_make는 성공
  - https://emanual.robotis.com/docs/en/platform/turtlebot3/manipulation/
  - https://answers.ros.org/question/254084/gazebo-could-not-load-controller-jointtrajectorycontroller-does-not-exist-mastering-ros-chapter-10/ 설치했음
  - https://emanual.robotis.com/docs/en/platform/turtlebot3/manipulation/ 에서 현재 인식되는 오픈 매니퓰레이터가 실제로 움직이는 것 빼고는 다 구동함 ( 
  https://www.youtube.com/watch?v=wmZQoTdtioY : lidar

-**Box Loading**
  - 적재 알고리즘 개발 진행
  
#### 할 일 
  - 매니퓰레이터 실제 구동 / 카메라 달기
  - 박스 3D 프린트(파랑, 초록, 보라 등의 색으로) 후 OpenCV 코드 테스트
  - 적재 알고리즘 개발
  - Jetson, RPi 간 데이터 송수신 방법 모색
    - python 코드 상에서 정보를 실시간으로 서버 또는 유선으로 송수신
  - Jetson 백업 방법 모색
  - 카메라 포커싱 문제 해결
    - 오토포커싱 카메라 찾아보기
    
---
## 2020.08.17.
- **Box Loading**
  - 주석 작성
  - 알고리즘 개발 진행
  
---
## 2020.08.18.
- **Box Detection**
  - 배경색과 상반되는 색의 상자를 사용해야 인식이 잘 됨
  - 상자를 인식하여 생기는 빨간색 사각형의 좌표를 추출하여 피타고라스 정리를 이용해 화면에서 상자가 차지하는 픽셀 크기를 구할 예정

---
## 2020.08.20.
- **Box Detection and Measuring**
  - 알고리즘 개발 진행
    - 피타고라스 정리를 이용하여 상자의 픽셀 크기를 측정하는 코드 추가
    - 스페이스바를 누르면 측정하도록 함
---
## 2020.08.22.
- **data sending**
  - 만약 젯슨 보드에서 데이터 디스플레이를 진행하게 될 경우 버거울 수 있음
  - 결론적으로 데이터를 보낼 필요가 있음
  - 현재 테스트를 해본 방식은 다음과 같음:
   1. 리눅스 노트북(코어 진행하는 노트북) 에서 코어 틀어놓음
   2. 라즈베리파이에서 퍼블리셔 틀기
   3. 리눅스 노트북에서 서브스크라이버 틀기
   
   - 참고: https://htsstory.tistory.com/entry/ROS-python%ED%8C%8C%EC%9D%B4%EC%8D%AC-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-1-%ED%86%A0%ED%94%BD-%EB%A9%94%EC%8B%9C%EC%A7%80-%ED%86%B5%EC%8B%A0?category=282702
   - 젯슨도 똑같은 코어 사용하기 때문에 그에 대한 데이터 받아오기 
   
   - 받아올 데이터: 카메라 박스 좌표, 카메라 적재 데이터
   - 카메라 영상정보도 한번 비슷하게 해보기
- **Loading Box**
  - 알고리즘 개발 진행
    - 적재할 수 있는 상자의 조건 추가(아래가 비어있지 않을 때, 적재했을 때 높이가 트럭 높이를 넘지 않을 때)

---
## 2020.08.25.
- **Loading Box**
  - 알고리즘 개발 진행

---
## 2020.08.27.
- **open manipulator**
  - 패키지를 따로 분석해야할 듯 함 (그리퍼의 범위를 봐서 길이별로 데이터 보내기, 각도제어도 코드로 보낼 방법 생각하기)
  - 일단 gui 쓸 수 있음 -> 자세가 이상하게 측정되서 그에 대한 해결 필요
  - 움직이는 것 확인 후 패키지에 대입해보기 / 매니퓰레이터에 카메라 어떤식으로 할지 고민하기
- **Loading Box**
  - 알고리즘 개발 진행
    - 5x5 상자의 sample input으로 수행 성공
  - 트럭 높이 제한에 대한 코드 추가 필요
  - random input으로 테스트 필요
  
---
## 2020.08.28.
- **open manipulator**
  - 문제가 되었던 부분은 오픈매니퓰레이터의 혼이 조정이 안되었던 것이여서 현재는 조작에 큰 문제가 없음
  - 계획: 입구에서 상자를 받아서 지역별로 전달 (상자 받는 것은 마커로, 전달은 코드로) -> 정렬 후 읽어서 박스를 트럭에 전달
    - 1. ar마커 적용해보기 (캠 설치 및 코드 실행) / 관련 코드보고 원리 확인 (만약 바코드와 통일할 수 있으면 통일하기)
    - 2. 마커를 읽어서 특정 위치로 보내도록 코드 생성
    - 3. 재읽기 코드 생성
- **Loading Box**
  - 알고리즘 개발 진행
    - 트럭 높이, 길이 제한에 대한 코드 추가 필요(시도하다가 끝남)
- **Barcode Scanning**
  - tegra 관련 코드 추가 없이도 외장 USB카메라 작동함

---
## 2020.08.30
<details>
<summary><span style="color:green">📝ar 마커 관련 정리</span></summary>
  
```
참고 사이트: https://github.com/greattoe/ros_tutorial_kr/blob/master/rospy/ar_1_ar_track_alvar.md
  1. 마커에는 tf와 pose가 있다.
  2. 기본 launch 내용에는 marker 한변의 길이를 넣어준다 (나머지는 default로 사용)
  3. 마커는 정사각형이여야함
  4. 마커의 정보: header + markers -> 자신이 몇번 마커인지, `position.x,y,z`, 축

기본 `home service challenge`: 각각의 publish가 필요함 / scenario data를 저장해줄 때 이름, marker의 이름, position등을 저장해줌 (map에 대한 기본적인 정보가 필요하다 생각이 든다)-> 기본 simulator에 대한 이해 진행 후 현재 가지고 있는 맵에 맞도록 구성 짜보기
- http://wiki.ros.org/ar_track_alvar

- ROS에서 제공하는 기본 마커 파일: http://wiki.ros.org/ar_track_alvar?action=AttachFile&do=view&target=markers0to8.png

- 기본 구동 예제: https://www.youtube.com/watch?v=sV7vOTvUCx8

- 마커 생성: https://webnautes.tistory.com/1040
```

</details>

- **Loading Box**
  - 알고리즘 개발 진행
    - 트럭 높이, 길이 제한에 대한 코드 추가 완료

---
## 2020.09.03.
- **Loading Box and Truck Visualization**
  - 적재 알고리즘과 matplotlib를 활용한 시각화 코드 병합
  - 최적화 필요
    - 뒤로 갈수록 속도가 느려짐 → plot할 것이 점점 많아져서 그런 것으로 보임
    - for문을 벡터연산으로 바꾸어 속도 향상 필요

---
## 2020.09.06.
- **BoxDetection and SizeMeasuring + BarcodeScanning**
  - 상자인식&크기측정 코드와 바코드스캔 코드 병합
    - 상자 영역의 x축 방향 중심점이 화면의 중앙(315~325)에 올 때 크기를 측정하고 바코드를 스캔하도록 함
    
- **카메라 구매**
  - 기존에 쓰던 'Logitech C270' 모델이 오토포커스 기능을 지원하지 않아 '앱코 APC930 FHD' 구매

---
## 2020.09.09.
- **BoxDetection & SizeMeasuring & BarcodeScanning + LoadingBox & TruckVisualization**
  - 모든 코드 병합(초안): MergeAll.py
    - 상자가 모두 지나간 후 카메라 캡처를 종료하는 조건 필요
      - time.time() 활용?
    - 실제 환경을 구성하여 상자의 픽셀 크기와 실제 크기의 비율 적용 필요
    - Jetson에서 구동 가능한지 확인 필요

--- 
## 2020.09.10.
- 바코드 프린트한 것 테스트
  - 3pixel로 제작했더니 인식이 되지 않음 → 1pixel로 다시 제작 필요
  
- **중간 정리**
  - 진행계획
    1. 로봇팔
      - 최대 목표: AR마커를 이용하여 상자를 지역별로 분류
        - ssh문제로 로보티즈에 문의 중
        - 현재 문제를 해결한다면 Pick&Place 패키지를 수정하여(상자에 붙어있는 마커를 인식하여 pick하도록) 진행
      - 최소 목표: GUI 패키지를 해석하여 레일 위 상자의 바코드를 직접 인식 또는 젯슨으로부터 데이터를 수신하여 특정 위치로 미는 것 까지만 진행
        - 이 경우 뒤의 시나리오도 수행하지 못함(상자를 다시 찾아서 트럭 앞까지 가져다 주는 것)
    2. 병합한 핵심 코드
      - 바코드 1pixel짜리로 다시 제작하기
    3. 시나리오 수행 환경
      - 상자 3D 프린팅
      - USB카메라 고정할 방법 찾기: 삼각대, 절연테이프
      - 바닥: 검정색 전지
      - 트럭: 우드락, 우드락 본드
      - (경사면: 아크릴)
  - 일정
    - ~ 9/12
      - 미경: 바코드 제작, BSB 코드 테스트(화면 픽셀값 해결, 해상도 조절)
      - 민지: 로보티즈 문의, C++ 공부, GUI 뜯어보기
    - ~ 9/13
      - 수행환경 만들기
      - 민지: 로보티즈 답변 적용(안되면 GUI이용 코드 제작)
      - 미경: 데이터 송수신 공부 및 적용, 바코드 적용하여 최종 병합 코드 테스트
    - ~ 9/27 모든 구현 완료

---
## 2020.09.12.
- **BSB 코드 테스트 및 수정**
  - 카메라의 pixel이 Jetson에서 2592\*1944로 인식되어 frame의 너비를 640, 높이를 480으로 설정하는 코드 추가
  - 카메라가 읽어오는 화면이 변화할 때마다 반응 속도가 느리고 꿀렁거림이 발생하여 FPS(프레임 속도)를 30으로 설정하는 코드 추가
    - 참고: https://076923.github.io/posts/Python-opencv-4/
  - 카메라 제조사에서 제공하는 프로그램을 이용하여 화면 밝기 자동 보정 기능을 해제함(속도 저하의 원인이 될 수 있기 때문)

---
## 2020.09.13.
- **BSB**
  - 상자의 실제 크기를 계산하는 코드 추가(실제:픽셀=1:40.8)
  
- 시나리오 수행 환경 구성
  - 폼보드로 트럭의 3면 표현
  - 컨베이어벨트의 빛반사, 색혼동 방지를 위한 절연테이프 부착
  - 절연테이프를 이용하여 삼각대와 카메라 고정
  
- 계획
  - ~9/14 
    - 민지: 젯슨 ssh, debugging(값 받아서 돌리는것), ar marker 질문
  - ~9/16
    - 미경: 바코드 부착 후 최종 코드 확인, 모든 상자 인식 후 종료조건 아이디어 생각 및 적용
    - 민지: AR마커 로보티즈 문의, C++ 공부, GUI 뜯어보기 / 치는 것 완료하기
  - 9/17
    - 미경: 데이터 송수신 공부 및 적용
---
## 2020.09.14
- **manipulator**
  - 1차적으로 큰 틀을 짰음 - 빌드는 됨
  - 내일 할일:
    - 다른 클래스 (home_pose_clicked 등을 통하여 행동 지침 코드 짜기)
    - ar marker 답이 없으므로 새로 질문 올리기
---
---
## 2020.09.15
- **manipulator**
  - gui 버튼을 누른것처럼 조정할 수 있는 방법?
  - 목요일에 각 함수들에서 value() 가 갈 때 어떠한 값이 가는지 확인을 해보기 ( 만약 main_window.cpp을 사용 불가능 하다면 qnode.cpp에 직접 value 값을 지정해주는 방식을 사용해보는 것으로 생각하기)
  - 로스에서 받은 데이터 바로 처리하도록 하는 방식 

---
## 2020.09.16.
- **Merged Code**
  - 상자에 바코드 부착하여 테스트
    - 바코드 index는 0부터 시작하여 빈 숫자가 없어야 실행 가능
    - 개수 무관
    - A,B,C 순서 무관

---
## 2020.09.17.
-**roscore**
  - 끌 때 killall -9 roscore, killall -9 rosmaster
- **gui code**
  - 코드 일부 완성
  - 함수만 이제 변경해서 내려가서 치고 돌아오는 형식으로 바꾸기
  - 해야할일:
    - 코드 정리 주소지 main.cpp
    - 주소지 데이터 받고 계속 돌리는 걸로 코드 바꾸기
    - 통신 젯슨에서 해보기
    - port 22

---
## 2020.09.20
- **Jetson SSH**
  - 해결  https://rtime.felk.cvut.cz/hw/index.php/NVIDIA_Jetson_TX2
  
-**현재 상황**
  - roscore와 연결이 되지 않아서 다른 노트북으로 진행해야함
  - 22 결국 해결 안됨 - 치는것으로 마무리해야함
- **open_manipulator**
  - 1차적으로 코드 완성함
    - 해야할 일:
       - 박스 담는 경사면 위치 확실하게 하고 터틀봇 위치 확실하게 잡기
       - 각 주소지 별로 쳐야하는 위치 숫자 지정해서 행동 데이터 완성하기
       - 통신으로 데이터 받아서 그에 따라 행동하고 종료하도록 하기

- **다음주 마무리**
  - 9.21: 환경 세팅 확실하게 마무리하기, 정보 확실하게 저장하기 ( 팔 움직이는 값), 구동 진행해서 확인하기, 통신 확실하게 하기(해결책) // `종료조건`
  - 9.22~23: 통신과 관련해서 코드 다시 수정하기
  - 9.24: 모든 것 합쳐서 진행하고 이상한 것 수정 및 추가적으로 필요한 것 찾아서 넣기
  - 9.27: 동영상 촬영

---
## 2020.09.21
- **ssh로 matplotlib 띄우기**
  - 환경에 pyzbar/opencv 깔려 있어야함
  - 로스에서 충돌나기 때문에 import sys -> sys.path.remove('/opt/ros/kinetic/lib/python2.7/dist-packages')
  - 결국에 모니터 연결해서 하는 것을 확정
- **통신**
  - 통신 자체가 단방향이여서 타이밍 문제가 크기 때문에 카메라 하나를 더 추가해서 roslaunch하는 부분에서 실행하기로 함
- **전반적인 형태**
  - launch 메인 코드에 CPP pyzbar를 사용하기 (cpp pyzbar 및 opencv 다운받아야함)
  - 메인코드에서 카메라에서 박스 인식해서 주소지 인식하면 n에 주소지 형태 저장하고 저장이 되면 행동을 실행하도록 해야함 (행동을 하는 while문에서 마지막에 n을 초기화해줘야함)

- `현재 문제`
  - 원래 노트북으로 돌아오고 나서 gui를 여는데 문제가 있음 (다른 opencr로도 실행해볼 예정이지만 수정 필요)
  
- **일정**
  - pyzbar: 화요일에 찾아봐서 cpp 실행방법 알고 수요일에 launch 메인코드에 넣어보기
    - https://stackoverflow.com/questions/56068886/how-to-configure-c-zbar-scanner-to-decode-only-qr-code-data-type
  - open-manipulator: 화요일에 에러나는 부분 수정, 수요일에 위치 확실하게 고정하기, 목요일까지 pyzbar와 합쳐서 메인코드 완성
  - 통신: 화요일에 talker.py-> listener.cpp 테스트 해보고 안되면 메일보내서 질문하기
  - 종료 조건: 금요일에 적재 종료 및 manipulator 종료 코드 확실하게 하기 (manipulator는 없어도 상관은 없음)
  - 목요일 및 금요일에 환경 고정 및 감도 조절, 테스트
  - 토요일까지 코드 수정 및 확실하게 하고 일요일에 영상 찍기
  
---
## 2020.09.22
  - **에러관련**
    - 1차 해결방안으로 ntpdate로 싱크 맞추는 것을 알려줌 - 안된다
    - 내일 해볼 방법: 새로 라즈비안 받아서 패키지 다운받고 실행해보기
    ```
    $ sudo apt-get install ntpdate
    $ sudo ntpdate ntp.ubuntu.com
    ```
    - https://yjs-program.tistory.com/42 이거 해줌 
 
  - **통신**
    - cpp로 해보는 것 실패함
    - 데이터 확장 찾아봄 - https://eunguru.tistory.com/84

  - **C++ 바코드 인식**
    - Window에서 코드 작성 및 테스트: VS에서 안되는 것으로 보임
      - zbar 설치(http://zbar.sourceforge.net/download.html → https://sourceforge.net/projects/zbar/)
      - 프로젝트에 라이브러리 추가(https://blessingdev.wordpress.com/2017/09/26/visual-studio%EC%97%90-%EC%99%B8%EB%B6%80-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC-%EC%B6%94%EA%B0%80%ED%95%98%EA%B8%B0/)
    - G++로 컴파일 시도해봐야 함
      - 코드 참고
        - https://www.learnopencv.com/barcode-and-qr-code-scanner-using-zbar-and-opencv/
        - https://stackoverflow.com/questions/56068886/how-to-configure-c-zbar-scanner-to-decode-only-qr-code-data-type
        - http://blog.naver.com/PostView.nhn?blogId=jdkim2004&logNo=221125562861&parentCategoryNo=27&categoryNo=&viewDate=&isShowPopularPosts=true&from=search
        - https://github.com/ayoungprogrammer/WebcamCodeScanner/blob/master/main.cpp
        
---
## 2020.09.23
  - **에러관련**
    - 결국에 이미지 새로 구워서 다시 진행하니까 해결됨 / 결론적으로 이유는 찾지 못함
    
  - **위치확정**
    - 테스트로 위치 확정 (목)
    - 이후 옮겨서 안에서 계속 해보는게 제일 확실
  
  - **로봇에서 데이터 받기**
     - 수요일부터 목요일까지 시도하기
     1. 노트북에 카메라 연결해서 zbar/opencv install하여 데이터 그대로 cpp로 받기 - 현재 버전으로 안될 것 같음
     2. 원래 카메라에서 데이터 받아서 파이썬 투 씨쁠쁠로 데이터 변경하기
     3. publisher.py -> subscriber.cpp 이거로 하기로 결정
     - cpp 참고: https://pinkwink.kr/900#google_vignette
     - pub.cpp -> sub.py 참고: https://youtu.be/dQLIbEdUCRM
     
  
---
## 2020.09.24
  - **매니퓰레이터**
    - 콜백함수여서 키보드 인터럽트가 필요함 - c++에서 `키보드 플래그` 찾아서 넣기
    - 경사면은 그대로 고정하기
  - **cv 조건문**
    - g++ 로 빌드 할 때 'g++ -o cv cv.cpp `pkg-config opencv --cflags --libs`' 로 하는데 cv c++코드를 main에 넣게 되면 catkin_make 빌드가 안됨. path 관련 문제 해결이 필요
  - **ros data**
    - 일단 subscriber자체로 돌리면 데이터가 들어오지만 main에서는 들어오지 않음 (확인 프린트 넣었으나 전혀 없음)
    - 통신이 가장 시급 
  
  - **현재 상황 총정리**
    - 통신으로 데이터가 오지 않아서 while문에서 ros msg 받는거에 대한 수정이 필요할 것으로 보임
    - 일단은 키보드 입력으로 실험을 해봤는데 콜백함수여서 무조건 sleep도 안먹고 키보드 입력이 필요함 - 키보드 플래그로 나중에 변경해야함 (일단은 무조건 데이터 넣어줘야함)
    - cv가 g++로 빌드는 되지만 catkin에서는 명령어가 또 달라서 그에 대한 해결책 필요. (안되면 sleep(5)로 일단 함수 넣었는데 그거로 진행하는 수 밖에 없음)
  - **추가사항 - 멀티쓰레드**
    - c++에서 fork함수로 진행하면 데이터값 실시간으로 받을 수 있을 것이라 예상
    - 하지만 확인 필요

---
## 2020.09.25
  - **Ros Data Communication**
    1. spin()은 데이터를 받으면 main으로 돌아가지 않음(다른 코드를 수행하지 않음), 즉 계속해서 데이터를 받는 동작만 수행함. 기존의 tutorial 실행 시에는 다른 동작이 없고 오로지 데이터를 받기만 하면 됐으므로 상관없지만 우리 코드에서는 while을 돌며 다른 동작을 수행해야 함 → **spin() 대신 spinOnce()와 rate()를 사용하여 해결**(printf로 의미없는 문자를 출력해봤을 때 문제없이 동작함)
		- spin(), spinOnce() 참고
			- https://programming.vip/docs/ros-ros-spin-and-ros-spinonce-differences-and-use.html
			- https://answers.ros.org/question/357705/stop-rosspin/
			- https://answers.ros.org/question/11887/significance-of-rosspinonce/
		- thread 관련(spin 구글링 중 나온 자료)
			- https://yuzhangbit.github.io/tools/several-ways-of-writing-a-ros-node/
			- http://wiki.ros.org/roscpp/Overview/Callbacks%20and%20Spinning
    2. char c[4]와 float c[4]의 변수명이 중복되어 `char c[4] → char curr[4]`, `char d[4] → char prev[4]`로 변경, "XXX"로 문자열 초기화
    3. Queue에 push하는 부분 수정 → 동작 수행 성공
		- strcmp, memcpy 사용
    - 데이터를 받아 queue에 push하는 것까지 동작 확인하였으나 지역별 로봇팔 수행 부분(if문)을 주석해제할 시 데이터를 받지 못하고(chatterCallback 수행 X), 에러도 나지 않는 상태로 유지됨
	```cpp
	#include <stdio.h>
	#include <stdlib.h>
	#include <QtGui>
	#include <QApplication>
	#include "../include/turtlebot3_manipulation_gui/main_window.hpp"
	#include "ros/ros.h"
	#include "std_msgs/String.h"
	#include <iostream>
	#include <queue>
	#include <string.h>
	//#include <opencv2/opencv.hpp>


	char curr[4]="XXX";
	void chatterCallback(const std_msgs::String::ConstPtr& msg)
	{
	  ROS_INFO("I heard: [%s]", msg->data.c_str());
	  memcpy(curr, msg->data.c_str(), sizeof(msg->data.c_str()));
	  printf("%c\n", curr[0]);
	}



	int main(int argc, char **argv) {
	    QApplication app(argc, argv);
	    turtlebot3_manipulation_gui::MainWindow w(argc,argv);

	/*
	    // cv
	    Mat img_color, img_blurred, img_gray;
	    Rect rect;
	    VideoCapture cap(1);
	*/



	    //주소지 따른 행동 
	    float a[4] = {0.950,0.200, 0.400, 0.500}; // 주소지 a가 행동할 방향 
	    float b[4] = {0.700,0.250, 0.000, 0.800};; // 주소지 b가 행동할 방향 
	    float c[4] = {0.950, 0.650, -0.300, 0.300}; // 주소지 c가 행동할 방향 
	    // 치기 
	    float a_1[4] = {0.950,0.400, 0.400, -0.350}; // a 박스 전달 
	    float b_1[4] = {0.65, 0.550, -0.250, 0.400}; // b 박스 전달 
	    float c_1[4] = {0.600, 0.650, -0.300, 0.300}; // c 박스 전달 

	    // 통신 init
	    ros::init(argc, argv, "listener");
	    ros::NodeHandle n;

	    using namespace std;
	    //using namespace cv;
	    queue<char> q; // data 전달받을 주소지 
	    char prev[4]="XXX";
	    char A = 'A';
	    char B = 'B';
	    char C = 'C';

	    // application 보여주고 거기서 실행 log 보여주기 
	    w.show();
	    w.on_btn_timer_start_clicked(); // 처음에 Qtimer 시작해야하기 때문에 클릭 
	    std::cout << " timer clicked ";
	    //w.on_btn_gripper_close_clicked(); // start with gripper closed
	    std::cout << " gripper clossed ";
	    app.connect(&app, SIGNAL(lastWindowClosed()), &app, SLOT(quit()));

	    while(1){
			/*
			cap.read(img_color);
			if (img_color.empty()) {
				break;
			}
			GaussianBlur(img_color, img_blurred, Size(5, 5), 0, 0, 4);
			cvtColor(img_blurred, img_gray, COLOR_BGR2GRAY);
			threshold(img_gray, img_gray, 100, 255, THRESH_BINARY);
			vector<vector<Point> > contours; // Vector for storing contour
			vector<Vec4i> hierarchy;

			double area;
			double max_area = 0;
			int max_index = -1;
			findContours(img_gray, contours, hierarchy, CV_RETR_CCOMP, CV_CHAIN_APPROX_SIMPLE); // Find the contours in the image

			for (int i = 0; i < contours.size(); i++) {
				area = contourArea(contours[i], false);
				if (area > max_area) {
					max_area = area;
					max_index = i;
					rect = boundingRect(contours[i]);
				}
			}
			drawContours(img_color, contours, max_index, Scalar(0, 255, 0), 2, 8, hierarchy);
			rectangle(img_color, rect, Scalar(0, 0, 255), 2, 8, 0);
			*/

			// communication start 
			ros::Subscriber sub = n.subscribe("chatter", 10, chatterCallback);
			ros::Rate loop_rate(1);


			if (strcmp(prev, curr)!=0){
			    memcpy(prev, curr, sizeof(prev));
			    q.push(curr[0]);
			    printf("success to push\n");
			}


			//std::cout << "input:";
			//std::cin >> prev;
			//q.push(prev[0]);

			/*
			if(q.front()==A){
			    //sleep(5);
			    std::cout << "A start\n";
			    w.on_btn_send_joint_angle_clicked(a[0], a[1], a[2], a[3]);
			    std::cin >> prev[0]; // 버퍼가 필요 - 다른 아이디어 있으면 대체 
			    std::cout << " buffer 1\n ";
			    w.on_btn_send_joint_angle_clicked(a_1[0], a_1[1], a_1[2], a_1[3]);
			    std::cin >> prev[0]; // 버퍼가 필요 - 다른 아이디어 있으면 대체 
			    std::cout << " buffer 2\n ";
			    w.on_btn_init_pose_clicked();
			    std::cout << "a\n";
			    std::cin >> prev[0];
			    q.pop();
			}

			else if(q.front()==B){
			    std::cout << "B start\n";
			    w.on_btn_send_joint_angle_clicked(b[0], b[1], b[2], b[3]);
			    std::cin >> prev[0]; // 버퍼가 필요 - 다른 아이디어 있으면 대체 
			    std::cout << " buffer 1\n ";
			    w.on_btn_send_joint_angle_clicked(b_1[0], b_1[1], b_1[2], b_1[3]);
			    std::cin >> prev[0]; // 버퍼가 필요 - 다른 아이디어 있으면 대체 
			    std::cout << " buffer 2\n ";
			    w.on_btn_init_pose_clicked();
			    std::cout << "b\n";
			    std::cin >> prev[0];
			    q.pop();
			    std::cout << " pop\n";
			}

			else if(q.front()==C){
			    std::cout << "C start\n";
			    w.on_btn_send_joint_angle_clicked(c[0], c[1], c[2], c[3]);
			    std::cin >> prev[0]; // 버퍼가 필요 - 다른 아이디어 있으면 대체 
			    std::cout << " buffer 1\n ";
			    w.on_btn_send_joint_angle_clicked(c_1[0], c_1[1], c_1[2], c_1[3]);
			    std::cin >> prev[0]; // 버퍼가 필요 - 다른 아이디어 있으면 대체 
			    std::cout << " buffer 2\n ";
			    w.on_btn_init_pose_clicked();
			    std::cout << "c\n";
			    std::cin >> prev[0];
			    q.pop();
			    std::cout << " pop\n";
			}

			else{ // data sending 종료 시  
			    continue;
			}

	*/

			//int result = app.exec();
			//return result;
			loop_rate.sleep();
			ros::spinOnce();
			printf("after spinOnce()\n");
	    }
	    int result = app.exec();
	    return result;
	}
	```

---
## 2020.11.10.
  - **대회들 준비**
    - 임베디드 sw 경진대회 결선 준비 - 코드 일부 업그레이드 
    - 창의설계 및 sw경진대회 시연 준비

---
## 2020.11.16.
  - **에러 발생(No matching signal)**
    - 에러 메세지
      ```
      QMetaObject::connectSlotsByName: No matching signal for 'on_btn_read_joint_angle_clicked'
      ```
	
---
## 2020.11.19.
  - **에러 해결(No matching signal)**
    - 함수 이름의 `on_`을 삭제하고 실행함(btn_read_joint_angle_clicked)
        - 참고 링크: https://m.blog.naver.com/cherrypeanut/220978305061
    
---
## 2020.11.20.
  - **에러 발생**
    - 이름 변경한 함수(btn_read_joint_angle_clicked)가 둘 중 첫번째만 실행되고 에러가 발생함
    - 에러 메세지
      ```
      ABORTED: Solution found but controller failed during execution
      ```

---
## 2020.11.23.
  - **에러 해결**
    - 시간 딜레이를 조금 더 늘려주니 문제가 해결 되었다. (usleep 조정 필요)
      ```
      ABORTED: Solution found but controller failed during execution
      ```
  - **카메라 위치 조정**
    - 위치를 끝으로 조정하고 한개로 하기로 변경함 
  
  - 할일
    1. 카메라 위치 맞게 사이즈 조정 (박스) 
    2. 카메라 지지대 프린트하기
    3. 박스 데이터 딜레이 조정
    4. 슬립 조정 및 다이나믹셀 속도 변경 관련 확인
    5. box gripper
    6. 발표 준비
    
---
## 2020.11.26.
  - **dynamixel velocity**
    - http://docs.ros.org/en/kinetic/api/moveit_tutorials/html/doc/time_parameterization/time_parameterization_tutorial.html 참고하여 설정 변경해봤으나 속도 동일
    - 답변에서 내부적으로 설정 고정 가능하다하면 설정 변경할 예정
  - 카메라 위치 변경: 박스 크기 확실히 나옴
  - 할 일
    - velocity (~일요일)
    - 발표 세팅 (대본쓰기 / 웹캠 설치 - 일)
    - 화요일 리허설
    
---
## 2020.11.29.
   - **cotouring error**
     - 문제: 카메라 위치를 변경하면서 팔이 근처를 지나가니까 프로그램이 종료됨
     - 원인: 검출된 컨투어가 없어서 오류 발생
     - 해결: box_detection()에서 예외 처리 추가
     	- max_index가 -1이면 이전 상태의 result와 box를 그대로 return
   
   - **delay**
     - 카메라 위치가 바뀌면서 팔 시작 전에 딜레이가 필요없어서 이를 지움
     - 최적으로 제일 빠른게 2400000이라 이로 조정해둠
     
   - 할 일
     1. 대회 측에 시간 구성 질문 아침에 진행(월)
     2. 로보티즈에서 다이나믹셀 답변 나오면 시도
     3. 발표 자료 준비(월)
     4. 리허설 (화)

---
## 2021.02.23.

### Box Detection using Image Segmentation

1. 기존의 컨투어링이 아닌, 이미지 세그멘테이션 기법을 이용하여 박스를 검출하는 코드로 변경 (rev0)
	- 기존의 반환형과 호환되지 않는 문제 발생 -> 객체를 발견할 수 있으나, 객체의 꼭짓점을 검출해야함

2. 이미지 세그멘테이션 코드와 기존의 컨투어링 코드를 합침 (rev1)
	- 똑같이 불안정한 문제

3. 이미지 세그멘테이션 후 코너를 검출하여 객체 주위에 사각형 그리기(rev2)
	- 코너가 정확하게 꼭짓점에 검출되지 않음
 
4. 이미지 세그멘테이션 후 반복문을 통해 객체 주변의 좌표 중 최댓값, 최솟값을 구하여 꼭짓점 찾기(rev3)
	- 실행 속도가 아주 느려짐

---
## 2021.02.24.

### Box Detection using Image Segmentation

#### 검출한 전경(=박스)의 꼭짓점을 찾고, 그 주위에 윤곽을 그리는 것이 필요

1. 마커가 -1인 부분(경계선)의 최대 좌표, 최소 좌표를 찾아서 꼭짓점 알아내자. -> 실행속도가 느려지지만 일단 시도해보자.
  	- 전경의 외곽선으로 조건을 줬는데도, 최소좌표, 최대좌표를 엉뚱한데서 잡아냄

2. 코너를 검출해서 검출해낸 코너의 최대 좌표, 최소 좌표를 찾아서 꼭짓점 알아내자.
  	- 성공했으나, 파란색과 초록색 박스의 경우 처음부터 전경을 제대로 검출하지 못해 코너 디텍트를 통한 꼭짓점 4개가 이상하게 잡힌다
  		- 전경을 제대로 검출해야함 -> 워터세드+그랩컷? 노이즈 제거? 바이너리 이미지?
        	- 그랩컷을 합칠 경우 효과 별로 없었음, 오히려 이상한 결과
        	- 초기 바이너리 이미지가 파란색 상자나 초록색 상자의 경우 제대로 안 만들어지고 있는 것을 발견
        	- **임계치를 더 낮춰주어 바이너리 이미지를 생성했더니, 파란색과 초록색 상자도 전경을 제대로 검출하는 것을 확인**
 
3. 컨투어링 코드를 완전히 이해하고 서로 합치면 더 나을까?
  	- 시도하지 않았음

#### 다음 할 일

1. 세그멘테이션 코드 다시 확인하고, 정리하기
2. 박스의 높이 측정 코드 작성
3. 박스의 무게 측정 -> 바코드 데이터 읽는 부분 수정 필요
4. 적재 알고리즘 수정?

---
## 2021.02.25

### Box Detection using Image Segmentation

- 링크 참고하여, 이미지 세그멘테이션 더 안정화시켜보기
	- 오프닝과 클로징 함께 사용해보기
	- 커널 크기 늘려주기, 반복횟수 늘려주기

[모폴로지연산](https://webnautes.tistory.com/1257)
[모폴로지연산2](https://076923.github.io/posts/Python-opencv-27/)

### Track Bar

```python
ret, sure_fg = cv2.threshold(result_dist_transform, 0.7*result_dist_transform.max(), 255, cv2.THRESH_BINARY)
# 이 코드 참고하여 전체 화면에서 가장 큰 픽셀 값의 특정 비율로 이진화 진행한다면 트랙바 사용하지 않아도 조도에 따라 조절해줄 수 있지 않을까?
```

### Box Height

- 박스의 높이가 높을수록 화면에서 윗면이 차지하는 비율이 커질 것이라 판단하여 그 비율로 높이를 측정하려 했음
	- 그러면 다른 박스와 높이는 같은데, 윗면의 넓이 자체가 큰 박스가 있다면? 이 생각은 실패

- 컨베이어 벨트에 크기가 고정된 스티커를 붙여놓고, 그 스티커와 윗면의 비율을 따져서 높이 구할 수 있지 않을까?
	- 첫번째 생각과 같은 문제 발생, 스티커라는 것을 추가하는 것도 뭔가.. 좋지 않다

- 박스에는 바코드 스티커가 항상 붙어있고, 이 크기는 항상 일정하다. 그렇다면 바코드 스티커가 화면에서 차지하는 비율을 이용해 높이 측정할 수 있지 않을까?
	- 가장 시도해볼만한 생각
	- 이를 시도하기 위해 바코드 스티커가 화면에서 차지하는 크기 측정 필요 -> 검출한 바코드의 w, h 이용하여 구할 수 있을 듯

### Box Weight

- 바코드에 무게 정보를 더 추가해야한다.
```
기존 바코드: A00 -> 새로운 바코드: A0020 (박스의 무게가 20kg이라는 것을 뜻함)
```

- 기존 코드 수정 필요
```python
        # 박스 실제크기 계산
        box_l = int(round(box_l_pixel / 35, 0)) # 길이
        box_w = int(round(box_w_pixel / 35, 0)) # 너비
        box_h = BOX_H # 높이
        box_k = int(barcode_data[3:5]) # 무게 정보 추가
	    
	if not int(barcode_data[1:3]) in inputBox[ord(barcode_data[0]) - 65]:  # 중복되는 데이터가 없다면
                inputBox[ord(barcode_data[0]) - 65][int(barcode_data[1:3])] = {'l': box_l, 'w': box_w, 'h': box_h, 'k': box_k} # 이 부분에 무게 정보 추가
                NUM_BOX[ord(barcode_data[0]) - 65] += 1
                print('Info of box: ', box_w, box_l, box_h, box_k)  # 상자의 크기와 무게 출력
```

---
## 2021.02.26

### Box Detection using Image Segmentation [완료]

- 모폴리지 연산에서 오프닝과 클로징을 같이 사용하고, 커널 사이즈와 반복 횟수를 늘려주었더니 전경 추출이 훨씬 안정화 됨
```python
# smart_logi_system_jetson_rev2_corner.py
    # 노이즈 제거
    kernel = np.ones((5,5),np.uint8)
    opening = cv2.morphologyEx(bin,cv2.MORPH_OPEN,kernel, iterations = 3) # 초기 바이너리 이미지로부터
    opening = cv2.morphologyEx(bin,cv2.MORPH_CLOSE,kernel, iterations = 3) # 초기 바이너리 이미지로부터
```
![이전](https://pbs.twimg.com/media/EvPjueHVgAcHZ1I?format=jpg&name=medium)
![이후](https://pbs.twimg.com/media/EvPjueHU4AE6T7p?format=jpg&name=medium)

- 전경 추출이 안정화되어 코너 디텍트 방식으로 꼭짓점을 찾는 대신 컨투어링을 사용해봄
	- 기존 코드보다 컨투어링이 안정되어 있으나, 코너 디텍트 방식보다 불안정함
	- 다만, 박스의 개수가 화면에 2개 이상 나올 때는 이 코드를 이용해도 나쁘지 않을 듯
```python
# smart_logi_system_jetson_rev1_contour.py
    # contour 검출
    val, contours, hierarchy = cv2.findContours(water_bin, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)

    # 면적이 가장 작은(=박스) 추출
    min_area = 1000000
    min_index = -1
    index = -1
    for i in contours:
        area = cv2.contourArea(i)
        index = index + 1
        if area < min_area:
            min_area = area
            min_index = index

    if min_index == -1: # 검출된 컨투어가 없으면
        return result, box  # 이전 상태 그대로 반환

    # 원본 이미지에 모든 컨투어 표시
    cv2.drawContours(result, contours, -1, (0, 0, 255), 2)

    # 컨투어를 둘러싸는 가장 작은 사각형 그리기
    cnt = contours[min_index]
    rect = cv2.minAreaRect(cnt)
    box = cv2.boxPoints(rect)
    box = np.int0(box)
    result = cv2.drawContours(result, [box], 0, (0, 255, 0), 2)
    return result, box
```

### Track Bar [완료]

- 아래 코드와 같은 방식으로 이진화 기준(가장 밝은 픽셀 값의 20%) 결정하여 이진화, 따라서 새롭게 업데이트된 코드는 트랙바 사용하지 않음
- 팀원과 논의 후 트랙바를 삭제할지 그대로 둘지 결정 예정
```python
ret, sure_fg = cv2.threshold(result_dist_transform, 0.7*result_dist_transform.max(), 255, cv2.THRESH_BINARY)
```

### Box Weight [완료]

- 바코드에 박스 무게 정보만 추가하여 새롭게 제작한다면, 앞서 수정해놓은 코드로 박스의 무게 정보 바로 담을 수 있을 듯

### Box Height [진행]

- 박스에는 바코드 스티커가 항상 붙어있고, 이 크기는 항상 일정하다. 그렇다면 바코드 스티커가 화면에서 차지하는 비율을 이용해 높이 측정할 수 있지 않을까?
	- 바코드를 검출할 때, 바코드 크기가 정확하게 검출되는 것이 아님
	- 바코드 스티커의 컨투어를 검출하고, 그 면적을 이용해야할 듯하다
		- 가장 작은 면적의 컨투어를 찾아야하지 않을까?

#### 다음 계획

1. 바코드의 면적 계산하여 전체 화면과의 비율로 박스 높이 측정하기
2. 적재 알고리즘 이해하고, 오류 원인 찾아내기
	- 현재 적재 알고리즘 적재 창이 아예 뜨지 않거나, 다음 운송지로 넘어갈 때 박스가 계속해서 적재되지 않는 오류 발생
	- 박스를 번호 순서대로 인식시켜야 적재알고리즘이 제대로 작동하는건가?
3. 높이와 무게를 고려한 적재 알고리즘으로 수정하기

---
## 2021.03.01

### Box Height [진행]

- 박스 디텍션 함수에서 객체가 검출된 이미지를 반환하여 바코드 함수에 주고, 이 이미지에 컨투어링을 진행하여 바코드를 검출
	- 객체가 검출된 이미지를 이진화 후 모폴로지 클로징 연산 적용 *안쪽 검은 바코드는 흰색으로 채워줌
	- 전처리된 이미지에서 가장 작은 컨투어를 찾아 그 면적(=바코드 라벨이 차지하는 면적)을 알아냄
	- 이 면적과 전체 화면의 비율을 구한 뒤 비례 상수를 곱하여 높이를 예측 가능
```python
# 상자의 크기와 바코드 정보를 얻어내는 알고리즘을 수행하는 함수
def get_box_info(img_color, result, box, barcode_data, inputBox, NUM_BOX):
    gray_barcode = cv2.cvtColor(img_color, cv2.COLOR_BGR2GRAY) # 그레이 스케일로 변환

    ''' 바코드 면적 계산 '''
    retval, bin_barcode = cv2.threshold(gray_barcode, 0.4*gray_barcode.max(), 255, cv2.THRESH_BINARY) # 바이너리 이미지 생성
    kernel = np.ones((5,5),np.uint8)    
    opening = cv2.morphologyEx(bin_barcode,cv2.MORPH_CLOSE,kernel, iterations = 3) # 바코드 라벨 컨투어 추출 위해 안쪽은 채워줌
    cv2.imshow('bin_barcode', opening) # 전처리된 바이너리 이미지 확인
    cv2.waitKey()

    # 컨투어 검출
    val, contours, hierarchy = cv2.findContours(opening, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)

    # 면적이 가장 작은 컨투어(= 바코드 라벨) 추출
    min_area = 1000000
    min_index = -1
    index = -1
    for i in contours:
        area = cv2.contourArea(i)
        index = index + 1
        if area < min_area:
            min_area = area
            min_index = index
    
    # 결과 이미지에 바코드 컨투어 표시
    cv2.drawContours(result, contours, min_index, (0, 0, 255), 2)
    cv2.imshow('result_barcode', result) # 바코드 컨투어 표시된 이미지 확인
    print(min_area) # 바코드 면적 확인
    cv2.waitKey()

    ''' 바코드 정보 읽어오기 '''
    cv2.rectangle(result, (310, 0), (330, 480), (255, 0, 0), 1) # 화면 중앙 표시
    center = box[1][0] + (box[3][0] - box[1][0]) / 2  # 상자 중심 X 좌표

    if img_color.shape[1] / 2 - 10 <= center <= img_color.shape[1] / 2 + 10:  # 상자가 화면의 중앙 부근에 왔을 때
        
        decoded = pyzbar.decode(gray_barcode) # 바코드 스캔

        for d in decoded:
            x, y, w, h = d.rect
            cv2.rectangle(result, (x, y), (x + w, y + h), (0, 255, 255), 2) # 삭제 가능? 

            barcode_data = d.data.decode("utf-8")  # 바코드 인식 결과
            barcode_type = d.type  # 바코드 타입

            # 화면에 바코드 정보 띄우기
            text = '%s (%s)' % (barcode_data[0:3], barcode_type)
            cv2.putText(result, text, (x, y), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 255), 2, cv2.LINE_AA)

            # 박스 픽셀크기 측정
            box_l_pixel = round(np.sqrt((box[2][0] - box[1][0]) ** 2 + (box[1][1] - box[2][1]) ** 2), 0)  # 상자의 픽셀 길이(가로)
            box_w_pixel = round(np.sqrt((box[0][0] - box[1][0]) ** 2 + (box[0][1] - box[1][1]) ** 2), 0)  # 상자의 픽셀 너비(세로)

            # 박스 실제크기 계산
            box_l = int(round(box_l_pixel / 35, 0)) # 길이(가로)
            box_w = int(round(box_w_pixel / 35, 0)) # 너비(세로)
            box_h = (min_area/(640*480))*250 # 높이 *바코드 면적과 전체화면의 비율로 계산, 비례상수는 카메라의 높이에 따라 조절 필요
            # box_k = int(barcode_data[3:5]) # 무게

            if not int(barcode_data[1:3]) in inputBox[ord(barcode_data[0]) - 65]:  # 중복되는 데이터가 없다면
                inputBox[ord(barcode_data[0]) - 65][int(barcode_data[1:3])] = {'l': box_l, 'w': box_w, 'h': box_h} # 박스의 정보 담아주기
                NUM_BOX[ord(barcode_data[0]) - 65] += 1 # 박스의 개수 하나 늘려주기
                print('Size of box: ', box_w, box_l, box_h)  # 상자의 정보 출력

    return barcode_data, result
```
![bin_barcode](https://user-images.githubusercontent.com/46590578/109465597-c94a0600-7aab-11eb-8f4d-8b3a1f9a4679.png)
![result_barcode](https://user-images.githubusercontent.com/46590578/109465641-dbc43f80-7aab-11eb-87fa-0e759fbf55b3.png)

- 오류 발견: 바코드 면적을 제대로 컨투어링 하지 못해서 불안정함
	- 바코드에 컨투어 처리를 위한 바이너리 이미지가 제대로 생성이 안된 것이 원인
	- 이진화를 위한 기준을 좀 더 밝은 픽셀이 될 수 있도록 높여주었음
		- 경계선과 바코드 라벨만 남길 수 있게됨

### Box Detection using Image Segmentation [진행]

- 오류1: '박스의 크기가 작은 경우'에 바코드를 객체로 인식함
	- 박스가 아닌 바코드를 전경으로 검출
		- 정확한 박스의 바이너리 이미지를 얻어도, 워터셰드 알고리즘 처리 중 바코드를 객체로 잡아버림
	- 박스를 전경으로 제대로 검출하더라도, 박스 하나가 정확하게 전경으로 검출되는 것이 아니라서 바코드 면적을 구할 때 오류
		- 전경의 외곽선을 검정색으로 바꾸고 최대 컨투어 면적으로 설정하면 좀 나아질까? 

- 오류2: 코너 디텍트가 완벽하지 않아 박스를 제대로 잡지 못하는 문제

#### 다음 계획1

1. 이미지 세그멘테이션 코드 개선
2. 바코드 면적 구하여 박스 높이 구하는 코드 보완

#### 다음 계획2

1. 적재 알고리즘 이해하고, 오류 원인 찾아내기
	- 현재 적재 알고리즘 적재 창이 아예 뜨지 않거나, 다음 운송지로 넘어갈 때 박스가 계속해서 적재되지 않는 오류 발생
	- 박스를 번호 순서대로 인식시켜야 적재 알고리즘이 제대로 작동하는건가?
2. 높이와 무게를 고려한 적재 알고리즘으로 수정하기

---

## 2021.03.02

### 오류1: 박스의 크기가 작은 경우, 박스가 아닌 바코드를 객체로 인식

- 원인: 박스의 크기가 작은 경우에, 워터셰드 알고리즘을 적용하기 위한 재료가 되는 전경을 바코드 라벨로 잡아서 그렇다.
- 해결: 재료가 되는 전경을 Dilation 연산을 통해서 확대시켜줌
```python
    #  확실한 전경 확보
    ret, sure_fg = cv2.threshold(result_dist_transform, 0.7*result_dist_transform.max(),255, cv2.THRESH_BINARY)
    sure_fg = np.uint8(sure_fg)
    sure_fg = cv2.dilate(sure_fg, kernel, iterations = 3)
```

### 오류2: 박스를 제대로 검출하더라도, 바코드의 윤곽을 제대로 잡지 못함

- 원인: 워터셰드 알고리즘을 적용할 때, 외곽선이 흰색으로 그려지면서 이게 바코드 라벨 컨투어링에 영향을 줌
- 해결: 
	- 워터셰드 알고리즘을 적용할 때 그려지는 외곽선의 색을 검정으로 변경
	- 가장 처음의 바이너리 이미지에 적용하는 모폴로지 연산은 클로징 연산만 적용
	- 면적이 가장 큰 컨투어를 검출
```python
    # 전경, 배경에 0 이상의 값, 불명확한 것에 0 -> 이 알고리즘이 불명확한 것을 판단 + 경계선을 -1로
    markers = cv2.watershed(img_color, markers)
    img_color[markers == -1] = [0, 0, 0] # 객체의 외곽부분은 검정색으로
    img_color[markers == 1] = [0, 0, 0] # 배경 부분은 검정색으로, 객체는 원래 색 그대로
```
```python
    # 면적이 가장 큰 컨투어(= 바코드 라벨) 추출
    max_area = 0
    max_index = -1
    index = -1
    for i in contours:
        area = cv2.contourArea(i)
        index = index + 1
        if area > max_area:
            max_area = area
            max_index = index
    
    # 결과 이미지에 바코드 컨투어 표시
    cv2.drawContours(result, contours, max_index, (0, 0, 255), 2)
```

### 오류3: 코너 디텍트로 꼭짓점을 잡는 것에 대한 불안정함

- 원인: 검출하는 코너의 개수가 적어 4개의 꼭짓점을 잡지 않을 때가 있음
- 해결: 검출하는 코너의 개수를 늘려주고, 검출하는 코너 사이의 거리를 더 짧게 설정
```python
corners = cv2.goodFeaturesToTrack(img_gray, 100, 0.01, 5) # 코너를 찾을 이미지, 코너 최대 검출 개수, 코너 강도, 코너 사이의 거리
```

### 오류 해결 결과

![solve_error](https://user-images.githubusercontent.com/46590578/109659649-3b554480-7bab-11eb-9c9d-928d47d0e844.png)

#### 다음 할 일

1. 적재 알고리즘 이해하기
2. 높이와 무게를 고려한 적재 알고리즘으로 수정하기

---

## 2021.03.03

### 트랙바 삭제

- 전체 화면의 픽셀 값을 고려하여 바이너리 이미지를 만들 수 있으므로 트랙바는 삭제
	- dummy, set_window 함수 삭제

### 높이, 무게 수정

- 바코드 라벨 변경 전이므로 무게는 일단 20으로 고정
- 카메라의 높이에 맞는 비례 상수 설정하여 높이가 5로 나올 수 있도록 설정해놓음

### 적재 알고리즘 [진행]

- 적재 알고리즘 코드에 따라 운송지 순서대로 박스를 읽혀야 동작함
	- 오류가 아니라, 코드 자체가 이렇게 짜여 있어서 그런 것
	- 이걸 운송지 순서 상관없이 적재할 수 있도록 변경할 수 있을까?
	- 일단 높이랑 무게 고려하려면 어떻게 해야할지부터 고민

- 적재 알고리즘 이해중..

#### 다음 할 일

1. 적재 알고리즘 이해하기
2. 높이와 무게를 고려한 적재 알고리즘으로 수정하기

---

## 2021.03.04

### 적재 알고리즘 이해 [진행]

- 적재 알고리즘의 이상행동 기록
	- 단순히 운송지 순서대로 박스를 읽히지 않아서 생기는 문제가 아님
```
1. C00 - C01 - B00 (O)
2. C02 - B01 (X, 적재 창이 아예 뜨지 않음)
3. A02 - A00 - B01 (X, 적재 창 뜨고 그 다음 동작X)
4. A00 - A02 - B01 (X, 적재 창 뜨고 그 다음 동작X)
5. A02 - A00 - B00 (X, 적재 창 뜨고 그 다음 동작X)
6. A01 - A02 - B01 (X, 적재 창 뜨고 그 다음 동작X)
7. A00 - A01 - C00 (O)
8. A00 - A02 - B00 (X, 적재 창 뜨고 그 다음 동작X)
9. B00 - B01 - A00 - C00 (O)
10. B01 - B00 - A00 (O)
11. B02 - B00 - C00 (X, 적재 창 뜨고 그 다음 동작X)
```

- 적재 알고리즘이 제대로 동작하기 위한 조건 가설
	- 박스 번호가 건너뛰어져서 읽혀지면 안됨
	- 박스의 번호가 읽혀지는 순서는 상관없음
	- 운송지별로 00 박스는 무조건 읽혀야 함
	- 운송지를 읽히는 순서는 상관이 없음

- 오류 원인
	- 박스 자체를 다 읽히면 앞의 저러한 오류는 애초에 발생하지 않음
	- 테스트를 하면서 일부 박스만 읽히다 보니 이런 오류가 발생하는 것
	- 따라서 이 부분은 오류로 판단하기 보다는 테스트 과정의 어쩔 수 없는 문제라고 해석됨
		- 적재 알고리즘 이해 / 높이, 무게를 고려한 적재 알고리즘에 집중한 후, 이 부분은 여유가 된다면 수정하거나 그대로 유지 

- 적재 알고리즘 이해중...

### 박스 크기 읽기 [진행]

- 오류: 바코드 정보를 얻을 때, 박스의 크기도 함께 측정하는데 이때 박스의 윤곽을 제대로 못 잡고 있으면 박스의 크기가 이상하게 측정됨
- 해결방안:
	- 박스의 윤곽을 제대로 잡지 못하는 문제므로 전경 검출, 코너 디텍트 부분 다시 확인해보기
	- 박스의 크기가 이상하게 측정되면 다시 측정할 수 있도록 조건문 만들어주기
		- 애초에 박스의 크기가 시스템에 저장되어 있는게 아니니 4cm을 3cm으로 측정했다고 오류로 잡을 수는 없다
		- 하지만 만약 박스의 크기 중 하나라도 0으로 잡히면 그것은 오류가 확실하니 다시 측정하도록 코드 변경 필요
	- 이 오류를 박스 디텍션 함수에서 근본적으로 고쳐줘야할지, 바코드 읽는 함수에서 오류 생기면 다시 측정하는 방식으로 바꿔야할지 고민중
		- 일단은 두 함수에서 동시에 고쳐보자 

#### 다음 할 일

1. 박스 크기 읽는 과정에서 발생하는 오류 수정
2. 적재 알고리즘 이해하기
3. 높이와 무게를 고려한 적재 알고리즘으로 수정하기

---

## 2021.03.05

### Box Detection Error

- 원인: 왼쪽 상단은 Y좌표의 최솟값, 왼쪽 하단은 X좌표의 최솟값, 오른쪽 하단은 Y좌표의 최댓값, 오른쪽 상단은 X좌표의 최댓값으로 꼭짓점을 찾도록 설정했는데, 박스가 카메라에 잡히는 모습에 따라 항상 이 방법이 통하지는 않는다. *사진을 보면 4개의 꼭짓점을 코너 디텍트로 제대로 잡고 있더라도, 결과가 저런 식으로 나옴 

![corner_error](https://user-images.githubusercontent.com/46590578/110129987-e2381b80-7e0b-11eb-9153-3b83f1aeeb9f.png)
![corner_error_result](https://user-images.githubusercontent.com/46590578/110130040-efeda100-7e0b-11eb-95cc-23d7827ed640.png)

1. 해결시도1: 전경을 검출한 후 전경을 바이너리 이미지로 변경 -> 모폴로지 클로징 연산 -> 윤곽선 검출 -> 윤곽선으로 사각형 그리고 꼭짓점 얻기
	- 윤곽선을 가지고 사각형을 그리는 것이기 때문에 정확한 박스의 꼭짓점 얻을 수 없음
		- 카메라를 정방향으로 맞춘다면 이게 해결시도2보다 성능이 좋아질 수 있지 않을까? 

2. 해결시도2: *카메라가 박스를 완전히 정방향으로 찍을 때 가장 적합한 코드
	- X 좌표의 최솟값을 찾고, 최솟값 기준 +-20픽셀에서 Y 좌표의 최댓값 최솟값을 찾아 왼쪽 상단, 왼쪽 하단 꼭짓점 찾기
	- X 좌표의 최댓값을 찾고, 최댓값 기준 +-20픽셀에서 Y 좌표의 최댓값 최솟값을 찾아 오른쪽 상단, 오른쪽 하단 꼭짓점 찾기
	- 현재 성능 불안정하고 좋지 않음
```python
    corners = cv2.goodFeaturesToTrack(img_gray, 40, 0.01, 5) # 코너를 찾을 이미지, 코너 최대 검출 개수, 코너 강도, 코너 사이의 거리
    
    # 코너로 검출된 점에서 최소 좌표와 최대 좌표를 찾아서 꼭짓점 결정
    pos = [0, 10000, 0, -1, 0, -1, 0, 10000] # x, min_y, x, max_y, x, max_y, x, min_y
    min_x = 10000
    max_x = -1

    for i in corners:
        x, y = i[0]
        if 5 < x < 635 and 5 < y < 475: # 카메라 화면의 꼭짓점 검출 방지
            if x < min_x: min_x = x # X의 최소 좌표 찾기
            if x > max_x: max_x = x # X의 최대 좌표 찾기
            cv2.circle(img_color, (x, y), 3, (0, 0, 255), 2) # 코너에 원으로 표시

    for i in corners:
        x, y = i[0]
        if min_x - 20 < x < min_x + 20: # X의 최소 좌표 근처에서
            if y < pos[1]: # Y의 최소 좌표 찾기 (왼쪽 상단)
                pos[0] = x
                pos[1] = y
            if y > pos[3]: # Y의 최대 좌표 찾기 (왼쪽 하단)
                pos[2] = x
                pos[3] = y

        if max_x - 20 < x < max_x + 20: # X의 최대 좌표 근처에서
            if y > pos[5]: # Y의 최대 좌표 찾기 (오른쪽 하단)
                pos[4] = x
                pos[5] = y
            if y < pos[7]: # Y의 최소 좌표 찾기 (오른쪽 상단)
                pos[6] = x
                pos[7] = y
```

3. 해결시도3: *카메라의 방향에 상관없이 성능이 괜찮은 코드
	- 기존 코드 그대로 유지, 가끔씩 못 잡더라도 거의 대부분의 경우에 잘 잡음
	- 잘못 잡혔을 때 바코드 정보를 읽는 일이 없도록 바코드 정보 읽는 함수를 고치기
		- 제대로 동작하지 않음
```python
# if box_l <= 1 or box_w <= 1: continue  # 제대로 동작 X
```

#### 다음 할 일

1. 박스 크기 읽는 과정에서 발생하는 오류 수정 시도 계속
2. 적재 알고리즘 이해하기
3. 높이와 무게를 고려한 적재 알고리즘으로 수정하기

---

## 2021.03.08

### Box Detection Error

- 해결시도: 카메라가 박스를 정방향으로 잡을 때와, 비뚤어진 방향으로 잡을 때의 코드를 나누어서 작성
	- 정방향으로 잡는지, 비뚤어진 방향으로 잡는지 조건은 어떻게 줄 수 있을까?
		- 조건 후보1: 꼭짓점을 네 곳에서 잡지 않고, 한 곳에서 두 개를 잡는다면 정방향 (사용O)
		- 조건 후보2: 왼쪽 변에 있는 두 꼭짓점의 X 좌표가 몇 픽셀 이내라면 정방향 (사용X)
	- 정방향으로 잡을 때는
		- 후보1: 컨투어링으로 4개의 꼭짓점 잡아주기 [완료]
		- 후보2: X의 최소좌표, X의 최대좌표 찾고 이를 통해 왼쪽 변, 오른쪽 변에서 2개의 꼭짓점 찾기 [진행]
			- X 좌표의 최솟값을 찾고, 최솟값 기준 +-20픽셀에서 Y 좌표의 최댓값 최솟값을 찾아 왼쪽 상단, 왼쪽 하단 꼭짓점 찾기
			- X 좌표의 최댓값을 찾고, 최댓값 기준 +-20픽셀에서 Y 좌표의 최댓값 최솟값을 찾아 오른쪽 상단, 오른쪽 하단 꼭짓점 찾기
	- 비뚤어진 방향으로 잡을 때는
		- 현재 사용 중인 코드 그대로 유지 [완료]
			- 왼쪽 상단은 Y좌표의 최솟값, 왼쪽 하단은 X좌표의 최솟값, 오른쪽 하단은 Y좌표의 최댓값, 오른쪽 상단은 X좌표의 최댓값으로 꼭짓점을 찾도록 설정

#### 다음 할 일

1. 정방향으로 잡을 때 후보2 시도해보기
2. 적재 알고리즘 이해하기
3. 무게를 고려하여 적재 알고리즘 작성: 무게 무거운 박스를 1층에 배치
4. (높이를 고려하여 적재 알고리즘 작성하기)

---

## 2021.03.10

### Box Detection Error [완료]

- 해결시도: 카메라가 박스를 정방향으로 잡을 때와, 비뚤어진 방향으로 잡을 때의 코드를 나누어서 작성 [완료]
	- 조건: 후보1 사용; 정방향: 후보1 사용; 비뚤어진방향: 원래 코드 그대로 사용 
	- 다양한 박스에 대해 모두 올바르게 박스의 윤곽을 잡는 것을 확인, 연두색 윤곽선은 삭제 예정

![right_direction_camera](https://user-images.githubusercontent.com/46590578/110625083-5430ac00-81e2-11eb-9f0c-1014051b9d27.png)
![misdirected_camera](https://user-images.githubusercontent.com/46590578/110625135-64e12200-81e2-11eb-9a9a-1bae03f23150.png)

- 정방향으로 잡을 때 후보2를 사용하는 코드를 작성해보고, 후보1과의 안정성을 비교
	- 오류: 기준으로 삼을 X의 최소 좌표, X의 최대 좌표는 잘 찾지만, 오른쪽 하단 꼭짓점 pos[4], pos[5]를 왼쪽 하단 꼭짓점인 pos[2], pos[3]과 똑같이 잡음
		- 오류의 원인은 찾지 못한 상태이나, 앞서 시도한 해결방법의 성능이 충분히 좋았으므로 이것을 사용하고 이 코드는 더이상 시도X 

![misdirected_error](https://user-images.githubusercontent.com/46590578/110633712-70d1e180-81ec-11eb-9a31-8a43061409bf.png)

### 적재 알고리즘

- 알아보는 중입니다...

#### 다음 할 일

1. 적재 알고리즘 이해하기
2. 무게를 고려하여 적재 알고리즘 작성: 무게 무거운 박스를 1층에 배치
3. (높이를 고려하여 적재 알고리즘 작성하기)

---

## 2021.03.14

### 적재 알고리즘

- 주석 추가, 테스트 후 코드 업데이트 예정

- 이해가 더 필요한 부분
```python
           for x in range(TRUCK_L):  # X축 방향으로 검사
                count_W = 0
                for y in range(TRUCK_W):  # Y축 방향으로 검사

                    if truck[x][y][floor * BOX_H] == 0:  # 0이면 (비어있는 공간이면)
                        if measureMode == 0:  # 측정 모드가 아니었다면
                            if x < MIN_L:  # 적재한 상자들이 차지하는 공간의 길이의 최소값이면
                                measureMode = 1  # 측정 모드로 전환
                                MIN_L = x  # 최소값 갱신
                                pos_X = x  # 상자를 적재할 x축 좌표 저장
                                pos_Y = y  # 상자를 적재할 y축 좌표 저장
                                pos_Z = floor * BOX_H # 상자를 적재할 z축 좌표 저장 # 이 부분 수정하면 높이에 따라 적재 가능?
                                count_W = 1  # 빈 공간의 너비를 세기 시작함
                        else:  # 측정모드면
                            count_W += 1  # 상자가 들어갈 수 있는 너비를 측정하기 위해 count_W를 증가

                    else:  # 1 또는 2이면 (막혀있는 공간이면)
                        if measureMode == 1: measureMode = 0  # 측정 모드였다면 측정 모드 해제

                if count_W > 0: break # 해당 줄에 빈 공간이 있었다면 x에 대한 반복문 탈출
```
```python
            if max_box_W == 0:  # 해당 공간에 적재할 수 있는 상자가 없다면 빈공간 채우기
                for y in range(count_W):
                    for z in range(BOX_H): # 이 부분 수정하면 높이에 따라 적재 가능?3
                        truck[pos_X][pos_Y + y][pos_Z + z] = 2 # 빈공간은 2로 채우기
```

- 높이나 무게 고려를 위해 수정할 후보
```python
                            if x < MIN_L:  # 적재한 상자들이 차지하는 공간의 길이의 최소값이면
                                measureMode = 1  # 측정 모드로 전환
                                MIN_L = x  # 최소값 갱신
                                pos_X = x  # 상자를 적재할 x축 좌표 저장
                                pos_Y = y  # 상자를 적재할 y축 좌표 저장
                                pos_Z = floor * BOX_H # 상자를 적재할 z축 좌표 저장 # 이 부분 수정하면 높이에 따라 적재 가능?
                                count_W = 1  # 빈 공간의 너비를 세기 시작함
```
```python
                if check[i][j] == 0 and j in inputBox[i]:    # 0. 아직 적재하지 않은 상자이고
                    if inputBox[i][j]['w'] <= count_W:       # 1. 너비가 count_W 이하면
                        if inputBox[i][j]['w'] > max_box_W:  # 2. 최대 너비를 가진 상자를 찾음
                            # 이 부분에 무게에 대한 조건 추가? 최대 무게를 가진 상자를 찾음
```
```python
                            # 5. 적재할 상자의 아래가 막혀있는지 확인
                            if pos_Z != 0:  # 가장 아래층인 경우는 제외
                                for x in range(inputBox[i][j]['l']):
                                    for y in range(inputBox[i][j]['w']):
                                        if truck[pos_X + x][pos_Y + y][pos_Z - 1] == 0: # 막혀있지 않다면 # 이 부분 수정하면 높이에 따라 적재 가능?2
                                            cannot_load = 1 # 적재할 수 없음
                                            break
                                if cannot_load == 1: continue # 불합격
```
```python
            if max_box_W == 0:  # 해당 공간에 적재할 수 있는 상자가 없다면 빈공간 채우기
                for y in range(count_W):
                    for z in range(BOX_H): # 이 부분 수정하면 높이에 따라 적재 가능?3
                        truck[pos_X][pos_Y + y][pos_Z + z] = 2 # 빈공간은 2로 채우기
```

#### 다음 할 일

1. 적재알고리즘 이해 안되는 부분 다시 보기
2. 무게 고려해서 수정해보기
3. (높이 고려해서 수정해보기)

---

## 2021.03.15

### 기존의 적재 알고리즘

- 수정한 코드 테스트 및 업데이트 완료

- 이해 안되는 부분 다시 보기
```python
''' 1. 그래프 그리기 '''

''' https://www.tutorialspoint.com/matplotlib/matplotlib_3d_wireframe_plot.htm '''

# 트럭 내부 모습을 시각화하는 함수
def draw_truck(x_range, y_range, z_range):
    yy, zz = np.meshgrid(y_range, z_range)  # 2차원의 평면에 정사각형 또는 직사각형 그리드 생성
    ax.plot_wireframe(x_range[0], yy, zz, color="black")  # 값의 그리드를 사용하여 3차원 평면에 투영
    ax.plot_wireframe(x_range[1], yy, zz, color="black")

    xx, zz = np.meshgrid(x_range, z_range)
    ax.plot_wireframe(xx, y_range[0], zz, color="black")
    ax.plot_wireframe(xx, y_range[1], zz, color="black")

# 트럭의 시각화를 위한 설정
fig = plt.figure()             # Figure 객체 생성
ax = fig.gca(projection='3d')  # Axe3D 객체 생성; 이 객체에 그림이 그려짐
ax.set_aspect('auto')          # X축과 Y축의 비율을 자동으로 결정
colors = ['gold', 'dodgerblue', 'limegreen']
```

```python
''' 2. 측정 모드 '''

'''x와 y 좌표를 모두 확인하는데, 
1_1. 만약 비어 있는데 측정모드 아니라면 측정모드로 바꾸고 상자를 적재할 위치 저장
1_2. 비어있는데 측정모드라면 상자가 들어갈 너비를 측정 (위치를 저장해놓고 너비 측정하는 것임)
2. Y축 방향으로 검사하다가 애매한 공간이면 측정모드 해제하기 (더 이상 너비를 측정하지 않음)
*해당 X축 방향에서 빈 공간을 찾았다면 X축을 더 보지않고 탈출'''

           for x in range(TRUCK_L):  # X축 방향으로 검사
                count_W = 0
                for y in range(TRUCK_W):  # Y축 방향으로 검사

                    if truck[x][y][floor * BOX_H] == 0:  # 0이면 (비어있는 공간이면)
                        if measureMode == 0:  # 측정 모드가 아니었다면
                            if x < MIN_L:  # 적재한 상자들이 차지하는 공간의 길이의 최소값이면
                                measureMode = 1  # 측정 모드로 전환
                                MIN_L = x  # 최소값 갱신
                                pos_X = x  # 상자를 적재할 x축 좌표 저장
                                pos_Y = y  # 상자를 적재할 y축 좌표 저장
                                pos_Z = floor * BOX_H # 상자를 적재할 z축 좌표 저장
                                count_W = 1  # 빈 공간의 너비를 세기 시작함
                        else:  # 측정모드면
                            count_W += 1  # 상자가 들어갈 수 있는 너비를 측정하기 위해 count_W를 증가

                    else:  # 1 또는 2이면 (막혀있는 공간이면)
                        if measureMode == 1: measureMode = 0  # 측정 모드였다면 측정 모드 해제

                if count_W > 0: break # 해당 줄에 빈 공간이 있었다면 x에 대한 반복문 탈출
```

```python
''' 3. 빈공간 채우기 '''

''' 왜 X축은 고려를 안하고 빈공간을 채울까? 이러면 평면 사각형 그려지지않나? '''

            if max_box_W == 0:  # 해당 공간에 적재할 수 있는 상자가 없다면 빈공간 채우기
                for y in range(count_W):
                    for z in range(BOX_H): # 이 부분 수정하면 높이에 따라 적재 가능?3
                        truck[pos_X][pos_Y + y][pos_Z + z] = 2 # 빈공간은 2로 채우기
```

### 무게를 고려한 적재 알고리즘

```python
# 적재할 상자를 선택할 때, 최대 무게를 가진 상자를 선택할 수 있도록 조건 추가

            ''' 적재할 상자 선택(6가지 조건 확인) '''
            max_box_W = 0  # count_W 너비 안에 들어갈 수 있는 상자의 최대 너비
            max_box_K = 0  # 상자의 최대 무게

            for j in range(NUM_BOX[i]): # 박스 하나하나에 대해 모두 검사
                cannot_load = 0

                if check[i][j] == 0 and j in inputBox[i]:    # 0. 아직 적재하지 않은 상자이고
                    if inputBox[i][j]['w'] <= count_W:       # 1. 너비가 count_W(앞에서 측정함) 이하면
                        if inputBox[i][j]['w'] > max_box_W:  # 2. 최대 너비를 가진 상자를 찾음
                            if inputBox[i][j]['k'] > max_box_K:  # 3. 최대 무게를 가진 상자를 찾음

                                # 4. 해당 위치에 상자를 적재했을 때 트럭 높이를 넘지 않는지 확인
                                count_H = 0  
                                while truck[pos_X][pos_Y][count_H] != 0: count_H += 1 # 해당 위치에 상자 적재 전 높이를 구해줌
                                if count_H + inputBox[i][j]['h'] > TRUCK_H: continue # 높이 넘으면 불합격

                                # 5. 해당 위치에 상자를 적재했을 때 트럭 길이를 넘지 않는지 확인
                                count_L = 0
                                while truck[count_L][pos_Y][pos_Z] != 0: count_L += 1 # 해당 위치에 상자 적재 전 길이를 구해줌
                                if count_L + inputBox[i][j]['l'] > TRUCK_L: continue # 길이 넘으면 불합격

                                # 6. 적재할 상자의 아래가 막혀있는지 확인
                                if pos_Z != 0:  # 가장 아래층인 경우는 제외
                                    for x in range(inputBox[i][j]['l']):
                                        for y in range(inputBox[i][j]['w']):
                                            if truck[pos_X + x][pos_Y + y][pos_Z - 1] == 0: # 막혀있지 않다면 # 이 부분 수정하면 높이에 따라 적재 가능?2
                                                cannot_load = 1 # 적재할 수 없음
                                                break
                                    if cannot_load == 1: continue # 불합격

                                boxIndex = j  # 적재할 상자 인덱스 저장
                                max_box_W = inputBox[i][j]['w']  # 최대 너비 갱신
                                max_box_K = inputBox[i][j]['k']  # 최대 무게 갱신
```

#### 다음 할 일

1. 무게 고려해서 수정해보기
2. (높이 고려해서 수정해보기)

---

## 2021.03.18

### 무게 고려 적재 알고리즘 [완료]

- 무게 정보를 추가한 새로운 바코드 생성 (https://wepplication.github.io/tools/barcodeGen/)
	- 새로운 바코드를 인식하고 무게 정보를 제대로 읽는 것을 확인
	- 폰으로 이미지를 다운받아 인식시켜서인지, 바코드 자체의 문제인지 바코드를 잘 인식하는 것은 아님
		- 추후 바코드 라벨 사이즈에 맞게 생성하여 박스에 붙인 후 테스트 해보아야 확인 가능할 듯

![test1](https://user-images.githubusercontent.com/46590578/111612548-ebb87f00-8820-11eb-8292-fbefc0c4151d.png)
![test2](https://user-images.githubusercontent.com/46590578/111612587-f410ba00-8820-11eb-8546-63d8768ec23b.png)

- 무게 정보 추가에 따른 코드 수정
```python
            box_k = int(barcode_data[3:5]) # 무게
```
```python
send_data_to_host(barcode_data[0:3])  # 바코드 데이터를 Host PC로 전송, 무게 정보는 빼고 전송할 수 있도록 설정
```
```python
# 적재할 상자를 선택할 때, 최대 무게를 가진 상자를 선택할 수 있도록 조건 추가

            ''' 적재할 상자 선택(6가지 조건 확인) '''
            max_box_W = 0  # count_W 너비 안에 들어갈 수 있는 상자의 최대 너비
            max_box_K = 0  # 상자의 최대 무게

            for j in range(NUM_BOX[i]): # 박스 하나하나에 대해 모두 검사
                cannot_load = 0

                if check[i][j] == 0 and j in inputBox[i]:    # 0. 아직 적재하지 않은 상자이고
                    if inputBox[i][j]['w'] <= count_W:       # 1. 너비가 count_W(앞에서 측정함) 이하면
                        if inputBox[i][j]['w'] > max_box_W:  # 2. 최대 너비를 가진 상자를 찾음
                            if inputBox[i][j]['k'] > max_box_K:  # 3. 최대 무게를 가진 상자를 찾음

                                # 4. 해당 위치에 상자를 적재했을 때 트럭 높이를 넘지 않는지 확인
                                count_H = 0  
                                while truck[pos_X][pos_Y][count_H] != 0: count_H += 1 # 해당 위치에 상자 적재 전 높이를 구해줌
                                if count_H + inputBox[i][j]['h'] > TRUCK_H: continue # 높이 넘으면 불합격

                                # 5. 해당 위치에 상자를 적재했을 때 트럭 길이를 넘지 않는지 확인
                                count_L = 0
                                while truck[count_L][pos_Y][pos_Z] != 0: count_L += 1 # 해당 위치에 상자 적재 전 길이를 구해줌
                                if count_L + inputBox[i][j]['l'] > TRUCK_L: continue # 길이 넘으면 불합격

                                # 6. 적재할 상자의 아래가 막혀있는지 확인
                                if pos_Z != 0:  # 가장 아래층인 경우는 제외
                                    for x in range(inputBox[i][j]['l']):
                                        for y in range(inputBox[i][j]['w']):
                                            if truck[pos_X + x][pos_Y + y][pos_Z - 1] == 0: # 막혀있지 않다면 # 이 부분 수정하면 높이에 따라 적재 가능?2
                                                cannot_load = 1 # 적재할 수 없음
                                                break
                                    if cannot_load == 1: continue # 불합격

                                boxIndex = j  # 적재할 상자 인덱스 저장
                                max_box_W = inputBox[i][j]['w']  # 최대 너비 갱신
                                max_box_K = inputBox[i][j]['k']  # 최대 무게 갱신
```

- 적재 알고리즘에 미치는 영향
	- 처음에 적재 알고리즘이 실행X -> 데이터를 보낼 때, 무게까지 같이 보낸 것이 오류 원인이었음
	- 무게에 대한 조건이 너비에 대한 조건보다 우선순위가 낮으므로 무게가 가장 무겁다고 가장 먼저 적재되는 것은 아님

- 참고
	- smart_logi_system_jetson_rev2_corner.py: 무게 고려하지 않은 적재 알고리즘 코드
	- smart_logi_system_jetson_rev5_weight.py: 무게 고려한 적재 알고리즘 코드

#### 다음 할 일

1. 무게 고려 코드 확인 및 보완
2. (높이 고려해서 수정해보기)

---

## 2021.03.05

### 무게 고려 적재 알고리즘

- 새로운 바코드 라벨 제작 및 라벨지에 출력
	- [라벨지사이트](https://printec.co.kr/product/v3530-20%EC%9D%B8%EB%8D%B1%EC%8A%A4%EB%9D%BC%EB%B2%A896%EC%B9%B820%EB%A7%A4/132/category/42/display/1/)
	- [바코드제작](https://wepplication.github.io/tools/barcodeGen/)
	- 출력한 바코드의 크기가 라벨지에 맞게 꽤 커야 바코드를 인식함
		- 너비 25.1, 높이 12.1, 표 가운데 정렬
		- ![barcode_setting](https://user-images.githubusercontent.com/46590578/111954458-7bb13e00-8b2b-11eb-8bc7-e83793c09e71.PNG)
 

### 추가적인 부분

- 박스를 크라프트지로 감싸 더 실제적으로 제작
	- 다양한 규격의 작은 박스를 시중에서 구매하는 건 어떨까?
		- [박스구매](https://front.wemakeprice.com/product/981150866?utm_source=google_ss&utm_medium=cpc&utm_campaign=r_sa&gclid=Cj0KCQjw3duCBhCAARIsAJeFyPXN0j_Bit4gDQG7-qAcdDQIiA3Pf4KpWhUlZ65WyNuf9Y8Uv4anf1EaAvVIEALw_wcB)
		- [박스주문제작](https://boxmania.co.kr/)
			- 견적 문의 후 가격에 따라 다음 방안 결정 
	- 두꺼운 크라프트지로 미니 박스를 만드는 건 어떨까
		- [미니택배상자제작](https://www.youtube.com/watch?v=Cq5ECjitgVg)
	- 얇은 크라프트지를 기존 박스에 붙이는 건 어떨까

- 컨베이어 벨트 자동화
	- 아두이노와 모터를 활용
		- 스위치와 모터를 연결하여 스위치를 꾹 누르고 있으면 모터가 작동하도록 제작

#### 다음 할 일

- 강민지: 라벨지 출력, 박스 견적 문의
- 백이주: 바코드 라벨지 파일 제작
