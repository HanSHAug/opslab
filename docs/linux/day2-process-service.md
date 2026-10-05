# Day 2 — Linux Process & Service Operations

## 1. 목표

* Linux 프로세스 구조 이해
* PID와 프로세스 상태 확인
* CPU/메모리 사용량 분석
* 프로세스 종료 실습
* systemd 서비스 상태 확인
* 장애 상황에서 프로세스와 서비스를 확인하는 기본 절차 익히기

---

## 2. 사용한 명령어

```bash
ps
ps aux
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
top
pstree -p
pgrep -a bash
systemctl status ssh
systemctl is-active ssh
systemctl is-enabled ssh
systemctl list-units --type=service
```

---

## 3. 프로세스란?

Linux에서 실행 중인 프로그램을 프로세스라고 한다.

각 프로세스에는 PID(Process ID)가 부여된다.

주요 정보:

| 항목      | 의미         |
| ------- | ---------- |
| PID     | 프로세스 ID    |
| PPID    | 부모 프로세스 ID |
| USER    | 실행 사용자     |
| %CPU    | CPU 사용률    |
| %MEM    | 메모리 사용률    |
| STAT    | 프로세스 상태    |
| COMMAND | 실행 명령      |

---

## 4. CPU 사용량 확인

```bash
ps aux --sort=-%cpu | head
```

CPU 사용량이 높은 프로세스를 먼저 확인할 수 있다.

서버 장애 발생 시 CPU 과부하 여부를 빠르게 확인하는 데 사용할 수 있다.

---

## 5. 메모리 사용량 확인

```bash
ps aux --sort=-%mem | head
```

메모리를 많이 사용하는 프로세스를 확인할 수 있다.

메모리 부족이나 특정 애플리케이션의 메모리 과다 사용을 조사할 때 활용할 수 있다.

---

## 6. 실습한 프로세스

테스트를 위해 다음 명령을 실행했다.

```bash
sleep 300 &
```

프로세스를 확인했다.

```bash
pgrep -a sleep
```

특정 PID의 상세 정보를 확인했다.

```bash
ps -p PID -o pid,ppid,user,%cpu,%mem,stat,cmd
```

이후 정상 종료 신호를 사용했다.

```bash
kill PID
```

종료 여부를 다시 확인했다.

```bash
pgrep -a sleep
```

---

## 7. kill의 이해

`kill PID`는 단순히 프로세스를 즉시 제거하는 명령이 아니라 기본적으로 SIGTERM 신호를 전달한다.

SIGTERM은 프로세스가 정상적으로 종료할 수 있도록 요청하는 방식이다.

강제 종료가 필요한 경우 SIGKILL을 사용할 수 있다.

```bash
kill -9 PID
```

운영 환경에서는 일반적으로 정상 종료를 먼저 시도하고 필요한 경우 강제 종료를 고려한다.

---

## 8. systemd 서비스

Ubuntu에서는 systemd가 주요 시스템 서비스를 관리한다.

서비스 상태 확인:

```bash
systemctl status ssh
```

현재 실행 여부 확인:

```bash
systemctl is-active ssh
```

부팅 시 자동 실행 여부 확인:

```bash
systemctl is-enabled ssh
```

서비스 상태에서 `active (running)`은 현재 서비스가 정상적으로 실행되고 있음을 의미한다.

---

## 9. 장애 대응 사고방식

### 상황

서버가 갑자기 느려졌다는 장애 신고가 발생했다.

### 기본 확인 절차

```text
서버 이상 감지
      ↓
CPU 사용량 확인
      ↓
메모리 사용량 확인
      ↓
문제 프로세스 식별
      ↓
PID 및 PPID 확인
      ↓
프로세스 상태 및 실행 명령 확인
      ↓
필요한 경우 정상 종료
      ↓
서비스 상태 확인
      ↓
원인 및 조치 기록
```

---

## 10. Incident Note

### Incident

테스트 프로세스가 실행 중인 상황을 장애 상황으로 가정했다.

### Detection

```bash
pgrep -a sleep
```

을 통해 실행 중인 테스트 프로세스를 확인했다.

### Investigation

```bash
ps -p PID -o pid,ppid,user,%cpu,%mem,stat,cmd
```

을 사용하여 프로세스의 PID, PPID, CPU, 메모리, 상태 및 실행 명령을 확인했다.

### Action

```bash
kill PID
```

를 사용하여 정상 종료를 요청했다.

### Verification

```bash
pgrep -a sleep
```

을 다시 실행하여 프로세스가 종료되었는지 확인했다.

### Result

테스트 프로세스가 정상적으로 종료되었음을 확인했다.

---

## 11. 오늘 배운 핵심

1. Linux에서 실행 중인 프로그램은 프로세스로 관리된다.
2. PID는 프로세스를 식별하는 중요한 정보다.
3. `ps`는 프로세스의 현재 상태를 확인할 때 사용한다.
4. `top`은 실시간 시스템 및 프로세스 상태 확인에 유용하다.
5. `pgrep`을 이용하면 특정 프로세스를 쉽게 찾을 수 있다.
6. `kill`은 프로세스에 신호를 전달한다.
7. systemd는 Linux의 주요 서비스를 관리한다.
8. 장애 대응에서는 CPU → 메모리 → 프로세스 → 서비스 순으로 상태를 좁혀가는 사고방식이 중요하다.

---

## 12. 오늘의 운영 관점

단순히 명령어를 외우는 것보다 다음 질문에 답할 수 있는 것이 중요하다.

> 서버가 느려졌다면 무엇을 확인할 것인가?

> CPU를 많이 사용하는 프로세스는 무엇인가?

> 특정 프로세스가 어떤 프로그램에서 생성되었는가?

> 문제가 되는 프로세스를 안전하게 종료하려면 어떻게 해야 하는가?

> 특정 서비스가 현재 실행되고 있는가?

> 서버 재부팅 후에도 해당 서비스가 자동으로 시작되는가?
