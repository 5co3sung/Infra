# AI / Linux / Rocky 정리본

## 1. GPU 관련 메모

- GPU가 필요한 이유 : 산술연산을 병렬로 처리하기 위해 사용
- RTX3090에서 90은 CPU안에 있는 보조산술연산의 개수가 많다는 것을 의미

## 5. Rocky 8.10 Python 관련 메모

- rocky 8.10의 경우 기본 pyhton3.6.8이기 때문에
- `yum install python39`로 설치 후
- `update-alternative --config python3`으로 버전 지정해주기
- `python3` 의 경우 `pip3`로 확인 가능
- 아래 명령어 설치 시 오류 발생 메모:
  - `pip install gluonnlp pandas tqdm`
  - 위 명령어 설치 시 오류 발생 파이썬 버전은 3.6

### 5-1. step 1) python 버전을 3.6 -> 3.8로 변경

```bash
yum install python3.8
```

원문 설치 출력 메모:

```text
==============================================================================================================================
 꾸러미                             구조            버전                                             저장소              크기
==============================================================================================================================
설치 중:
 python38                           x86_64          3.8.17-2.module+el8.9.0+1418+f0d66789            appstream           80 k
종속 꾸러미 설치 중:
 python38-libs                      x86_64          3.8.17-2.module+el8.9.0+1418+f0d66789            appstream          8.3 M
 python38-pip-wheel                 noarch          19.3.1-7.module+el8.9.0+1418+f0d66789            appstream          1.0 M
 python38-setuptools-wheel          noarch          41.6.0-5.module+el8.9.0+1418+f0d66789            appstream          303 k
취약한 종속 꾸러미 설치 중:
 python38-pip                       noarch          19.3.1-7.module+el8.9.0+1418+f0d66789            appstream          1.7 M
 python38-setuptools                noarch          41.6.0-5.module+el8.9.0+1418+f0d66789            appstream          666 k
모듈 스트림 활성화:
 python38                                           3.8

연결 요약
==============================================================================================================================
설치  6 꾸러미
```

### 5-2. Python 패키지 및 보조 패키지 설치

- 3.8로 업그레이드 간 필요한 패키지
- 버전을 3.8로 바꿔도 오류가 발생하면 `yum install python3-devel`를 설치한다

```bash
dnf install -y gcc gcc-c++ make cmake protobuf-devel
pip install mxnet
python3 -m pip install --upgrade pip setuptools wheel
pip install sentencepiece==0.1.91
pip install transformers==4.8.2
pip3 install torch
pip3 install git+https://git@github.com/SKTBrain/KoBERT.git@master
pip3 install mxnet-mkl==1.6.0 numpy==1.23.1
pip install scikit-learn
pip install jupyter
```

추가 메모:

- 레드햇 엔터프라이즈 9.5버전은 `sentencepiece`를 `0.1.94`로 깔음
- `pip3 install torch` <-- 시간 오래걸림
- `pip3 install git+https://git@github.com/SKTBrain/KoBERT.git@master` <-- 시간 오래걸림

설치 후:

```bash
pip install --ignore-installed requests
pip install --ignore-installed urllib3==1.25.11
```

---

## 6. Ollama 관련 메모

### 6-1. 개요 메모

- 올라마 고룬이우
- LLM은 방대한 양의 데이터로 사전 학습된 초대형 딥 러닝 모델로서 ollama는 로컬에서 설치하기 적합하기 떄문에 ollama로 진행하였다.

### 6-2. ollama 설치

```bash
yum -y update
yum install curl wget git build-essential
```

참고 메모:

- 록키리눅스는 `build-essential` 설치 시 없다고 나오는데 개발용 툴 깔면 된다고 함
- `nvidia-smi` (nvidia 드라이버 사용하면 설치)

원문 메모:
- `https://ollama.com/download/windows`에 들어가서 자신의 환경에 맞는 OS 선택 후 진행
- 리눅스는 아래 명령어로 실행

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

원문 설치 출력:

```text
[root@localhost ~]# curl -fsSL https://ollama.com/install.sh | sh
>>> Installing ollama to /usr/local
>>> Downloading Linux amd64 bundle
######################################################################## 100.0%
>>> Creating ollama user...
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Adding current user to ollama group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
Created symlink /etc/systemd/system/default.target.wants/ollama.service → /etc/systemd/system/ollama.service.
>>> Downloading Linux ROCm amd64 bundle
######################################################################## 100.0%
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.
>>> AMD GPU ready.
```

### 6-3. 상태 확인 및 모델 다운로드

```bash
systemctl status ollama
ollama pull deepseek-r1:70b
```

메모:
- ollama로 자신의 메모리 상황에 맞게 가져온다
- 메모리 상황의 맞게 계산 하는 방법
- 가져온 모델이 `deepseek-r1:70b` 일경우 `70/2 *1.3`을 해서

원문 다운로드 출력:

```text
[root@localhost aidoctor]# ollama pull  deepseek-r1:70b
pulling manifest
pulling 4cd576d9aa16: 100% ▕████████████████████████████████████████████████████████████▏  42 GB
pulling c5ad996bda6e: 100% ▕████████████████████████████████████████████████████████████▏  556 B
pulling 6e4c38e1172f: 100% ▕████████████████████████████████████████████████████████████▏ 1.1 KB
pulling f4d24e9138dd: 100% ▕████████████████████████████████████████████████████████████▏  148 B
pulling 5e9a45d7d8b9: 100% ▕████████████████████████████████████████████████████████████▏  488 B
verifying sha256 digest
writing manifest
success
```

### 6-4. ollama 버전 업그레이드

원문 메모:
1. `https://ollama.com/download/windows`에 들어가서 자신의 환경에 맞는 OS 선택 후 진행
2. 리눅스는 `curl -fsSL https://ollama.com/install.sh | sh` 로 실행

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

추가 메모:
- ㅁㅁ설치와 동일하게 다운로드 실행 명령어를 수행하면 되는 듯 함
- 다운로드 후 ollama를 재기동 해주어야 함

원문 예시:

```text
[root@localhost ollamaroom]# ollama --version
ollama version is 0.9.2
Warning: client version is 0.12.11
[root@localhost ollamaroom]# systemctl stop ollama
Warning: The unit file, source configuration file or drop-ins of ollama.service changed on disk. Run 'systemctl daemon-reload' to reload units.
[root@localhost ollamaroom]# systemctl daemon-reload
[root@localhost ollamaroom]# systemctl stop ollama
[root@localhost ollamaroom]# systemctl start ollama
[root@localhost ollamaroom]# ollama --version
ollama version is 0.12.11
[root@localhost ollamaroom]#
```

### 6-5. 올라마 개조 메모

```bash
ollama create my-medical-ai --prompt-file prompt.txt
```

### 6-6. Python으로 Ollama 질문/응답 받기

```python
from langchain_ollama import OllamaLLM
llm = OllamaLLM(model="deepseek-r1:70b")
response = llm.invoke("물어보고 싶은 내용")
print(response)
```

원문 예시:

```text
python3
GCC 8.5.0 20210514 (Red Hat 8.5.0-22)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from langchain_ollama import OllamaLLM
>>> llm = OllamaLLM(model="deepseek-r1:70b")
>>> response = llm.invoke("안녕하세요! 오늘 날씨 어때?")
>>> print(response)
<think>
Okay, the user greeted me in Korean and asked about the weather today. I should respond politely.

I need to let them know that while I can't provide real-time information like current weather, I can certainly help with general weather-related questions or anything else they might need assistance with.

I'll keep it friendly and open-ended so they feel comfortable asking more if they want.
</think>

안녕하세요! 오늘 날씨가 어떤지 알려드리면 좋겠지만, 실시간 天氣 情報를 제공할 수 없습니다. 다른 방법으로 도와드릴까요?
```

### 6-7. ollama 실행

- 설치가 완료되면 실행한다. 명령어 자체는 도커와 유사
- `ollama list`로 자신의 목록 확인
- `ollama run 이미지 이름`

원문 예시:

```text
[root@localhost aidoctor]# ollama list
NAME               ID              SIZE     MODIFIED
deepseek-r1:70b    d37b54d01a76    42 GB    45 seconds ago
[root@localhost aidoctor]# ollama run deepseek-r1:70b
```

### 6-8. LangChain / OpenAI 설치 메모

```bash
pip install langchain
pip install langchain-openai
```

메모:
- 그다음 openai인증키 등록을 위해 돈드니까 패스

### 6-9. 검색엔진 추가

원문 메모:
- `https://app.tavily.com/home`에 접속하여 API키를 발급 받는다

```python
from langchain_community.tools import TavilySearchResults
```

```bash
pip3 install langchain langchain-community tavily-python
```

### 6-10. 올라마에 랭체인 연결하기

```bash
#pip3 install langchain-ollama
```

---

## 7. Ubuntu CUDA 드라이버 설치

### 7-1. 기존 NVIDIA / CUDA 관련 정리

```bash
sudo apt-get purge nvidia*
sudo apt-get autoremove
sudo apt-get autoclean
sudo rm -rf /usr/local/cuda*

sudo apt-get update
sudo apt-get upgrade
sudo apt-get install ubuntu-drivers-common
ubuntu-drivers devices
```

원문 출력 예시:

```text
root@ailinux-recommended-PC-for-work:/usr/local# ubuntu-drivers devices
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
udevadm hwdb is deprecated. Use systemd-hwdb instead.
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00002204sv00003842sd00003987bc03sc00i00
vendor   : NVIDIA Corporation
model    : GA102 [GeForce RTX 3090]
driver   : nvidia-driver-570 - distro non-free
driver   : nvidia-driver-470-server - distro non-free
driver   : nvidia-driver-535-open - distro non-free
driver   : nvidia-driver-580-server - distro non-free
driver   : nvidia-driver-470 - distro non-free
driver   : nvidia-driver-570-server-open - distro non-free
driver   : nvidia-driver-535-server - distro non-free
driver   : nvidia-driver-535-server-open - distro non-free
driver   : nvidia-driver-570-server - distro non-free
driver   : nvidia-driver-570-open - distro non-free
driver   : nvidia-driver-535 - distro non-free
driver   : nvidia-driver-580 - distro non-free
driver   : nvidia-driver-580-open - distro non-free recommended   <--recommended가 붙은것을 가장 추천
driver   : nvidia-driver-580-server-open - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

root@ailinux-recommended-PC-for-work:/usr/local#
```

메모:
- `recommend` 붙은게 nvidia에서 검증이 완료 되고, 안정성이 뛰어나기 떄문에 설치를 추천한다.

### 7-2. 권장 드라이버 설치 및 재부팅

```bash
apt-get install <드라이버명칭>
reboot
```

### 7-3. nvidia-smi로 CUDA 버전 확인

- 아래 명령어로 쿠다드라이버 버전 확인을 한다.
- 쿠다 툴킷을 설치하기 위한 버전을 체크

원문 예시:

```text
root@ailinux-recommended-PC-for-work:~# nvidia-smi
Thu Nov 13 14:30:15 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.95.05              Driver Version: 580.95.05      CUDA Version: 13.0     |     <-----여기 버전 확인
+-----------------------------------------+------------------------+----------------------|
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3090        Off |   00000000:01:00.0  On |                  N/A |
|  0%   49C    P8             16W /  420W |      80MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A            1566      G   /usr/lib/xorg/Xorg                       39MiB |
|    0   N/A  N/A            1780      G   /usr/bin/gnome-shell                     13MiB |
+-----------------------------------------------------------------------------------------+
```

### 7-4. CUDA Toolkit 설치

원문 메모:
- 확인 하였으면 NVIDIA CUDA Toolkit Archive 사이트로 들어가서 자신의 버전에 맞는 환경으로 선택하면 명령어가 나오는데 실행 해준다.

```bash
wget https://developer.download.nvidia.com/compute/cuda/13.0.0/local_installers/cuda_13.0.0_580.65.06_linux.run
sudo sh cuda_13.0.0_580.65.06_linux.run
```

설치 중 메모:
- Existing package manager installation of the driver found.  
- `Continue`를 선택

EULA 단계 메모:
- `accept`를 입력 <-- EULA 라이센스에 동의하는 지여부를 물어보는듯

설치 선택 메모:
- 쿠다그라이버는 설치 되어 있기 떄문에 툴킷만 다운로드
- 화살표를 아래로 내려서 `install`에 커서를 위치 한 후 설치를 진행한다.

원문 설치 결과:

```text
root@ailinux-recommended-PC-for-work:~# sudo sh cuda_13.0.0_580.65.06_linux.run
===========
= Summary =
===========

Driver:   Not Selected
Toolkit:  Installed in /usr/local/cuda-13.0/

Please make sure that
 -   PATH includes /usr/local/cuda-13.0/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-13.0/lib64, or, add /usr/local/cuda-13.0/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-13.0/bin
***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 580.00 is required for CUDA 13.0 functionality to work.
To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:
    sudo <CudaInstaller>.run --silent --driver

Logfile is /var/log/cuda-installer.log
```

### 7-5. 환경변수 설정

- `vi /etc/profile` 파일을 열어서 맨 하단에 아래 내용 추가 후 적용

```bash
export PATH=$PATH:/usr/local/cuda/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda:/lib64
export CUDADIR=/usr/local/cuda
```

적용 및 확인:

```bash
source /etc/profile
nvcc -V
```

원문 예시:

```text
root@ailinux-recommended-PC-for-work:~# nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2025 NVIDIA Corporation
Built on Wed_Jul_16_07:30:01_PM_PDT_2025
Cuda compilation tools, release 13.0, V13.0.48
Build cuda_13.0.r13.0/compiler.36260728_0
```

---

## 8. Rocky Linux CUDA 드라이버 설치 (GPU가 있는 경우)

- 올라마가 너무 느려서 연동 후 가속 시키려 함

초기 메모:

```bash
#dnf install epel-release
#sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
#dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

### 8-1. kernel-devel / kernel-headers 설치

```bash
dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

원문 설치 출력 일부:

```text
[root@localhost ~]# dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
cuda-rhel8-x86_64                                                                                                                 36 kB/s | 3.5 kB     00:00
꾸러미 kernel-headers-4.18.0-553.56.1.el8_10.x86_64가 이미 설치되어 있습니다.
종속성이 해결되었습니다.
...
연결 요약
================================================================================================================================================================
```

### 8-2. nvidia-driver / nvidia-settings 설치

```bash
dnf install nvidia-driver nvidia-settings
```

원문 설치 출력 일부:

```text
[root@localhost ~]# dnf install nvidia-driver nvidia-settings
마지막 메타자료 만료확인(0:00:39 이전): 2025년 06월 25일 (수) 오전 10시 18분 27초.
종속성이 해결되었습니다.
=================================================================================================================================================================
 꾸러미                                      구조                         버전                                     저장소                                   크기
=================================================================================================================================================================
설치 중:
 nvidia-driver                               x86_64                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                       5.2 M
 nvidia-settings                             x86_64                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                       859 k
종속 꾸러미 설치 중:
 dkms                                        noarch                       3.2.1-1.el8                              epel                                     90 k
 egl-gbm                                     x86_64                       2:1.1.2.1-1.el8                          cuda-rhel8-x86_64                        22 k
 egl-wayland                                 x86_64                       1.1.19-2.el8                             cuda-rhel8-x86_64                        44 k
 egl-x11                                     x86_64                       1.0.2-1.el8                              cuda-rhel8-x86_64                        57 k
 gcc-c++                                     x86_64                       8.5.0-26.el8_10                          appstream                                12 M
 kmod-nvidia-open-dkms                       noarch                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                        15 M
 libglvnd-opengl                             x86_64                       1:1.3.4-2.el8                            appstream                                46 k
 libstdc++-devel                             x86_64                       8.5.0-26.el8_10                          appstream                               2.1 M
 libvdpau                                    x86_64                       1.5-11.el8                               cuda-rhel8-x86_64                        17 k
 mesa-vulkan-drivers                         x86_64                       23.1.4-4.el8_10                          appstream                               9.8 M
 nvidia-driver-libs                          x86_64                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                       156 M
 nvidia-kmod-common                          noarch                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                        73 M
 nvidia-libXNVCtrl                           x86_64                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                        24 k
 nvidia-modprobe                             x86_64                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                        38 k
 vulkan-loader                               x86_64                       1.3.283.0-1.el8_10                       appstream                               147 k
취약한 종속 꾸러미 설치 중:
 xorg-x11-nvidia                             x86_64                       3:575.57.08-1.el8                        cuda-rhel8-x86_64                       2.2 M

연결 요약
=================================================================================================================================================================
```

추가 메모:

```bash
#sudo dnf install cuda-driver <- 안나옴
yum install nvidia-driver-cuda
```

### 8-3. CUDA 드라이버 제거 명령

```bash
dnf remove nvidia-driver nvidia-settings cuda-driver kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

---

## 9. ROCm 설치

### 9-1. 개요 메모

- Rocm설치 : Nvidia에 쿠다드라이버가 있다면 라이젠에는 Rocm이 있다.
- 2025.07.31 기준으로 Rocky linux도 지원 됨
- AMD ROCm 설치 문서에서 확인 가능

### 9-2. 설치 절차 메모

1. 아래 패키지 설치

```bash
sudo dnf install https://repo.radeon.com/amdgpu-install/6.4.1/rhel/8.10/amdgpu-install-6.4.60401-1.el8.noarch.rpm
```

2. 설치 확인

```bash
sudo amdgpu-install --list-usecase
```

3. ROCm 패키지 설치

```bash
sudo amdgpu-install --usecase=rocm --no-dkms
```

메모:
- 모든 종속성 패키지를 설치한다, 즉 풀패키지를 다운받음

4. 구성 패키지 설치

```bash
sudo dnf install environment-modules
```

5. ROCm 실행 패키지 다운로드  
- ROCm runfile installer 페이지에 접속하여 자신의 환경 / 설치하고자 하는 패키지 버전을 선택 후 링크를 복사한다.
- 예시 rocky-8.10의 os에서 최신버전을 다운받아서 할 경우

```bash
wget https://repo.radeon.com/rocm/installer/rocm-runfile-installer/rocm-rel-6.4.3/el8/rocm-installer_1.1.3.60403-64-128~el8.run
```

원문 출력:

```text
[root@localhost ~]# wget https://repo.radeon.com/rocm/installer/rocm-runfile-installer/rocm-rel-6.4.3/el8/rocm-installer_1.1.3.60403-64-128~el8.run
--2025-08-09 14:40:30--  https://repo.radeon.com/rocm/installer/rocm-runfile-installer/rocm-rel-6.4.3/el8/rocm-installer_1.1.3.60403-64-128~el8.run
Resolving repo.radeon.com (repo.radeon.com)... 125.56.218.72, 125.56.218.73, 2600:140b:2::7d38:da48, ...
Connecting to repo.radeon.com (repo.radeon.com)|125.56.218.72|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7928161826 (7.4G) [application/x-makeself]
Saving to: ‘rocm-installer_1.1.3.60403-64-128~el8.run’
```

추가 메모:

```bash
sudo dnf install amdgpu-dkms
```

### 9-3. rocminfo 출력 예시

```text
[root@localhost emotion]# rocminfo
ROCk module is loaded
=====================
HSA System Attributes
=====================
Runtime Version:         1.15
Runtime Ext Version:     1.7
System Timestamp Freq.:  1000.000000MHz
Sig. Max Wait Duration:  18446744073709551615 (0xFFFFFFFFFFFFFFFF) (timestamp count)
Machine Model:           LARGE
System Endianness:       LITTLE
Mwaitx:                  DISABLED
XNACK enabled:           NO
DMAbuf Support:          NO
VMM Support:             NO

==========
HSA Agents
==========
*******
Agent 1
*******
  Name:                    AMD Ryzen 9 7900X 12-Core Processor
  Uuid:                    CPU-XX
  Marketing Name:          AMD Ryzen 9 7900X 12-Core Processor
  Vendor Name:             CPU
  Feature:                 None specified
  Profile:                 FULL_PROFILE
  Float Round Mode:        NEAR
  Max Queue Number:        0(0x0)
  Queue Min Size:          0(0x0)
  Queue Max Size:          0(0x0)
  Queue Type:              MULTI
  Node:                    0
  Device Type:             CPU
  Cache Info:
    L1:                      32768(0x8000) KB
  Chip ID:                 0(0x0)
  ASIC Revision:           0(0x0)
  Cacheline Size:          64(0x40)
  Max Clock Freq. (MHz):   5733
  BDFID:                   0
  Internal Node ID:        0
  Compute Unit:            24
  SIMDs per CU:            0
  Shader Engines:          0
  Shader Arrs. per Eng.:   0
  WatchPts on Addr. Ranges:1
  Memory Properties:
  Features:                None
  Pool Info:
    Pool 1
      Segment:                 GLOBAL; FLAGS: FINE GRAINED
      Size:                    97563340(0x5d0b2cc) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:4KB
      Alloc Alignment:         4KB
      Accessible by all:       TRUE
    Pool 2
      Segment:                 GLOBAL; FLAGS: EXTENDED FINE GRAINED
      Size:                    97563340(0x5d0b2cc) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:4KB
      Alloc Alignment:         4KB
      Accessible by all:       TRUE
    Pool 3
      Segment:                 GLOBAL; FLAGS: KERNARG, FINE GRAINED
      Size:                    97563340(0x5d0b2cc) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:4KB
      Alloc Alignment:         4KB
      Accessible by all:       TRUE
    Pool 4
      Segment:                 GLOBAL; FLAGS: COARSE GRAINED
      Size:                    97563340(0x5d0b2cc) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:4KB
      Alloc Alignment:         4KB
      Accessible by all:       TRUE
  ISA Info:
*******
Agent 2
*******
  Name:                    gfx803
  Uuid:                    GPU-XX
  Marketing Name:          Radeon RX 580 Series
  Vendor Name:             AMD
  Feature:                 KERNEL_DISPATCH
  Profile:                 BASE_PROFILE
  Float Round Mode:        NEAR
  Max Queue Number:        128(0x80)
  Queue Min Size:          64(0x40)
  Queue Max Size:          131072(0x20000)
  Queue Type:              MULTI
  Node:                    1
  Device Type:             GPU
  Cache Info:
    L1:                      16(0x10) KB
  Chip ID:                 26591(0x67df)
  ASIC Revision:           1(0x1)
  Cacheline Size:          64(0x40)
  Max Clock Freq. (MHz):   1340
  BDFID:                   256
  Internal Node ID:        1
  Compute Unit:            36
  SIMDs per CU:            4
  Shader Engines:          4
  Shader Arrs. per Eng.:   1
  WatchPts on Addr. Ranges:4
  Coherent Host Access:    FALSE
  Memory Properties:
  Features:                KERNEL_DISPATCH
  Fast F16 Operation:      TRUE
  Wavefront Size:          64(0x40)
  Workgroup Max Size:      1024(0x400)
  Workgroup Max Size per Dimension:
    x                        1024(0x400)
    y                        1024(0x400)
    z                        1024(0x400)
  Max Waves Per CU:        40(0x28)
  Max Work-item Per CU:    2560(0xa00)
  Grid Max Size:           4294967295(0xffffffff)
  Grid Max Size per Dimension:
    x                        4294967295(0xffffffff)
    y                        4294967295(0xffffffff)
    z                        4294967295(0xffffffff)
  Max fbarriers/Workgrp:   32
  Packet Processor uCode:: 730
  SDMA engine uCode::      58
  IOMMU Support::          None
  Pool Info:
    Pool 1
      Segment:                 GLOBAL; FLAGS: COARSE GRAINED
      Size:                    8388608(0x800000) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:2048KB
      Alloc Alignment:         4KB
      Accessible by all:       FALSE
    Pool 2
      Segment:                 GLOBAL; FLAGS: EXTENDED FINE GRAINED
      Size:                    8388608(0x800000) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:2048KB
      Alloc Alignment:         4KB
      Accessible by all:       FALSE
    Pool 3
      Segment:                 GROUP
      Size:                    64(0x40) KB
      Allocatable:             FALSE
      Alloc Granule:           0KB
      Alloc Recommended Granule:0KB
      Alloc Alignment:         0KB
      Accessible by all:       FALSE
  ISA Info:
    ISA 1
      Name:                    amdgcn-amd-amdhsa--gfx803
      Machine Models:          HSA_MACHINE_MODEL_LARGE
      Profiles:                HSA_PROFILE_BASE
      Default Rounding Mode:   NEAR
      Default Rounding Mode:   NEAR
      Fast f16:                TRUE
      Workgroup Max Size:      1024(0x400)
      Workgroup Max Size per Dimension:
        x                        1024(0x400)
        y                        1024(0x400)
        z                        1024(0x400)
      Grid Max Size:           4294967295(0xffffffff)
      Grid Max Size per Dimension:
        x                        4294967295(0xffffffff)
        y                        4294967295(0xffffffff)
        z                        4294967295(0xffffffff)
      FBarrier Max Size:       32
*******
Agent 3
*******
  Name:                    gfx1036
  Uuid:                    GPU-XX
  Marketing Name:          AMD Radeon Graphics
  Vendor Name:             AMD
  Feature:                 KERNEL_DISPATCH
  Profile:                 BASE_PROFILE
  Float Round Mode:        NEAR
  Max Queue Number:        128(0x80)
  Queue Min Size:          64(0x40)
  Queue Max Size:          131072(0x20000)
  Queue Type:              MULTI
  Node:                    2
  Device Type:             GPU
  Cache Info:
    L1:                      16(0x10) KB
    L2:                      256(0x100) KB
  Chip ID:                 5710(0x164e)
  ASIC Revision:           1(0x1)
  Cacheline Size:          64(0x40)
  Max Clock Freq. (MHz):   2200
  BDFID:                   4096
  Internal Node ID:        2
  Compute Unit:            2
  SIMDs per CU:            2
  Shader Engines:          1
  Shader Arrs. per Eng.:   1
  WatchPts on Addr. Ranges:4
  Coherent Host Access:    FALSE
  Memory Properties:       APU
  Features:                KERNEL_DISPATCH
  Fast F16 Operation:      TRUE
  Wavefront Size:          32(0x20)
  Workgroup Max Size:      1024(0x400)
  Workgroup Max Size per Dimension:
    x                        1024(0x400)
    y                        1024(0x400)
    z                        1024(0x400)
  Max Waves Per CU:        32(0x20)
  Max Work-item Per CU:    1024(0x400)
  Grid Max Size:           4294967295(0xffffffff)
  Grid Max Size per Dimension:
    x                        4294967295(0xffffffff)
    y                        4294967295(0xffffffff)
    z                        4294967295(0xffffffff)
  Max fbarriers/Workgrp:   32
  Packet Processor uCode:: 22
  SDMA engine uCode::      9
  IOMMU Support::          None
  Pool Info:
    Pool 1
      Segment:                 GLOBAL; FLAGS: COARSE GRAINED
      Size:                    524288(0x80000) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:2048KB
      Alloc Alignment:         4KB
      Accessible by all:       FALSE
    Pool 2
      Segment:                 GLOBAL; FLAGS: EXTENDED FINE GRAINED
      Size:                    524288(0x80000) KB
      Allocatable:             TRUE
      Alloc Granule:           4KB
      Alloc Recommended Granule:2048KB
      Alloc Alignment:         4KB
      Accessible by all:       FALSE
    Pool 3
      Segment:                 GROUP
      Size:                    64(0x40) KB
      Allocatable:             FALSE
      Alloc Granule:           0KB
      Alloc Recommended Granule:0KB
      Alloc Alignment:         0KB
      Accessible by all:       FALSE
  ISA Info:
    ISA 1
      Name:                    amdgcn-amd-amdhsa--gfx1036
      Machine Models:          HSA_MACHINE_MODEL_LARGE
      Profiles:                HSA_PROFILE_BASE
      Default Rounding Mode:   NEAR
      Default Rounding Mode:   NEAR
      Fast f16:                TRUE
      Workgroup Max Size:      1024(0x400)
      Workgroup Max Size per Dimension:
        x                        1024(0x400)
        y                        1024(0x400)
        z                        1024(0x400)
      Grid Max Size:           4294967295(0xffffffff)
      Grid Max Size per Dimension:
        x                        4294967295(0xffffffff)
        y                        4294967295(0xffffffff)
        z                        4294967295(0xffffffff)
      FBarrier Max Size:       32
    ISA 2
      Name:                    amdgcn-amd-amdhsa--gfx10-3-generic
      Machine Models:          HSA_MACHINE_MODEL_LARGE
      Profiles:                HSA_PROFILE_BASE
      Default Rounding Mode:   NEAR
      Default Rounding Mode:   NEAR
      Fast f16:                TRUE
      Workgroup Max Size:      1024(0x400)
      Workgroup Max Size per Dimension:
        x                        1024(0x400)
        y                        1024(0x400)
        z                        1024(0x400)
      Grid Max Size:           4294967295(0xffffffff)
      Grid Max Size per Dimension:
        x                        4294967295(0xffffffff)
        y                        4294967295(0xffffffff)
        z                        4294967295(0xffffffff)
      FBarrier Max Size:       32
*** Done ***
[root@localhost emotion]# rocm-smi
```

### 9-4. rocm-smi 출력 예시

```text
=========================================== ROCm System Management Interface ===========================================
===================================================== Concise Info =====================================================
Device  Node  IDs              Temp    Power    Partitions          SCLK    MCLK     Fan     Perf  PwrCap  VRAM%  GPU%
              (DID,     GUID)  (Edge)  (Avg)    (Mem, Compute, ID)
========================================================================================================================
0       1     0x67df,   39589  29.0°C  7.226W   N/A, N/A, 0         600Mhz  300Mhz   100.0%  auto  145.0W  0%     0%
1       2     0x164e,   3392   36.0°C  43.132W  N/A, N/A, 0         N/A     1800Mhz  0%      auto  N/A     4%     0%
========================================================================================================================
================================================= End of ROCm SMI Log ==================================================
```

### 9-5. clinfo 출력 예시

```text
[root@localhost emotion]# clinfo
Number of platforms:                             1
  Platform Profile:                              FULL_PROFILE
  Platform Version:                              OpenCL 2.1 AMD-APP (3649.0)
  Platform Name:                                 AMD Accelerated Parallel Processing
  Platform Vendor:                               Advanced Micro Devices, Inc.
  Platform Extensions:                           cl_khr_icd cl_amd_event_callback


  Platform Name:                                 AMD Accelerated Parallel Processing
Number of devices:                               1
  Device Type:                                   CL_DEVICE_TYPE_GPU
  Vendor ID:                                     1002h
  Board name:                                    AMD Radeon Graphics
  Device Topology:                               PCI[ B#16, D#0, F#0 ]
  Max compute units:                             1
  Max work items dimensions:                     3
    Max work items[0]:                           1024
    Max work items[1]:                           1024
    Max work items[2]:                           1024
  Max work group size:                           256
  Preferred vector width char:                   4
  Preferred vector width short:                  2
  Preferred vector width int:                    1
  Preferred vector width long:                   1
  Preferred vector width float:                  1
  Preferred vector width double:                 1
  Native vector width char:                      4
  Native vector width short:                     2
  Native vector width int:                       1
  Native vector width long:                      1
  Native vector width float:                     1
  Native vector width double:                    1
  Max clock frequency:                           2200Mhz
  Address bits:                                  64
  Max memory allocation:                         402653184
  Image support:                                 Yes
  Max number of images read arguments:           128
  Max number of images write arguments:          8
  Max image 2D width:                            16384
  Max image 2D height:                           16384
  Max image 3D width:                            16384
  Max image 3D height:                           16384
  Max image 3D depth:                            8192
  Max samplers within kernel:                    16
  Max size of kernel argument:                   1024
  Alignment (bits) of base address:              2048
  Minimum alignment (bytes) for any datatype:    128
  Single precision floating point capability
    Denorms:                                     Yes
    Quiet NaNs:                                  Yes
    Round to nearest even:                       Yes
    Round to zero:                               Yes
    Round to +ve and infinity:                   Yes
    IEEE754-2008 fused multiply-add:             Yes
  Cache type:                                    Read/Write
  Cache line size:                               64
  Cache size:                                    16384
  Global memory size:                            536870912
  Constant buffer size:                          402653184
  Max number of constant args:                   8
  Local memory type:                             Local
  Local memory size:                             65536
  Max pipe arguments:                            16
  Max pipe active reservations:                  16
  Max pipe packet size:                          402653184
  Max global variable size:                      402653184
  Max global variable preferred total size:      536870912
  Max read/write image args:                     64
  Max on device events:                          1024
  Queue on device max size:                      8388608
  Max on device queues:                          1
  Queue on device preferred size:                262144
  SVM capabilities:
    Coarse grain buffer:                         Yes
    Fine grain buffer:                           Yes
    Fine grain system:                           No
    Atomics:                                     No
  Preferred platform atomic alignment:           0
  Preferred global atomic alignment:             0
  Preferred local atomic alignment:              0
  Kernel Preferred work group size multiple:     32
  Error correction support:                      0
  Unified memory for Host and Device:            1
  Profiling timer resolution:                    1
  Device endianess:                              Little
  Available:                                     Yes
  Compiler available:                            Yes
  Execution capabilities:
    Execute OpenCL kernels:                      Yes
    Execute native function:                     No
  Queue on Host properties:
    Out-of-Order:                                No
    Profiling :                                  Yes
  Queue on Device properties:
    Out-of-Order:                                Yes
    Profiling :                                  Yes
  Platform ID:                                   0x7fa20dc73f90
  Name:                                          gfx1036
  Vendor:                                        Advanced Micro Devices, Inc.
  Device OpenCL C version:                       OpenCL C 2.0
  Driver version:                                3649.0 (HSA1.1,LC)
  Profile:                                       FULL_PROFILE
  Version:                                       OpenCL 2.0
  Extensions:                                    cl_khr_fp64 cl_khr_global_int32_base_atomics cl_khr_global_int32_extended_atomics cl_khr_local_int32_base_atomics cl_khr_local_int32_extended_atomics cl_khr_int64_base_atomics cl_khr_int64_extended_atomics cl_khr_3d_image_writes cl_khr_byte_addressable_store cl_khr_fp16 cl_khr_gl_sharing cl_amd_device_attribute_query cl_amd_media_ops cl_amd_media_ops2 cl_khr_image2d_from_buffer cl_khr_subgroups cl_khr_depth_images cl_amd_copy_buffer_p2p cl_amd_assembly_program
```

---

## 10. TensorFlow / PyTorch with ROCm 메모

### 10-1. tensorflow 설치

1. 설치하기 전 자신의 Rocm이 몇버전인지 아래 명령어로 확인 한다.

```bash
rocminfo
```

원문 예시:

```text
[root@warehouse emotion]# rocminfo
ROCk module version 6.12.12 is loaded
=====================
```

2. rocm 라이브러리를 다운받는다.

```bash
yum install rocm-libs miopen-hip
```

원문 설치 출력 일부:

```text
[root@warehouse emotion]# yum install rocm-libs miopen-hip
마지막 메타자료 만료확인(1:52:02 이전): 2025년 11월 17일 (월) 오후 10시 58분 35초.
꾸러미 miopen-hip-3.4.0.60401-83.el8.x86_64가 이미 설치되어 있습니다.
종속성이 해결되었습니다.
======================================================================================================================================================================
 꾸러미                                 구조                                버전                                              저장소                             크기
======================================================================================================================================================================
설치 중:
 rocm-libs                              x86_64                              6.4.1.60401-83.el8                                rocm                              8.4 k

연결 요약
======================================================================================================================================================================
설치  1 꾸러미

전체 내려받기 크기: 8.4 k
설치된 크기 : 9
진행할까요? [y/N]:
```

3. 설치 메모

- Rocm버전이 6.12버전이므로 `pip3 install -Iv tensorflow-rocm==2.13.*`
- 설치가 좀 오래걸리며 반드시 `tensorflow-rocm`으로 깔아야 함

### 10-2. pip dependency 오류 시 가상환경 구성 메모

에러 메모:
- `ERROR: After October 2020 you may experience errors when installing or updating packages. This is because pip will change the way that it resolves dependency conflicts`

아래 명령어 처럼 별도의 환경으로 구성해서 해야 하는 긋 함

```bash
python3 -m venv ~/tf_rocm_env
source ~/tf_rocm_env/bin/activate
pip3 install --upgrade pip
pip3 install numpy==1.24.3 protobuf==3.20.3 typing-extensions==4.5.0 packaging==24.0
pip3 install tensorflow-rocm==2.13.1.600
```

위 명령어 수행 후 다시 `pip3 install -Iv tensorflow-rocm==2.13.*` 로 설치 진행하다가 안되면 아래 명령어로 새환경을 다시 만들어서 해야 함  
텐서플로가 버전이 하나라도 삐긋하면 고장이 나기 때문에 까다로움

```bash
python3 -m venv ~/tf_rocm_env
source ~/tf_rocm_env/bin/activate
pip3 install --upgrade pip
pip3 install numpy==1.24.3
pip3 install protobuf==3.20.3
pip3 install typing-extensions==4.5.0
pip3 install packaging==24.0
pip3 install tensorboard==2.13.0
pip3 install keras==2.13.1
pip3 install tensorflow-rocm==2.13.1.600
pip3 install tensorflow-hub==0.12.0
```

가상환경 종료:

```bash
deactivate
```

### 10-3. tensorflow 대신 pytorch 시도 메모

- 하다가 알았는데 텐서플로가 rocm 5버전까지만 지원이 되어서 내꺼에서는 Rocm 6버전이라 못 돌린다고 한다..........
- 그래서 찾아보니 파이토치라는게 지원한다고 해서 해볼려 한다

1. 기존 가상환경 나가기 및 삭제

```bash
deactivate
rm -rf ~/tf_rocm_env
```

2. 새 가상환경 만들기

```bash
python3 -m venv ~/torch_rocm_env
source ~/torch_rocm_env/bin/activate
pip3 install --upgrade pip
```

원문 설치 메모:

```bash
pip3 install torch==2.4.0+rocm6.1 torchvision==0.15.0+rocm6.1 torchaudio==0.14.0 --index-url https://download.pytorch.org/whl/rocm6.1
pip3 install torch==2.4.0+rocm6.1 torchvision==0.20.0+rocm6.1 torchaudio==0.14.0 --index-url https://download.pytorch.org/whl/rocm6.1
pip3 install numpy==1.24.3
```

테스트:

```bash
python3 -c "import torch; print(torch.__version__); print(torch.cuda.is_available()); print(torch.version.hip)"
```

원문 예시:

```text
(tf_rocm_env) [root@warehouse ~]# python3 -c "import torch; print(torch.__version__); print(torch.cuda.is_available()); print(torch.version.hip)"
2.6.0+rocm6.1
True
6.1.40091-a8dbc0c19
(tf_rocm_env) [root@warehouse ~]#
```

메모:
- 테스트 구문을 위에 처럼 날려서 값 잘 가져오면 성공
- 그리고 테스트 환경에서 스크립트 돌리기 전에 `pip3 install sentence-transformers` 실행
- 그리고 이것도 안된다...........

---

## 11. JSON 데이터 학습 메모

### 11-1. JSON 데이터 준비

1. AI허브에서 Json파일 데이터를 준비한다.
2. 위

```bash
pip install torch --index-url https://download.pytorch.org/whl/rocm6.0
```

---

## 12. 내 IP로 브라우저 창에 접속해서 파이썬 돌리기

### 12-1. FastAPI / Uvicorn 설치

```bash
pip install fastapi uvicorn
```

원문 설치 출력:

```text
[root@localhost emotion]# pip install fastapi uvicorn
WARNING: Running pip install with root privileges is generally not a good idea. Try `pip install --user` instead.
Collecting fastapi
  Downloading fastapi-0.121.2-py3-none-any.whl (109 kB)
     |████████████████████████████████| 109 kB 9.7 MB/s
Collecting uvicorn
  Downloading uvicorn-0.38.0-py3-none-any.whl (68 kB)
     |████████████████████████████████| 68 kB 866 kB/s
Requirement already satisfied: typing-extensions>=4.8.0 in /usr/local/lib/python3.9/site-packages (from fastapi) (4.13.1)
Requirement already satisfied: pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4 in /usr/local/lib/python3.9/site-packages (from fastapi) (2.11.7)
Collecting starlette<0.50.0,>=0.40.0
  Downloading starlette-0.49.3-py3-none-any.whl (74 kB)
     |████████████████████████████████| 74 kB 405 kB/s
Collecting annotated-doc>=0.0.2
  Downloading annotated_doc-0.0.4-py3-none-any.whl (5.3 kB)
Requirement already satisfied: h11>=0.8 in /usr/local/lib/python3.9/site-packages (from uvicorn) (0.14.0)
Requirement already satisfied: click>=7.0 in /usr/local/lib/python3.9/site-packages (from uvicorn) (8.1.8)
Requirement already satisfied: pydantic-core==2.33.2 in /usr/local/lib64/python3.9/site-packages (from pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4->fastapi) (2.33.2)
Requirement already satisfied: annotated-types>=0.6.0 in /usr/local/lib/python3.9/site-packages (from pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4->fastapi) (0.7.0)
Requirement already satisfied: typing-inspection>=0.4.0 in /usr/local/lib/python3.9/site-packages (from pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4->fastapi) (0.4.1)
Requirement already satisfied: anyio<5,>=3.6.2 in /usr/local/lib/python3.9/site-packages (from starlette<0.50.0,>=0.40.0->fastapi) (4.9.0)
Requirement already satisfied: idna>=2.8 in /usr/local/lib/python3.9/site-packages (from anyio<5,>=3.6.2->starlette<0.50.0,>=0.40.0->fastapi) (3.10)
Requirement already satisfied: exceptiongroup>=1.0.2; python_version < "3.11" in /usr/local/lib/python3.9/site-packages (from anyio<5,>=3.6.2->starlette<0.50.0,>=0.40.0->fastapi) (1.2.2)
Requirement already satisfied: sniffio>=1.1 in /usr/local/lib/python3.9/site-packages (from anyio<5,>=3.6.2->starlette<0.50.0,>=0.40.0->fastapi) (1.3.1)
Installing collected packages: starlette, annotated-doc, fastapi, uvicorn
Successfully installed annotated-doc-0.0.4 fastapi-0.121.2 starlette-0.49.3 uvicorn-0.38.0
```

### 12-2. 실행 명령

```bash
uvicorn step_3_web_input:app --host 0.0.0.0 --port 8000
```

메모:
- `uvicorn 파일명:객체명`

---

## 13. Hugging Face 인증 메모

허깅인터페이스로 학습시키기 중 아래 에러가 뜨면:

```text
401 Client Error: Unauthorized for url: https://huggingface.co/gpt-neo-125M/resolve/main/config.json
OSError: Can't load config for 'gpt-neo-125M'.
```

다음 명령어 실행:

```bash
pip3 install huggingface_hub
huggingface-cli login
```

---

## 14. Ollama + 내 학습데이터(텍스트 기반) 파인튜닝 메모

### 14-1. JSON을 JSONL로 변환

- 내가 학습시킨 텍스트 기반의 데이터 json 형식을 jsonl로 바꾸어 주어야 한다

원문 스크립트:

```python
import json

# 입력 JSON 파일 로드
with open("train_data.json", "r", encoding="utf-8") as f:
    data = json.load(f)

# JSONL 파일 생성
with open("train_data.jsonl", "w", encoding="utf-8") as out:
    for item in data:
        user_msg = item["input"]
        assistant_msg = item["output"]

        jsonl_obj = {
            "messages": [
                {"role": "user", "content": user_msg},
                {"role": "assistant", "content": assistant_msg}
            ]
        }
        out.write(json.dumps(jsonl_obj, ensure_ascii=False) + "\n")

print("완료: train_data.jsonl 생성됨")
```

메모:
- 이 내용으로 변환을 시킬 수 있다.

### 14-2. 데이터 / Modelfile 준비

1. 데이터를 준비한다.
2. `vi`로 `Modelfile`을 만든다.

원문 예시:

```text
[root@warehouse medical_treat]# cat Modelfile
FROM llama3.1:8b  <-- 어떤 모델로 쓸것인가
from llama3.1:8b

parameter lora true

adapter ./train_ollama.txt

template """
{{ .Prompt }}
"""
```

### 14-3. Modelfile 기반 생성

```bash
ollama create medical_lora -f Modelfile
```

원문 에러:

```text
[root@warehouse medical_treat]# ollama create medical_lora -f Modelfile

Error: unexpected EOF: lora
```

---

