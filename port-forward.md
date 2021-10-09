# kubectl port-forward 사용법
## 기본 KIND 구성도

![test](http://drive.google.com/uc?export=view&id=16Q0Sn0ut-3wC1ikjsi_xdsTUEXHlM5_F?raw=true)

## ingress 외부 Test 방법
- client에서 hosts file에 '218.236.22.95 life.com' 등록 (windows: C:\Windows\System32\drivers\etc\hosts)
```
atid@kibana:~$ kubectl -n ingress-nginx port-forward --address 218.236.22.95 service/nginx-ingress-ingress-nginx-controller 5762:80
Forwarding from 218.236.22.95:5762 -> 80
```
## Pod 외부 Test 방법
- client에서 http://218.236.22.95:5762
```
atid@kibana:~$ k port-forward --address 218.236.22.95 pod/jenkins-7b57496c88-qbbxc 5762:80
Forwarding from 218.236.22.95:5762 -> 80
```
## Service 외부 Test 방법
- client에서 http://218.236.22.95:5762
```
atid@kibana:~$ k port-forward --address 218.236.22.95 service/jenkins 5762:80
Forwarding from 218.236.22.95:5762 -> 80
```
## iptables를 사용한 Test 방법
- cluster에 접속된 kubectl 환경에서
```
iptables -t nat -A PREROUTING -p TCP --dport 5762 -j DNAT --to-destination 172.18.70.4:80
iptables -A FORWARD -p tcp --dport 80 -d 172.18.70.4 -j ACCEPT
```
