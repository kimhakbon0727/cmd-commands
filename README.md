# 💻 PowerShell 명령어 정리

> Windows 환경에서 시스템 관리, 자동화, 네트워크 설정 등을 위한 **PowerShell** 명령어 모음  
> `Cmdlet` 형식: `동사-명사` (예: `Get-Process`, `Set-Item`)

---

## 📋 목차

- [📁 파일 & 디렉토리 관리](#-파일--디렉토리-관리)
- [🌐 네트워크 & 방화벽](#-네트워크--방화벽)
- [⚙️ 프로세스 & 서비스 관리](#️-프로세스--서비스-관리)
- [🖥️ 시스템 정보 & 모니터링](#️-시스템-정보--모니터링)
- [📝 텍스트 처리 & 파이프라인](#-텍스트-처리--파이프라인)
- [👤 사용자 & 권한 관리](#-사용자--권한-관리)
- [🔗 Linux 명령어 대응표](#-linux-명령어-대응표)

---

## 📁 파일 & 디렉토리 관리

### 목록 조회

```powershell
# 현재 디렉토리 목록
Get-ChildItem
Get-ChildItem -Path C:\Users
ls          # 별칭(alias)
dir         # 별칭(alias)

# 숨김 파일 포함
Get-ChildItem -Force

# 재귀 탐색
Get-ChildItem -Recurse

# 특정 확장자만
Get-ChildItem -Filter "*.log"
Get-ChildItem -Recurse -Filter "*.txt"

# 파일만 / 디렉토리만
Get-ChildItem | Where-Object { $_.PSIsContainer -eq $false }  # 파일만
Get-ChildItem | Where-Object { $_.PSIsContainer -eq $true }   # 디렉토리만
```

### 현재 경로 확인 & 이동

```powershell
# 현재 경로 확인
Get-Location
pwd                          # 별칭

# 경로 이동
Set-Location C:\Users
cd C:\Users                  # 별칭
cd ..                        # 상위 디렉토리
cd ~                         # 홈 디렉토리
```

### 파일 & 폴더 생성

```powershell
# 파일 생성
New-Item -Path ".\file.txt" -ItemType File
New-Item -Path ".\file.txt" -ItemType File -Value "내용"

# 폴더 생성
New-Item -Path ".\newfolder" -ItemType Directory
mkdir newfolder              # 별칭

# 중첩 폴더 생성
New-Item -Path ".\a\b\c" -ItemType Directory -Force
```

### 복사 & 이동

```powershell
# 파일 복사
Copy-Item ".\source.txt" ".\dest.txt"
Copy-Item ".\source.txt" "C:\backup\"

# 폴더 복사 (재귀)
Copy-Item ".\src" ".\dst" -Recurse

# 이동 & 이름 변경
Move-Item ".\old.txt" ".\new.txt"
Move-Item ".\file.txt" "C:\backup\"
```

### 삭제

```powershell
# 파일 삭제
Remove-Item ".\file.txt"
del ".\file.txt"             # 별칭

# 폴더 삭제 (재귀)
Remove-Item ".\folder" -Recurse
Remove-Item ".\folder" -Recurse -Force   # 확인 없이 강제 삭제

# 와일드카드 삭제
Remove-Item ".\*.log"
```

### 파일 내용 읽기 & 쓰기

```powershell
# 파일 내용 출력
Get-Content ".\file.txt"
cat ".\file.txt"             # 별칭

# 마지막 n줄 출력
Get-Content ".\file.txt" -Tail 20

# 실시간 로그 모니터링 (tail -f)
Get-Content ".\app.log" -Wait -Tail 10

# 파일 쓰기 (덮어쓰기)
Set-Content ".\file.txt" "내용"

# 파일 추가 (append)
Add-Content ".\file.txt" "추가 내용"

# 파일로 저장 (리다이렉션)
Get-Process | Out-File ".\process.txt"
Get-Process > ".\process.txt"
Get-Process >> ".\process.txt"   # 추가
```

### 경로 & 파일 정보

```powershell
# 파일 속성 확인
Get-Item ".\file.txt" | Select-Object Name, Length, LastWriteTime

# 파일 존재 여부 확인
Test-Path ".\file.txt"
Test-Path "C:\Windows"

# 경로 결합
Join-Path "C:\Users" "Administrator\Desktop"

# 파일 크기 (MB 단위)
(Get-Item ".\file.txt").Length / 1MB

# 디렉토리 용량 합계
(Get-ChildItem -Recurse | Measure-Object -Property Length -Sum).Sum / 1MB
```

### 압축 & 해제

```powershell
# 압축
Compress-Archive -Path ".\folder" -DestinationPath ".\archive.zip"
Compress-Archive -Path ".\*.log" -DestinationPath ".\logs.zip"

# 해제
Expand-Archive -Path ".\archive.zip" -DestinationPath ".\output"
Expand-Archive -Path ".\archive.zip" -DestinationPath ".\output" -Force
```

---

## 🌐 네트워크 & 방화벽

### 네트워크 인터페이스 확인

```powershell
# IP 주소 확인
Get-NetIPAddress
Get-NetIPAddress -AddressFamily IPv4

# 네트워크 어댑터 목록
Get-NetAdapter
Get-NetAdapter | Where-Object { $_.Status -eq "Up" }

# 라우팅 테이블
Get-NetRoute
Get-NetRoute -AddressFamily IPv4
```

### 연결 상태 확인

```powershell
# 포트 연결 상태 (netstat)
Get-NetTCPConnection
Get-NetTCPConnection -State Listen         # 리스닝 포트만
Get-NetTCPConnection -LocalPort 8080       # 특정 포트
Get-NetTCPConnection | Where-Object { $_.State -eq "Established" }

# UDP 연결
Get-NetUDPEndpoint

# 특정 포트 사용 프로세스 확인
Get-NetTCPConnection -LocalPort 80 | Select-Object LocalPort, OwningProcess
```

### 연결 테스트

```powershell
# ping 테스트
Test-Connection google.com
Test-Connection 8.8.8.8 -Count 4
Test-Connection google.com -Quiet           # True/False만 반환

# 포트 연결 테스트 (telnet 대체)
Test-NetConnection google.com -Port 443
Test-NetConnection 192.168.1.1 -Port 22

# 트레이스라우트
Test-NetConnection google.com -TraceRoute
```

### DNS 조회

```powershell
# DNS 조회
Resolve-DnsName google.com
Resolve-DnsName google.com -Type A
Resolve-DnsName google.com -Type MX

# DNS 캐시 확인
Get-DnsClientCache

# DNS 캐시 초기화
Clear-DnsClientCache
```

### 방화벽 관리

```powershell
# 방화벽 상태 확인
Get-NetFirewallProfile
Get-NetFirewallProfile -Name Domain,Public,Private

# 방화벽 규칙 목록
Get-NetFirewallRule
Get-NetFirewallRule -Enabled True
Get-NetFirewallRule -Direction Inbound | Where-Object { $_.Action -eq "Allow" }

# 방화벽 규칙 추가 (인바운드 허용)
New-NetFirewallRule -DisplayName "Allow HTTP" -Direction Inbound `
    -Protocol TCP -LocalPort 80 -Action Allow

# 방화벽 규칙 추가 (아웃바운드 차단)
New-NetFirewallRule -DisplayName "Block FTP" -Direction Outbound `
    -Protocol TCP -RemotePort 21 -Action Block

# 방화벽 규칙 삭제
Remove-NetFirewallRule -DisplayName "Allow HTTP"

# 방화벽 활성화 / 비활성화
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled True
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

### 네트워크 설정

```powershell
# IP 주소 설정
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.100 `
    -PrefixLength 24 -DefaultGateway 192.168.1.1

# DNS 서버 설정
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
    -ServerAddresses 8.8.8.8, 8.8.4.4

# 네트워크 어댑터 재시작
Restart-NetAdapter -Name "Ethernet"
```

---

## ⚙️ 프로세스 & 서비스 관리

### 프로세스 조회

```powershell
# 실행 중인 프로세스 목록
Get-Process
ps                            # 별칭

# 특정 프로세스
Get-Process -Name "chrome"
Get-Process -Id 1234

# CPU/메모리 사용량 상위 정렬
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10

# 특정 정보만 출력
Get-Process | Select-Object Name, Id, CPU, WorkingSet
```

### 프로세스 시작 & 종료

```powershell
# 프로세스 시작
Start-Process notepad.exe
Start-Process "C:\Program Files\app.exe"
Start-Process cmd -ArgumentList "/c ipconfig" -Wait

# 관리자 권한으로 실행
Start-Process powershell -Verb RunAs

# 프로세스 종료
Stop-Process -Name "notepad"
Stop-Process -Id 1234
Stop-Process -Name "chrome" -Force      # 강제 종료

# 특정 이름의 모든 프로세스 종료
Get-Process -Name "chrome" | Stop-Process
```

### 서비스 관리

```powershell
# 서비스 목록
Get-Service
Get-Service -Name "wuauserv"             # Windows Update
Get-Service | Where-Object { $_.Status -eq "Running" }
Get-Service | Where-Object { $_.Status -eq "Stopped" }

# 서비스 시작 / 중지 / 재시작
Start-Service -Name "wuauserv"
Stop-Service -Name "wuauserv"
Restart-Service -Name "wuauserv"

# 서비스 시작 유형 변경
Set-Service -Name "wuauserv" -StartupType Automatic
Set-Service -Name "wuauserv" -StartupType Manual
Set-Service -Name "wuauserv" -StartupType Disabled

# 서비스 상태 확인
(Get-Service -Name "wuauserv").Status
```

### 작업 스케줄러

```powershell
# 예약 작업 목록
Get-ScheduledTask
Get-ScheduledTask | Where-Object { $_.State -eq "Ready" }

# 예약 작업 실행
Start-ScheduledTask -TaskName "TaskName"

# 예약 작업 등록
$action = New-ScheduledTaskAction -Execute "powershell.exe" `
    -Argument "-File C:\scripts\backup.ps1"
$trigger = New-ScheduledTaskTrigger -Daily -At "02:00"
Register-ScheduledTask -TaskName "DailyBackup" -Action $action -Trigger $trigger

# 예약 작업 삭제
Unregister-ScheduledTask -TaskName "DailyBackup" -Confirm:$false
```

---

## 🖥️ 시스템 정보 & 모니터링

### 시스템 기본 정보

```powershell
# OS 정보
Get-ComputerInfo
Get-ComputerInfo | Select-Object OsName, OsVersion, CsName

# Windows 버전
[System.Environment]::OSVersion
winver                        # GUI 창

# 호스트명
$env:COMPUTERNAME
hostname

# 업타임
(Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime
```

### CPU & 메모리

```powershell
# CPU 정보
Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores, MaxClockSpeed

# CPU 사용률
(Get-CimInstance Win32_Processor).LoadPercentage

# 메모리 정보
Get-CimInstance Win32_PhysicalMemory | Select-Object Capacity, Speed
(Get-CimInstance Win32_OperatingSystem) | Select-Object TotalVisibleMemorySize, FreePhysicalMemory

# 메모리 사용률 (%)
$os = Get-CimInstance Win32_OperatingSystem
[math]::Round(($os.TotalVisibleMemorySize - $os.FreePhysicalMemory) / $os.TotalVisibleMemorySize * 100, 2)
```

### 디스크

```powershell
# 디스크 사용량
Get-PSDrive -PSProvider FileSystem
Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID, Size, FreeSpace

# GB 단위로 출력
Get-CimInstance Win32_LogicalDisk | Select-Object DeviceID,
    @{Name="Size(GB)"; Expression={[math]::Round($_.Size/1GB, 2)}},
    @{Name="Free(GB)"; Expression={[math]::Round($_.FreeSpace/1GB, 2)}}
```

### 이벤트 로그

```powershell
# 이벤트 로그 목록
Get-EventLog -List

# 최근 이벤트 조회
Get-EventLog -LogName System -Newest 20
Get-EventLog -LogName Application -Newest 50

# 에러 레벨만 필터
Get-EventLog -LogName System -EntryType Error -Newest 10

# 특정 시간대 조회
Get-EventLog -LogName System -After (Get-Date).AddDays(-1)

# Windows 10/11 이후 권장 방법
Get-WinEvent -LogName System -MaxEvents 20
Get-WinEvent -FilterHashtable @{LogName='System'; Level=2} -MaxEvents 10  # Level 2 = Error
```

### 환경 변수

```powershell
# 환경 변수 목록
Get-ChildItem Env:
ls env:                       # 별칭

# 특정 환경 변수
$env:PATH
$env:COMPUTERNAME
$env:USERNAME

# 환경 변수 설정 (현재 세션)
$env:MY_VAR = "value"

# 환경 변수 영구 설정
[System.Environment]::SetEnvironmentVariable("MY_VAR", "value", "Machine")  # 시스템
[System.Environment]::SetEnvironmentVariable("MY_VAR", "value", "User")     # 사용자
```

### 날짜 & 시간

```powershell
# 현재 날짜/시간
Get-Date
Get-Date -Format "yyyy-MM-dd HH:mm:ss"
Get-Date -Format "yyyyMMdd"

# 날짜 계산
(Get-Date).AddDays(-7)         # 7일 전
(Get-Date).AddHours(3)         # 3시간 후
```

---

## 📝 텍스트 처리 & 파이프라인

### 텍스트 검색 (grep 대체)

```powershell
# 문자열 검색
Select-String -Path ".\*.log" -Pattern "error"
sls "error" ".\app.log"       # 별칭

# 대소문자 무시
Select-String -Path ".\*.log" -Pattern "error" -CaseSensitive:$false

# 재귀 검색
Get-ChildItem -Recurse -Filter "*.log" | Select-String -Pattern "error"

# 일치하지 않는 줄
Select-String -Path ".\file.txt" -Pattern "debug" -NotMatch

# 전후 줄 출력
Select-String -Path ".\app.log" -Pattern "ERROR" -Context 2, 3
```

### 필터링 & 정렬

```powershell
# 조건 필터 (grep/awk 대체)
Get-Process | Where-Object { $_.CPU -gt 10 }
Get-Process | Where-Object Name -like "chrome*"

# 정렬
Get-Process | Sort-Object CPU -Descending
Get-ChildItem | Sort-Object Length

# 상위 n개
Get-Process | Select-Object -First 10
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
```

### 속성 선택 & 출력 형식

```powershell
# 특정 속성만 선택
Get-Process | Select-Object Name, Id, CPU

# 계산된 속성 추가
Get-Process | Select-Object Name,
    @{Name="Memory(MB)"; Expression={[math]::Round($_.WorkingSet/1MB, 2)}}

# 테이블 형식
Get-Process | Format-Table Name, Id, CPU -AutoSize

# 리스트 형식
Get-Process -Name "svchost" | Format-List *

# 넓은 형식 (이름만)
Get-Service | Format-Wide Name -Column 4
```

### 파이프라인 & 집계

```powershell
# 개수 세기 (wc -l 대체)
Get-Process | Measure-Object
(Get-ChildItem).Count

# 합계 / 평균 / 최대 / 최소
Get-ChildItem | Measure-Object -Property Length -Sum -Average -Maximum -Minimum

# 중복 제거
Get-Process | Select-Object -Property Name -Unique

# 그룹화 (카운트)
Get-Process | Group-Object -Property Name | Sort-Object Count -Descending
```

### 문자열 처리

```powershell
# 문자열 치환
"Hello World" -replace "World", "PowerShell"
(Get-Content ".\file.txt") -replace "old", "new" | Set-Content ".\file.txt"

# 문자열 분리
"a,b,c" -split ","
"Hello World" -split " "

# 문자열 합치기
"Hello", "World" -join " "

# 문자열 포함 여부
"Hello World" -like "*World*"
"Hello World" -match "W\w+"

# 앞뒤 공백 제거
"  hello  ".Trim()
"  hello  ".TrimStart()
"  hello  ".TrimEnd()
```

### 출력 & 내보내기

```powershell
# CSV로 내보내기
Get-Process | Export-Csv -Path ".\process.csv" -NoTypeInformation

# CSV 읽기
Import-Csv ".\data.csv"

# JSON으로 내보내기
Get-Process | Select-Object Name, Id, CPU | ConvertTo-Json | Out-File ".\process.json"

# JSON 읽기
Get-Content ".\data.json" | ConvertFrom-Json

# 클립보드로 복사
Get-Process | Out-String | Set-Clipboard

# 화면 + 파일 동시 출력 (tee 대체)
Get-Process | Tee-Object -FilePath ".\output.txt"
```

---

## 👤 사용자 & 권한 관리

### 사용자 조회

```powershell
# 로컬 사용자 목록
Get-LocalUser

# 현재 로그인 사용자
$env:USERNAME
whoami

# 사용자 상세 정보
Get-LocalUser -Name "Administrator"

# 로그인 세션 목록
query user
```

### 사용자 생성 & 수정

```powershell
# 사용자 생성
$password = ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force
New-LocalUser -Name "newuser" -Password $password -FullName "New User"

# 사용자 활성화 / 비활성화
Enable-LocalUser -Name "newuser"
Disable-LocalUser -Name "newuser"

# 비밀번호 변경
$newpw = ConvertTo-SecureString "NewP@ss!" -AsPlainText -Force
Set-LocalUser -Name "newuser" -Password $newpw

# 사용자 삭제
Remove-LocalUser -Name "newuser"
```

### 그룹 관리

```powershell
# 로컬 그룹 목록
Get-LocalGroup

# 그룹 멤버 확인
Get-LocalGroupMember -Group "Administrators"
Get-LocalGroupMember -Group "Remote Desktop Users"

# 그룹에 사용자 추가
Add-LocalGroupMember -Group "Administrators" -Member "newuser"
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "newuser"

# 그룹에서 사용자 제거
Remove-LocalGroupMember -Group "Administrators" -Member "newuser"

# 그룹 생성 / 삭제
New-LocalGroup -Name "DevTeam"
Remove-LocalGroup -Name "DevTeam"
```

### 파일 권한 (ACL)

```powershell
# 권한 확인
Get-Acl ".\file.txt"
Get-Acl "C:\folder" | Format-List

# 권한 부여
$acl = Get-Acl "C:\folder"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "newuser", "FullControl", "Allow")
$acl.SetAccessRule($rule)
Set-Acl "C:\folder" $acl

# 권한 복사 (다른 파일에서 복사)
Get-Acl ".\source.txt" | Set-Acl ".\dest.txt"

# 소유자 확인
(Get-Acl ".\file.txt").Owner
```

### 실행 정책

```powershell
# 현재 실행 정책 확인
Get-ExecutionPolicy
Get-ExecutionPolicy -List

# 실행 정책 변경
Set-ExecutionPolicy RemoteSigned           # 로컬 스크립트 허용
Set-ExecutionPolicy Unrestricted           # 모든 스크립트 허용
Set-ExecutionPolicy Restricted             # 스크립트 실행 금지
Set-ExecutionPolicy Bypass -Scope Process  # 현재 세션만 적용
```

---

## 🔗 Linux 명령어 대응표

| Linux | PowerShell | 설명 |
|-------|-----------|------|
| `ls` | `Get-ChildItem` | 목록 조회 |
| `cd` | `Set-Location` | 디렉토리 이동 |
| `pwd` | `Get-Location` | 현재 경로 |
| `cat` | `Get-Content` | 파일 내용 출력 |
| `cp` | `Copy-Item` | 복사 |
| `mv` | `Move-Item` | 이동/이름변경 |
| `rm` | `Remove-Item` | 삭제 |
| `mkdir` | `New-Item -ItemType Directory` | 폴더 생성 |
| `touch` | `New-Item -ItemType File` | 파일 생성 |
| `grep` | `Select-String` | 패턴 검색 |
| `find` | `Get-ChildItem -Recurse` | 파일 탐색 |
| `ps` | `Get-Process` | 프로세스 목록 |
| `kill` | `Stop-Process` | 프로세스 종료 |
| `top` | `Get-Process \| Sort-Object CPU -Descending` | 리소스 모니터 |
| `df` | `Get-PSDrive` | 디스크 사용량 |
| `du` | `Measure-Object -Property Length -Sum` | 용량 확인 |
| `free` | `Get-CimInstance Win32_OperatingSystem` | 메모리 확인 |
| `tail -f` | `Get-Content -Wait -Tail` | 실시간 로그 |
| `wc -l` | `Measure-Object` | 줄 수 세기 |
| `sort` | `Sort-Object` | 정렬 |
| `uniq` | `Select-Object -Unique` | 중복 제거 |
| `tee` | `Tee-Object` | 화면+파일 동시 출력 |
| `chmod` | `Set-Acl` | 권한 변경 |
| `chown` | `Set-Acl` | 소유자 변경 |
| `ping` | `Test-Connection` | 연결 테스트 |
| `netstat` | `Get-NetTCPConnection` | 네트워크 상태 |
| `ifconfig` / `ip` | `Get-NetIPAddress` | 네트워크 정보 |
| `curl` / `wget` | `Invoke-WebRequest` | HTTP 요청 |
| `ssh` | `Enter-PSSession` | 원격 접속 |
| `cron` | `Register-ScheduledTask` | 스케줄링 |
| `history` | `Get-History` | 명령어 히스토리 |
| `whoami` | `$env:USERNAME` / `whoami` | 사용자 확인 |
| `hostname` | `$env:COMPUTERNAME` | 호스트명 확인 |

---

## 💡 유용한 팁

### 도움말

```powershell
# 명령어 도움말
Get-Help Get-Process
Get-Help Get-Process -Examples
Get-Help Get-Process -Full

# 도움말 업데이트
Update-Help
```

### 별칭 확인

```powershell
# 모든 별칭 목록
Get-Alias

# 특정 별칭 확인
Get-Alias ls
Get-Alias -Definition "Get-ChildItem"
```

### 히스토리

```powershell
# 명령어 히스토리
Get-History
h                             # 별칭

# 히스토리에서 재실행
Invoke-History 5              # 5번 명령어 재실행
r 5                           # 별칭

# 히스토리 검색 (Ctrl+R 대체)
Get-History | Where-Object CommandLine -like "*Process*"
```

### 원격 실행

```powershell
# 원격 명령 실행
Invoke-Command -ComputerName "Server01" -ScriptBlock { Get-Process }

# 원격 세션 접속
Enter-PSSession -ComputerName "Server01"
Exit-PSSession

# 여러 서버에 동시 실행
Invoke-Command -ComputerName "Server01", "Server02" -ScriptBlock { Get-Service }
```
