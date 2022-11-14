## 1. VMware workstation 16 player 다운로드 및 실행

## 2. ubuntu-20.04.4-live-server-amd64.iso 다운로드

## 3. Create a New Virtual Machine

## 4. 2번째 Installer disc image file(iso)의 Browse..

## 5. ubuntu-20.04.4-live-server-amd64.iso 등록

## 6. Linux OS 이름, 계정 이름과 비밀번호 설정

## 7. VM 이름 설정

## 8. VM의 hard disk size 설정 (본인은 20GB로 설정)

## 9. Customize Hardware 클릭
    * Memory 설정 (Master Node 예정인 VM에게는 4096MB , Worker Node 예정인 VM에게는 2048MB)
    * Processors 설정 (Master Node 예정인 VM은 Core가 2개 이상이어야해서 2개로 설정, Worker Node 예정인 VM은 1개)

## 10. Finish

<hr>   
<hr>

## 1. 언어선택 > 본인은 English 선택

## 2. Installer update > Continue without updating

## 3. Keyboard Configure > Done

## 4. Network Connections 설정 > Done

## 5. Configure proxy 설정 > Done

## 6. Configure ubuntu archive mirror 설정 > Done

## 7. Guided storage configureation > Done

## 8. Storage Configuration > Done > Confirm destructive action > continue

## 9. Profile Setup
   > 이름 , Server Name , pick user name , 비밀번호 , 비밀번호 확인
   > Done

## 10. Enable ubuntu advantage > Done

## 11. SSH Setup > Install openSSH Server 체크 후 > Done

## 12. Featured Server Snaps > Done

## 13. Installing System 마무리 후 Reboot

## 14. 설치 후 ip 확인
      > ifconfig  안되면
      > sudo apt install net-tools 설치 후 다시 ifconfig

<hr>   
<hr>   

## 오류 및 해결방안
## 1. BUG: soft lockup -CPU stuck for 22s...
    커널 업데이트
    > sudo apt-get upgrade
    > sudo apt-get dist-upgrade
    > sudo reboot
    
## 2. dpkg 관련 오류 
    의존성 설치
    > sudo dpkg --configure -a 를 실행시켜 자체적으로 수정할 수 있게끔. 
    실행 안되면
    > sudo apt-get install -f 실행 후 다시 !

## 3. 설치 중 멈춤
    정확하진 않지만 NVIDIA GPU가 있을 경우 충돌이 생긴다고 함 (해결되진 않았고 재설치해서 사용중)
    > Grub 파일을 열어서 수정
    > sudo vim /etc/default/grub , i입력(수정)
    > GRUB_CMDLINE_LINUX_DEFAULT = "quiet splash nomodeset" 로 수정
    > ESC 누르고 :wq (수정된 파일로 저장)
    > sudo update-grub
    > sudo reboot
    > sudo add-apt-repository ppa:graphics-drivers/ppa 쳐서 설치 가능한 그래픽 드라이버 목록 불러오기
    > sudo apt-get update 그래픽 목록 업뎃
    > sudo apt-get install (recommended 된 드라이버확인 후 (nvidia-deiver-470) 설치)
    > sudo reboot 실행
    > 후에 잘 안되면 sudo apt-get update나 upgrade
    참고 : https://velog.io/@se0yeon00/Ubuntu-20.04-Windows-10-%EB%93%80%EC%96%BC-%EB%B6%80%ED%8C%85-%EC%84%A4%EC%B9%98-%EB%B0%8F-nvidia-driver-%EC%84%A4%EC%B9%98

