1. Linux용 Windows 하위 시스템
  * 제어판 -> 프로그램 및 기능 -> Windows 기능 켜기/끄기 -> Linux용 Windows 하위 시스템 선택 -> 확인
  ![image](https://user-images.githubusercontent.com/96723249/230844186-9e34b699-64ba-4931-99e4-3ef650fba290.png)

2. 개발자 모드
  * 설정 -> 업데이트 및 보안 -> 개발자용 -> 개발자 모드 켬 -> 그 후 컴퓨터 재시작
  ![image](https://user-images.githubusercontent.com/96723249/230844279-1f09c21a-9b49-4092-b1d4-47606ef8d92d.png)

3. Windows 10 최신 업데이트하기
  ![image](https://user-images.githubusercontent.com/96723249/230844340-9e1e143a-9673-49bf-b02e-7d0b1a6e4d79.png)
  * 설정 -> Windows 업데이트 -> 고급 옵션 -> “Window를 업데이트할 때 다른 Microsoft 제품에 대한 업데이트 받기” 활성화

4. powershell 권리자 실행
  * powershell 검색 -> 권리자 권한으로 실행

5. WSL 시스템 활성화
  * dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
 
6. Virtual Machine 기능을 활성화
  * dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

7. Linux 커널 업데이트 패키지 다운로드
  * x64 머신용 최신 WSL2 Linux 커널 업데이트 패키지
  * https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
  * 링크를 누르면 자동으로 다운받아지고 설치를 진행하면 된다.

8. WSL 2 기본 버전 세팅
  * wsl --set-default-version 2
  * 만약 에러가 발생한다면 ?
  * bcdedit /set hypervisorlaunchtype auto 를 친 후 재부팅을 한다.

9. Ubuntu 22.04 Lts 설치
  * Microsoft Store에서 Ubuntu 22.04 LTS를 검색해 다운받는다.

10. Microsoft store에서 터미널을 다운받고, 커스터마이징 할 수 있다.

