# java-ci-test

## CI/CD 설정

`main` push 시 Java 17로 테스트·빌드한 뒤 EC2로 배포합니다.
`main` 대상 PR은 테스트·빌드만 실행합니다. Actions 탭에서 main을 선택해
수동 실행할 수도 있습니다.

### EC2 준비

- Java 17, `java-ci-test` systemd 서비스가 설치되어 있어야 합니다.
- 서비스는 `/home/ec2-user/app.jar`를 실행하고 8080 포트를 사용합니다.
- SSH 계정은 `ec2-user`이며 `sudo -n systemctl stop java-ci-test`,
  `sudo -n systemctl start java-ci-test`, `sudo -n journalctl` 실행 권한이 필요합니다.
- EC2 보안 그룹의 22번 포트에 GitHub Actions 실행 환경의 접근이 허용되어야 합니다.
  본인 PC IP만 허용하면 Actions는 접속할 수 없습니다. 좁은 IP 규칙이 필요하면
  고정 출구 IP를 가진 runner를 사용하세요. 상태 확인은 EC2 내부에서 하므로
  배포 확인을 위해 8080을 외부에 개방할 필요는 없습니다.

### GitHub Secrets

저장소 Settings → Secrets and variables → Actions → New repository secret에서 등록합니다.

| 이름 | 값 |
| --- | --- |
| `EC2_HOST` | EC2 퍼블릭 IP 또는 DNS 이름 (`http://` 제외) |
| `EC2_USER` | `ec2-user` |
| `EC2_SSH_KEY` | PEM 파일 전체 내용 (BEGIN/END 줄 포함) |

PEM 파일을 저장소에 커밋하지 마세요.

학습용 설정으로 SSH 서버 호스트 키 검증을 생략합니다.

### 배포 동작

JAR를 `app.jar.next`로 전송한 뒤 서비스를 중지하고 `app.jar`를 교체한 후
시작합니다. `/` 응답이 HTTP 200인지 확인합니다. 응답 문구는 자유롭게 바꿀 수 있습니다.
배포는 동시에 실행하지 않으며 진행 중인 배포를 새 push로 취소하지 않습니다.
재시작 중에는 잠시 서비스가 중단됩니다. 자동 롤백은 구현하지 않았습니다.
응답 확인에 실패하면 서비스 로그를 출력하고 Actions 실행이 실패합니다.

Secrets와 네트워크 설정 후 main에 push하고 Actions의 **Java CI and EC2 CD**
실행 결과를 확인하세요.
