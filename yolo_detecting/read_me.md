# `Yolo mark detecting`

## 개요
젯슨 tx2 보드의 장점을 이용하여 로봇의 영상처리 부분에 yolo를 사용하였습니다.
목표는 각각의 인지된 글자를 보고 건물의 층을 인식하는 것과 엘레베이터의 숫자를 인식하는 것, 화살표를 인식하는 것입니다.

## 진행상황

 - 2020.1월 ~ 2020.2월: 젯슨 tx2의 개발환경 구축 및 yolo_mark를 이용한 training을 진행하였습니다. 보드의 변화가 있었기 때문에 개발환경 구축을 여러번 진행하였으며 현재는 가장 stable 한 버전인 Jetpack(3.1)로 설치를 하였고 그에 따라 Ubuntu 16.04, Ros kinetic, OpenCV 3.3.1, TensorRT 4.0, cuDNN v7.1.5가 설치되었습니다.

 - 확실한 진행상황은 보드 위에서 yolo_mark 를 통해 training을 시키고 이를 통해 정지된 사진을 detect할 수 있는 것입니다. 동영상은 추후 USB 캠이 오는 대로 진행할 예정입니다.