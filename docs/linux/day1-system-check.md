# Day 1 — Linux System Health Check

## 1. 점검 목적

Linux 서버 운영의 기본 절차를 익히기 위해 시스템의 현재 상태를 점검했다.

이번 점검에서는 다음 항목을 확인했다.

- OS 및 Kernel
- CPU / Memory / Swap
- Disk
- Network Interface
- Routing
- External Connectivity
- DNS
- Listening Ports
- Process
- Systemd Services
- System Load / Uptime

---

## 2. 점검 환경

| 항목 | 결과 |
|---|---|
| OS | Ubuntu 26.04.1 LTS |
| Kernel | 6.18.40.1-microsoft-standard-WSL2 |
| Architecture | x86_64 |
| CPU | 16 logical CPUs |
| Memory | 15 GiB |
| Swap | 4 GiB |
| Environment | WSL2 |
| Project | OpsLab |

---

## 3. System Check

### Kernel

```bash
uname -a

주요 결과:

Linux PRAY 6.18.40.1-microsoft-standard-WSL2
x86_64 GNU/Linux

판단:

Linux Kernel 정상 확인
WSL2 환경에서 동작 중
x86_64 아키텍처 확인
CPU
nproc

결과:

16

논리 CPU 16개를 사용할 수 있다.

Memory / Swap
free -h

확인 결과:

Total Memory: 약 15 GiB
Used Memory: 약 1.3 GiB
Available Memory: 약 14 GiB
Swap: 4 GiB
Swap Used: 0 B

판단:

메모리 사용량이 낮으며 Swap도 사용하지 않고 있어 현재 메모리 압박은 확인되지 않았다.

4. Disk Check
df -h

주요 결과:

/       1007G total   2.6G used   954G available   1%
/mnt/c  953G total    733G used   221G available   77%

판단:

Linux root filesystem은 약 1%만 사용하고 있어 디스크 공간에 문제가 없다.

Windows C: 드라이브는 약 77% 사용 중이므로 향후 별도로 관리할 필요가 있다.

5. Network Interface
ip addr

주요 결과:

eth0
172.27.190.91/20

판단:

WSL의 eth0 네트워크 인터페이스가 활성화되어 있으며 IPv4 주소가 할당되어 있다.

6. Routing
ip route

결과:

default via 172.27.176.1 dev eth0
172.27.176.0/20 dev eth0 proto kernel scope link src 172.27.190.91

판단:

Default Gateway: 172.27.176.1
Local Network: 172.27.176.0/20
Source IP: 172.27.190.91
Network Interface: eth0
7. Network Connectivity
Gateway Test
ping -c 4 172.27.176.1

결과:

4 packets transmitted, 0 received, 100% packet loss

판단:

Default Gateway의 ICMP 응답은 확인되지 않았다.

그러나 이 결과만으로 네트워크 장애라고 판단하지 않고 외부 IP 및 DNS를 추가로 확인했다.

External IP Test
ping -c 4 8.8.8.8

결과:

4 packets transmitted, 4 received, 0% packet loss
rtt avg = 36.744 ms

판단:

외부 IP까지 통신이 가능하며 패킷 손실이 확인되지 않았다.

따라서 인터넷 연결은 정상으로 판단했다.

8. DNS Test
ping -c 4 google.com

결과:

PING google.com (142.250.197.142)
4 packets transmitted, 4 received, 0% packet loss
rtt avg = 58.600 ms

판단:

google.com 도메인이 IP 주소로 정상적으로 해석되었으며 통신도 성공했다.

따라서 DNS resolution과 외부 네트워크 연결 모두 정상으로 판단했다.

9. Listening Port Check
ss -tuln

확인된 주요 포트:

127.0.0.54:53
127.0.0.53:53
10.255.255.254:53
127.0.0.1:33785
127.0.0.1:35229

판단:

53번 포트에서 DNS 관련 서비스가 동작 중
33785, 35229 포트는 localhost에서만 LISTEN
외부에 공개된 HTTP/HTTPS 서비스는 현재 확인되지 않음
10. Port → Process Mapping
ss -tulpn

확인 결과:

127.0.0.1:33785 → PID 6085
127.0.0.1:35229 → PID 6025

PID 6085 확인:

ps -fp 6085

주요 결과:

/home/murphyhan/.vscode-server/...

판단:

해당 프로세스들은 현재 WSL 환경에서 연결된 VS Code Server와 관련된 프로세스이다.

운영 시에는 포트 번호만 확인하는 것이 아니라 PID를 통해 실제 프로세스까지 추적할 수 있다.

11. Process Check
ps aux --sort=-%cpu | head

CPU 사용량이 가장 높은 프로세스는 VS Code Server의 Extension Host였다.

주요 결과:

PID 6680
CPU 1.6%
MEM 2.9%

판단:

CPU 사용량이 매우 낮으며 시스템 전체에 높은 CPU 부하는 확인되지 않았다.

12. Systemd Service Check
systemctl --type=service --state=running

14개의 서비스가 active running 상태였다.

주요 서비스:

chrony
cron
dbus
networkd-dispatcher
polkit
rsyslog
systemd-journald
systemd-logind
systemd-resolved
systemd-udevd
unattended-upgrades

판단:

주요 시스템 서비스들이 정상적으로 실행 중이다.

특히 systemd-resolved가 실행 중이며 앞서 수행한 DNS 테스트도 정상적으로 성공했다.

13. System Load / Uptime
uptime

결과:

up 1:39
1 user
load average: 0.06, 0.07, 0.01

판단:

시스템 가동 시간: 약 1시간 39분
로그인 사용자: 1명
Load Average: 매우 낮음
현재 시스템 부하는 정상 범위로 판단
14. Final Assessment

이번 점검에서 Linux 시스템의 기본적인 상태를 확인했다.

정상 확인
Linux Kernel 정상
CPU / Memory 여유 있음
Linux root filesystem 여유 있음
Network Interface 정상
외부 IP 통신 정상
DNS resolution 정상
주요 systemd 서비스 정상
CPU Load 낮음
Swap 사용 없음
특이사항

Default Gateway 172.27.176.1에 대한 ICMP Ping은 100% packet loss가 발생했다.

그러나 외부 IP 8.8.8.8 및 google.com에 대한 통신은 모두 성공했기 때문에 현재 인터넷 연결 장애로 판단하지 않았다.

15. Operational Thinking

이번 점검에서 중요한 것은 개별 명령어를 암기하는 것이 아니라 하나의 장애를 여러 단계로 분리해서 확인하는 것이다.

Network Interface
        ↓
Routing
        ↓
Gateway
        ↓
External IP
        ↓
DNS
        ↓
Port
        ↓
Process
        ↓
Service
        ↓
System Load

특정 단계에서 문제가 발생하더라도 즉시 장애라고 결론 내리지 않고 다음 단계의 테스트를 통해 원인을 좁혀간다.

16. Commands Used
pwd
whoami
ls -la

uname -a
cat /etc/os-release
uname -m
nproc
free -h
top
df -h

ip addr
ip route
ping -c 4 172.27.176.1
ping -c 4 8.8.8.8
ping -c 4 google.com

ss -tuln
ss -tulpn
ps -fp 6085
ps aux --sort=-%cpu | head

systemctl --type=service --state=running
uptime
