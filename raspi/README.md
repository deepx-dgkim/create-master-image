# Raspberry Pi SD Card 백업 가이드 (PiShrink 사용)

이 문서는 Raspberry Pi SD Card를 `.img` 파일 형태로 백업하고,
`PiShrink`를 사용해 실제 사용 용량 기준으로 이미지를 최소화하는 방법을 설명합니다.

예시:

* SD Card 실제 용량: 128GB
* 실제 사용량: 30GB
* 최종 결과물: 약 8GB ~ 20GB 수준의 `.img.gz`

---

# 준비물

* Ubuntu/Debian Linux PC
* SD Card Reader
* Raspberry Pi SD Card
* 충분한 저장 공간

---

# 1. SD Card 연결

Linux PC에 Raspberry Pi SD Card를 연결합니다.

디바이스 확인:

```bash id="48k22m"
lsblk
```

예시 출력:

```bash id="t8p62h"
sda      931.5G
sdb      119.2G
├─sdb1     512M
└─sdb2   118.7G
```

위 예시에서:

```text id="vaw54p"
/dev/sdb 가 SD Card 전체 디바이스
```

> WARNING
> 디바이스 이름을 반드시 정확히 확인하세요.
> 잘못 지정하면 시스템 디스크를 덮어쓸 수 있습니다.

---

# 2. 파일시스템 검사 (권장)

이미지 생성 전에 ext4 filesystem 검사 수행:

```bash id="r4zjlwm"
sudo e2fsck -f /dev/sdb2
```

---

# 3. Raw Image 생성

`dd` 명령으로 전체 SD Card 이미지를 생성합니다.

```bash id="6xq1o5"
sudo dd if=/dev/sdb of=raspi_backup.img bs=64M status=progress conv=fsync
```

옵션 설명:

| 옵션                | 설명             |
| ----------------- | -------------- |
| `if=`             | 입력 디바이스        |
| `of=`             | 출력 이미지 파일      |
| `bs=64M`          | 복사 속도 향상       |
| `status=progress` | 진행률 표시         |
| `conv=fsync`      | 안전하게 디스크 flush |

---

# 4. PiShrink 설치

PiShrink 저장소 clone:

```bash id="lbp5fr"
git clone https://github.com/Drewsif/PiShrink.git
cd PiShrink
chmod +x pishrink.sh
```

공식 저장소:

[PiShrink GitHub](https://github.com/Drewsif/PiShrink?utm_source=chatgpt.com)

---

# 5. 이미지 Shrink 수행

압축 없이 shrink:

```bash id="5d0v4y"
sudo ./pishrink.sh ../raspi_backup.img
```

gzip 압축까지 함께 수행:

```bash id="v6m9cg"
sudo ./pishrink.sh -z ../raspi_backup.img
```

결과:

```text id="l4kpje"
raspi_backup.img.gz
```

생성됨.

---

# PiShrink가 수행하는 작업

PiShrink는 자동으로 다음 작업을 수행합니다.

1. ext4 filesystem 축소
2. partition 크기 축소
3. 사용하지 않는 block 제거
4. image 파일 truncate
5. gzip 압축 (옵션)

즉:

```text id="by2i80"
128GB 전체 이미지
↓
실제 사용량 기준 최소화
↓
작은 용량의 img 생성
```

---

# 6. SD Card에 이미지 복원

압축된 이미지 복원:

```bash id="z7xyk0"
gunzip -c raspi_backup.img.gz | sudo dd of=/dev/sdb bs=64M status=progress conv=fsync
```

압축되지 않은 이미지 복원:

```bash id="v4bggn"
sudo dd if=raspi_backup.img of=/dev/sdb bs=64M status=progress conv=fsync
```

---

# 자동 Filesystem 확장

대부분의 Raspberry Pi OS는 첫 부팅 시 자동으로 filesystem을 확장합니다.

즉:

* 작은 이미지로 배포 가능
* 큰 SD Card에서도 전체 용량 자동 사용 가능

---

# 유용한 팁

## 이미지 크기 확인

```bash id="8j91dc"
du -sh raspi_backup.img*
```

---

## 더 강한 압축 사용

gzip보다 더 높은 압축률:

```bash id="6mh94j"
xz -T0 -z raspi_backup.img
```

장점:

* 파일 크기 더 작음

단점:

* 압축 속도 느림

---

# 추천 Workflow

```text id="hlsg1j"
SD Card
   ↓
dd 백업
   ↓
PiShrink
   ↓
최소화된 compressed image 생성
```

---

# 참고사항

* PiShrink는 ext4 기반 Raspberry Pi OS에 가장 적합합니다.
* 가능하면 mount 해제된 상태에서 작업하는 것을 권장합니다.
* SD Card 제거 전 반드시 safely remove 수행하세요.

---

# 참고 링크

* [PiShrink GitHub](https://github.com/Drewsif/PiShrink?utm_source=chatgpt.com)
* [Raspberry Pi Official Website](https://www.raspberrypi.com/?utm_source=chatgpt.com)

