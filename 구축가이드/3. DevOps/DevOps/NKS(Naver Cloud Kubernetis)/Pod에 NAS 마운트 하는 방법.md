# Pod에 NAS 마운트 하는 방법

## Pod에 NAS 마운트 (NKS NAS CSI)

---

## 1) 설정 파일 (Kubernetes YAML)

### 1-1. StorageClass / PVC / PV (예: sc_pvc.yaml)

아래 3개 리소스를 한 파일에 순서대로 정의해서 적용합니다.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: static-nks-nas-csi
provisioner: nas.csi.ncloud.com
parameters:
  server: 169.254.84.59 # NAS IP
  share: /n2529009_content # NAS 경로
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
mountOptions:
  - hard
  - nolock
  - nfsvers=3
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: static-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Gi # NAS 용량
  volumeName: static-pv
  storageClassName: static-nks-nas-csi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: static-pvc
  annotations:
    pv.kubernetes.io/provisioned-by: nas.csi.ncloud.com
spec:
  capacity:
    storage: 500Gi
  csi:
    driver: nas.csi.ncloud.com
    volumeHandle: 169.254.84.59:/n2529009_content/static_pv/15276127
    # NAS-IP:/.../15276127 의 "15276127"는 NAS 인스턴스 ID
  volumeAttributes:
    server: 169.254.84.59
    share: /n2529009_content
  storageClassName: static-nks-nas-csi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  mountOptions:
    - hard
    - nolock
    - nfsvers=3
```

### 1-2. (Helm) Pod에 PVC 연결 values-dev.yaml 예시

워크로드 차트의 `values-dev.yaml`에 PVC 볼륨을 추가합니다(차트 구조에 따라 키 이름은 다를 수 있음).

```yaml
# 볼륨(PV/PVC) 설정
volumes:
  - name: pvc-volumes
    persistentVolumeClaim:
      claimName: static-pvc
```

---

## 2) 명령어 (적용/확인)

### 2-1. YAML 적용

```bash
kubectl apply --kubeconfig /root/.kube/prod_kubeconfig.yaml -f sc_pvc.yaml
```

### 2-2. StorageClass 확인

```bash
kubectl get sc --kubeconfig /root/.kube/prod_kubeconfig.yaml
```

### 2-3. PVC 확인

```bash
kubectl get pvc --kubeconfig /root/.kube/prod_kubeconfig.yaml
```

### 2-4. PV 확인

```bash
kubectl get pv --kubeconfig /root/.kube/prod_kubeconfig.yaml
```

---

## 3) 주의사항 (변경/재적용)

- 기존에 적용한 `sc_pvc.yaml`에 변동사항이 생겨 재적용해야 할 경우,
    
    NAS가 마운트된 파드에 대해 먼저 마운트를 해지해야 합니다.
    
- 해지 방법: Argo CD에서 해당 리소스(워크로드)를 `Delete` 처리하여 파드를 내려 마운트를 해지한 뒤 재적용합니다.

![image.png](Pod%EC%97%90%20NAS%20%EB%A7%88%EC%9A%B4%ED%8A%B8%20%ED%95%98%EB%8A%94%20%EB%B0%A9%EB%B2%95/image.png)