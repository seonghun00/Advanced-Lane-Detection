<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=auto&height=80&section=header" width="100%" />
</p>

##### [English](README.md) | **한국어**

# 🚗 차선 인식 고도화 프로젝트 (Advanced Lane Detection)
> **OpenCV를 활용하여 차량 주행 블랙박스 영상에서 차선을 안정적으로 인식하고, 도로 영역을 실시간으로 표시하는 시스템입니다.**

`update_version.ipynb` 파일에는 파이프라인의 각 단계에 대한 설명과 테스트 과정이 한국어 주석과 함께 상세히 작성되어 있습니다.

<p align="left">
    <img src="https://img.shields.io/badge/Python_3.10-3776AB?logo=python&logoColor=white">
    <img src="https://img.shields.io/badge/Jupyter_Notebook-F37626?logo=jupyter&logoColor=white">
</p>

본 프로젝트는 주행 중인 차량의 블랙박스 영상(`test_videos/project_video.mp4`)을 입력받아 카메라 렌즈 왜곡 보정, 버드아이 뷰(Bird's-eye View) 시점 변환, 슬라이딩 윈도우 기반 차선 추적 등 고도화된 컴퓨터 비전 파이프라인을 거쳐 시각화된 결과 영상(`project_clip_result.mp4`)을 추출하는 것이 목적입니다.

`update_version.ipynb` 파일에는 파이프라인의 각 단계에 대한 설명과 테스트 과정이 한국어 주석과 함께 상세히 작성되어 있습니다.

## 🛠 필수 파일 및 폴더 구조

코드를 실행하기 위해서는 아래의 파일과 폴더가 필요합니다.

- `update_version.ipynb` : 메인 코드가 작성된 주피터 노트북 파일입니다. 이 버전을 기준으로 실행합니다.
- `origin.ipynb` : 이전 버전의 주피터 노트북 (참고용 원본)
- `test_videos/project_video.mp4` : 처리에 사용할 주행 영상 원본
- `camera_cal/` : 카메라 왜곡 보정을 위해 사용되는 체스보드 이미지 파일들이 위치한 폴더
- `test_images/` : 파이프라인의 각 단계를 테스트하고 검증하기 위한 정지 이미지 폴더
- `output_images/` : 중간 테스트 결과물과 최종 결합 이미지가 저장되는 폴더

## 🚀 파이프라인 구성

이미지 및 영상을 처리하는 주요 과정은 다음과 같습니다.

### 1. 카메라 캘리브레이션 (Camera Calibration)
카메라 렌즈의 특성으로 인해 발생하는 영상 왜곡(Distortion)을 바로잡습니다. `camera_cal` 폴더에 있는 여러 장의 체스보드 격자 이미지를 분석하여 카메라 왜곡 계수를 계산하고, 주행 영상에 이를 적용하여 이미지를 평평하게 펴줍니다 (`cv2.undistort`).

### 2. 시점 변환 (Perspective Transform)
`cv2.getPerspectiveTransform`과 `cv2.warpPerspective`를 사용하여 이미지에서 차선 영역만 잘라내어 하늘에서 내려다보는 시점(Bird-eye View)으로 변환합니다. 차선의 곡률을 계산하기 쉬워집니다.

### 3. 이미지 임계값 처리 (Image Thresholding)
차선을 명확히 분리해 내기 위해 컬러 스페이스 변환과 그라디언트 필터링을 결합합니다. HLS 색상 공간의 S 채널과 Lab 색상 공간의 b 채널을 조합하여 노란색과 흰색 차선을 뚜렷하게 추출하고, 나머지 배경은 노이즈로 간주하여 지웁니다.

### 4. 차선 탐색 (Lane Searching)
- **Window Search**: 처음 차선을 찾을 때 히스토그램을 사용하여 픽셀 밀도가 가장 높은 곳을 찾아내고 위로 올라가면서 슬라이딩 윈도우 방식으로 좌우 차선의 픽셀들을 추적합니다.
- **Margin Search**: 이전 프레임에서 이미 차선의 곡선을 찾았다면, 전체를 검색할 필요 없이 기존 곡선의 마진 주변만 검색하여 계산 속도를 높입니다.

### 5. 곡률 반경 및 차량 위치 계산 (Radius of Curvature & Vehicle Position)
찾아낸 2차 함수 곡선(`np.polyfit`)을 실제 세계의 단위(미터)로 변환하여 차선의 곡률 반경과, 화면 중앙 기준으로 차량이 차선 내에서 얼마나 치우쳐 있는지 계산합니다.

### 6. 최종 시각화 (Assembling Image)
찾은 차선 영역을 원래 시점으로 되돌린 후(Inverse Perspective Transform), 원본 주행 이미지 위에 초록색 영역으로 투영합니다. 그리고 차선 정보, 중간 처리 과정(필터링, 윈도우 서치 결과)들을 모아 하나의 프레임에 출력되도록 구성합니다 (`assemble_img`).

## ⚙️ 실행 방법

`update_version.ipynb` 파일을 열고 처음부터 끝까지 전체 셀을 실행하기만 하면 됩니다.

## 🎬 최종 결과 영상

아래는 파이프라인을 통해 처리된 최종 차선 인식 결과 영상입니다.

<video src="./project_clip_result.mp4" controls="controls" width="100%" muted="muted" autoplay="autoplay"></video>