---
tags:
  - kafka
  - error
create: 2024-07-27 18:01:59
---

## InconsistentClusterIdException

### 해결

```sh
rm -rf /tmp/kafka-logs
```

### gpt

로그를 분석해보면, Kafka 서버가 시작 중에 `InconsistentClusterIdException` 오류를 발생시키며 종료되고 있습니다. 이 오류는 Kafka 클러스터의 ID가 `meta.properties` 파일에 저장된 클러스터 ID와 일치하지 않음을 나타냅니다. 구체적인 내용은 다음과 같습니다:

1. **오류 원인**:
   - **`InconsistentClusterIdException`**: 이 예외는 Kafka 브로커가 현재 연결된 ZooKeeper의 클러스터 ID와 `meta.properties` 파일에 저장된 클러스터 ID가 일치하지 않을 때 발생합니다. 현재의 클러스터 ID는 `FGvt6oxDTaScq61Hj0FieQ`인데, `meta.properties`에는 `6NLjhRndTQW9WpESn_EiEw`가 저장되어 있습니다.

2. **문제 해결 방법**:
   - **클러스터 ID 확인**: ZooKeeper에 저장된 클러스터 ID와 `meta.properties` 파일의 클러스터 ID가 일치하는지 확인합니다. `meta.properties` 파일은 Kafka 데이터 디렉터리 (`/tmp/kafka-logs` 등)에 위치합니다.
   - **클러스터 재설정**: 클러스터 ID 불일치 문제를 해결하기 위해, `meta.properties` 파일을 삭제하고 Kafka 브로커를 재시작하여 새로운 클러스터 ID를 생성할 수 있습니다. **주의**: 이 작업은 클러스터의 상태를 초기화하게 되므로, 데이터 손실이 발생할 수 있습니다.
   - **ZooKeeper 설정 확인**: `zookeeper.connect` 설정이 올바른지 확인합니다. 잘못된 ZooKeeper 주소가 설정되어 있을 경우 클러스터 ID 불일치 문제가 발생할 수 있습니다.

3. **조치 사항**:
   - Kafka가 실행되고 있는 디렉토리에서 `meta.properties` 파일을 확인하거나 삭제합니다. Kafka가 올바르게 시작될 수 있도록, 필요한 경우 `kafka-server-start.sh`를 다시 실행합니다.
   - ZooKeeper 설정이 올바른지 확인한 후, 브로커를 재시작하여 문제를 해결합니다.

이러한 단계를 통해 문제를 해결할 수 있으며, 상황에 따라 클러스터 설정을 재검토하거나 Kafka 및 ZooKeeper를 재설치해야 할 수도 있습니다.