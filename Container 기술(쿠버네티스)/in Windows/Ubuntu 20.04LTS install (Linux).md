## 1. VMware workstation 17 player 다운로드 및 실행
   * VMware workstation 16 player은 설치 및 개발 중에 알수없는 오류로 멈춤 현상이 있었다
## 2. ubuntu-20.04.4-live-server-amd64.iso 다운로드
   * https://releases.ubuntu.com/20.04/ 에서 20.04LTS 서버 버전 ISO를 다운받는다
## 3. VMware 실행 후 Create a New Virtual Machine
   ![image](https://user-images.githubusercontent.com/96723249/203467267-1e3d887b-661e-44b0-b145-b858592b1b24.png)
## 4. 2번째 Installer disc image file(iso)의 Browse..
   ![image](https://user-images.githubusercontent.com/96723249/203467336-1710aee5-cc4e-4afc-9279-b2119faf855d.png)
## 5. ubuntu-20.04.4-live-server-amd64.iso 등록
   
## 6. Linux OS 이름, 계정 이름과 비밀번호 설정
   ![image](https://user-images.githubusercontent.com/96723249/203467444-2ec802dc-d8b4-429c-8f11-945fc7000efa.png)

## 7. VM 이름 설정

## 8. VM의 hard disk size 설정 (본인은 20GB로 설정)
   ![image](https://user-images.githubusercontent.com/96723249/203467566-88fa7b53-e0ba-4108-99d2-2891d06f57e6.png)

## 9. Customize Hardware 클릭
    * Memory 설정 (Master Node 예정인 VM에게는 4096MB , Worker Node 예정인 VM에게는 2048MB)
    * Processors 설정 (Master Node 예정인 VM은 Core가 2개 이상이어야해서 2개로 설정, Worker Node 예정인 VM은 1개)
   ![image](https://user-images.githubusercontent.com/96723249/203467671-1954e271-8759-4fdc-917b-d07faf9439be.png)

## 10. Finish

<hr>   
<hr>

## 1. 언어선택 > 본인은 English 선택
   ![image](https://user-images.githubusercontent.com/96723249/203467830-c9fe9951-eeaf-4e78-8e17-92deaeafe3c2.png)

## 2. Installer update > Continue without updating
   ![image](https://user-images.githubusercontent.com/96723249/203467859-ede228d5-2b50-49e0-a34b-b5a9b780af50.png)

## 3. Keyboard Configure > Done
   ![image](https://user-images.githubusercontent.com/96723249/203467877-c4577274-5988-4474-978b-306a329c246a.png)

## 4. Network Connections 설정 > Done
   ![image](https://user-images.githubusercontent.com/96723249/203467904-19824c8c-cb38-45ba-8404-fc14e41e5cba.png)

## 5. Configure proxy 설정 > Done
   ![image](https://user-images.githubusercontent.com/96723249/203467921-77ce6f79-080f-4a4e-a50b-867a09cf9777.png)

## 6. Configure ubuntu archive mirror 설정 > Done
   ![image](https://user-images.githubusercontent.com/96723249/203467942-4bfde9d7-678c-49a2-b616-72ae08b864d8.png)

## 7. Guided storage configureation > Done
   ![image](https://user-images.githubusercontent.com/96723249/203467958-c71f58ed-d4c7-4eaf-adc1-6d4b9a4e6d1f.png)

## 8. Storage Configuration > Done > Confirm destructive action > continue
   ![image](https://user-images.githubusercontent.com/96723249/203467980-d5f8ce0c-e381-41a8-b797-e1c6bb521d3b.png)

## 9. Profile Setup
   > 이름 , Server Name , pick user name , 비밀번호 , 비밀번호 확인
   > Done
   ![image](https://user-images.githubusercontent.com/96723249/203468044-a31ecc90-e653-4066-a4f8-16a2bebf2ded.png)

## 10. Enable ubuntu advantage > Done
   ![image](https://user-images.githubusercontent.com/96723249/203468067-6833aeb4-738b-470b-aaa5-f84bf1061617.png)

## 11. SSH Setup > Install openSSH Server 체크 후 > Done
     ![image](https://user-images.githubusercontent.com/96723249/203468099-f7cb9c70-fcb5-42db-8a6d-ca6e7cf54fa9.png)

## 12. Featured Server Snaps > Done
   ![image](https://user-images.githubusercontent.com/96723249/203468125-aff3ed6e-6415-4d91-b5ec-58c84f3ce7f4.png)

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
