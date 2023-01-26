## Master + workder1 + worker2 구성은 상위 폴더에 정리되어있다.
 * PV, PVC , nas storage를 구축하고 연결한다.

![image](https://user-images.githubusercontent.com/96723249/214737282-bf6891cc-618e-4a68-ad33-1aa1933ee9e3.png)

## 















<hr>

## 8. 쿠버네티스 명령어를 통해 NFS를 통한 볼륨 마운트 테스트 (Master node에서)
  * nfs경로를 nginx파드 경로에 마운트됨을 확인하기
  * master node 접속
    * $ vi pv.yaml
    
![image](https://user-images.githubusercontent.com/96723249/214499674-37e5cfdf-279e-4f19-888f-0c4a54f4e9ed.png)

    * $ vi pvc.yaml
![image](https://user-images.githubusercontent.com/96723249/214499832-857f4cd5-dfe2-46f6-9797-b5e28734757c.png)

    * $ vi pod.yaml
![image](https://user-images.githubusercontent.com/96723249/214511559-f658dab2-a346-41d1-af80-99906ec565af.png)

  * NFS 서버에서 /nfsdir에 아무 파일을 추가한 후 아래 명령 실행
    * $ kubectl exec -it mypod --bash
    * mypod내에 마운트한 경로인 /var/www/html 디렉토리를 확인해보면 추가된 것을 볼 수 있음
