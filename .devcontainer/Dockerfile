# 개발 컨테이너 공식 Ubuntu 24.04 이미지(vscode 사용자 포함)를 버전 태그로 고정한다.
FROM mcr.microsoft.com/devcontainers/base:3.0.10-ubuntu24.04

# 팀 표준 도구 버전을 한곳에 고정한다. 올릴 때는 이 값만 바꾼다.
ARG TERRAFORM_VERSION=1.16.0
ARG TFLINT_VERSION=0.64.0
ARG KUBECTL_VERSION=1.36.4
ARG HELM_VERSION=3.21.2
ARG KUSTOMIZE_VERSION=5.8.1
ARG KUBECONFORM_VERSION=0.8.0
ARG ISTIOCTL_VERSION=1.30.4
ARG ARGOCD_VERSION=3.5.2
ARG ARGO_ROLLOUTS_VERSION=1.10.0
ARG KYVERNO_VERSION=1.19.0
ARG COSIGN_VERSION=3.1.3
ARG SYFT_VERSION=1.51.1
ARG TRIVY_VERSION=0.74.0
ARG AWSCLI_VERSION=2.36.44
# 빌드 대상 CPU 종류(amd64 또는 arm64)를 BuildKit이 플랫폼마다 채운다.
ARG TARGETARCH

# 파이프 중간 명령이 실패해도 그 자리에서 빌드를 멈춘다.
SHELL ["/bin/bash", "-euo", "pipefail", "-c"]

# 다운로드·압축 해제·DNS 확인에 쓰는 기본 패키지를 설치한다.
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends ca-certificates curl dnsutils jq unzip \
 && rm -rf /var/lib/apt/lists/*

# 자격 증명 폴더를 vscode 소유로 미리 만든다. 없으면 mount할 때 Docker가 root 소유로 만든다.
RUN mkdir -p /home/vscode/.aws /home/vscode/.config/gh /home/vscode/.kube \
 && chown -R vscode:vscode /home/vscode/.aws /home/vscode/.config /home/vscode/.kube

# 내려받은 파일은 임시 폴더에 두고, 단계마다 비운다.
WORKDIR /tmp/tools

# Terraform: 공식 SHA256SUMS로 검증한다. 값이 다르면 빌드가 멈춘다.
RUN curl -fsSLO "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_${TARGETARCH}.zip" \
 && curl -fsSLO "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_SHA256SUMS" \
 && sha256sum --check --ignore-missing "terraform_${TERRAFORM_VERSION}_SHA256SUMS" \
 && unzip -o "terraform_${TERRAFORM_VERSION}_linux_${TARGETARCH}.zip" terraform -d /usr/local/bin \
 && rm -rf /tmp/tools/*

# tflint: 릴리스의 checksums.txt로 검증한다.
RUN curl -fsSLO "https://github.com/terraform-linters/tflint/releases/download/v${TFLINT_VERSION}/tflint_linux_${TARGETARCH}.zip" \
 && curl -fsSLO "https://github.com/terraform-linters/tflint/releases/download/v${TFLINT_VERSION}/checksums.txt" \
 && sha256sum --check --ignore-missing checksums.txt \
 && unzip -o "tflint_linux_${TARGETARCH}.zip" tflint -d /usr/local/bin \
 && rm -rf /tmp/tools/*

# kubectl: 같은 주소의 .sha256 파일(해시 값만 들어 있음)로 검증한다.
RUN curl -fsSLO "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl" \
 && curl -fsSLO "https://dl.k8s.io/release/v${KUBECTL_VERSION}/bin/linux/${TARGETARCH}/kubectl.sha256" \
 && echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check \
 && install -m 0755 kubectl /usr/local/bin/kubectl \
 && rm -rf /tmp/tools/*

# helm: 압축 파일 옆의 .sha256sum으로 검증하고 helm 실행 파일만 꺼낸다.
RUN curl -fsSLO "https://get.helm.sh/helm-v${HELM_VERSION}-linux-${TARGETARCH}.tar.gz" \
 && curl -fsSLO "https://get.helm.sh/helm-v${HELM_VERSION}-linux-${TARGETARCH}.tar.gz.sha256sum" \
 && sha256sum --check "helm-v${HELM_VERSION}-linux-${TARGETARCH}.tar.gz.sha256sum" \
 && tar -xzf "helm-v${HELM_VERSION}-linux-${TARGETARCH}.tar.gz" -C /usr/local/bin --strip-components=1 "linux-${TARGETARCH}/helm" \
 && rm -rf /tmp/tools/*

# kustomize: 릴리스의 checksums.txt로 검증한다(태그 이름이 kustomize/v<버전>이라 /를 %2F로 쓴다).
RUN curl -fsSLO "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv${KUSTOMIZE_VERSION}/kustomize_v${KUSTOMIZE_VERSION}_linux_${TARGETARCH}.tar.gz" \
 && curl -fsSLO "https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv${KUSTOMIZE_VERSION}/checksums.txt" \
 && sha256sum --check --ignore-missing checksums.txt \
 && tar -xzf "kustomize_v${KUSTOMIZE_VERSION}_linux_${TARGETARCH}.tar.gz" -C /usr/local/bin kustomize \
 && rm -rf /tmp/tools/*

# kubeconform: 릴리스의 CHECKSUMS로 검증한다.
RUN curl -fsSLO "https://github.com/yannh/kubeconform/releases/download/v${KUBECONFORM_VERSION}/kubeconform-linux-${TARGETARCH}.tar.gz" \
 && curl -fsSLO "https://github.com/yannh/kubeconform/releases/download/v${KUBECONFORM_VERSION}/CHECKSUMS" \
 && sha256sum --check --ignore-missing CHECKSUMS \
 && tar -xzf "kubeconform-linux-${TARGETARCH}.tar.gz" -C /usr/local/bin kubeconform \
 && rm -rf /tmp/tools/*

# istioctl: 압축 파일 옆의 .sha256으로 검증한다.
RUN curl -fsSLO "https://github.com/istio/istio/releases/download/${ISTIOCTL_VERSION}/istioctl-${ISTIOCTL_VERSION}-linux-${TARGETARCH}.tar.gz" \
 && curl -fsSLO "https://github.com/istio/istio/releases/download/${ISTIOCTL_VERSION}/istioctl-${ISTIOCTL_VERSION}-linux-${TARGETARCH}.tar.gz.sha256" \
 && sha256sum --check "istioctl-${ISTIOCTL_VERSION}-linux-${TARGETARCH}.tar.gz.sha256" \
 && tar -xzf "istioctl-${ISTIOCTL_VERSION}-linux-${TARGETARCH}.tar.gz" -C /usr/local/bin istioctl \
 && rm -rf /tmp/tools/*

# argocd CLI: 릴리스의 cli_checksums.txt로 검증한다.
RUN curl -fsSLO "https://github.com/argoproj/argo-cd/releases/download/v${ARGOCD_VERSION}/argocd-linux-${TARGETARCH}" \
 && curl -fsSLO "https://github.com/argoproj/argo-cd/releases/download/v${ARGOCD_VERSION}/cli_checksums.txt" \
 && sha256sum --check --ignore-missing cli_checksums.txt \
 && install -m 0755 "argocd-linux-${TARGETARCH}" /usr/local/bin/argocd \
 && rm -rf /tmp/tools/*

# Argo Rollouts kubectl 플러그인: 릴리스의 argo-rollouts-checksums.txt로 검증한다.
RUN curl -fsSLO "https://github.com/argoproj/argo-rollouts/releases/download/v${ARGO_ROLLOUTS_VERSION}/kubectl-argo-rollouts-linux-${TARGETARCH}" \
 && curl -fsSLO "https://github.com/argoproj/argo-rollouts/releases/download/v${ARGO_ROLLOUTS_VERSION}/argo-rollouts-checksums.txt" \
 && sha256sum --check --ignore-missing argo-rollouts-checksums.txt \
 && install -m 0755 "kubectl-argo-rollouts-linux-${TARGETARCH}" /usr/local/bin/kubectl-argo-rollouts \
 && rm -rf /tmp/tools/*

# kyverno CLI: 릴리스의 checksums.txt로 검증한다(파일 이름의 CPU 표기는 x86_64·arm64).
RUN case "${TARGETARCH}" in amd64) KYVERNO_ARCH=x86_64 ;; arm64) KYVERNO_ARCH=arm64 ;; esac \
 && curl -fsSLO "https://github.com/kyverno/kyverno/releases/download/v${KYVERNO_VERSION}/kyverno-cli_v${KYVERNO_VERSION}_linux_${KYVERNO_ARCH}.tar.gz" \
 && curl -fsSLO "https://github.com/kyverno/kyverno/releases/download/v${KYVERNO_VERSION}/checksums.txt" \
 && sha256sum --check --ignore-missing checksums.txt \
 && tar -xzf "kyverno-cli_v${KYVERNO_VERSION}_linux_${KYVERNO_ARCH}.tar.gz" -C /usr/local/bin kyverno \
 && rm -rf /tmp/tools/*

# cosign: 릴리스의 cosign_checksums.txt로 검증한다.
RUN curl -fsSLO "https://github.com/sigstore/cosign/releases/download/v${COSIGN_VERSION}/cosign-linux-${TARGETARCH}" \
 && curl -fsSLO "https://github.com/sigstore/cosign/releases/download/v${COSIGN_VERSION}/cosign_checksums.txt" \
 && sha256sum --check --ignore-missing cosign_checksums.txt \
 && install -m 0755 "cosign-linux-${TARGETARCH}" /usr/local/bin/cosign \
 && rm -rf /tmp/tools/*

# syft: 릴리스의 syft_<버전>_checksums.txt로 검증한다.
RUN curl -fsSLO "https://github.com/anchore/syft/releases/download/v${SYFT_VERSION}/syft_${SYFT_VERSION}_linux_${TARGETARCH}.tar.gz" \
 && curl -fsSLO "https://github.com/anchore/syft/releases/download/v${SYFT_VERSION}/syft_${SYFT_VERSION}_checksums.txt" \
 && sha256sum --check --ignore-missing "syft_${SYFT_VERSION}_checksums.txt" \
 && tar -xzf "syft_${SYFT_VERSION}_linux_${TARGETARCH}.tar.gz" -C /usr/local/bin syft \
 && rm -rf /tmp/tools/*

# trivy: 릴리스의 trivy_<버전>_checksums.txt로 검증한다(파일 이름의 CPU 표기는 64bit·ARM64).
RUN case "${TARGETARCH}" in amd64) TRIVY_ARCH=64bit ;; arm64) TRIVY_ARCH=ARM64 ;; esac \
 && curl -fsSLO "https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-${TRIVY_ARCH}.tar.gz" \
 && curl -fsSLO "https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_checksums.txt" \
 && sha256sum --check --ignore-missing "trivy_${TRIVY_VERSION}_checksums.txt" \
 && tar -xzf "trivy_${TRIVY_VERSION}_Linux-${TRIVY_ARCH}.tar.gz" -C /usr/local/bin trivy \
 && rm -rf /tmp/tools/*

# AWS CLI v2: AWS 공식 설치 파일의 버전 지정 주소에서 받는다(파일 이름의 CPU 표기는 x86_64·aarch64).
RUN case "${TARGETARCH}" in amd64) AWS_ARCH=x86_64 ;; arm64) AWS_ARCH=aarch64 ;; esac \
 && curl -fsSLo awscliv2.zip "https://awscli.amazonaws.com/awscli-exe-linux-${AWS_ARCH}-${AWSCLI_VERSION}.zip" \
 && unzip -q awscliv2.zip \
 && ./aws/install \
 && rm -rf /tmp/tools/*

# Session Manager plugin: AWS 공식 .deb를 CPU에 맞는 경로에서 받아 설치한다.
RUN case "${TARGETARCH}" in amd64) SSM_ARCH=ubuntu_64bit ;; arm64) SSM_ARCH=ubuntu_arm64 ;; esac \
 && curl -fsSLo session-manager-plugin.deb "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/${SSM_ARCH}/session-manager-plugin.deb" \
 && dpkg -i session-manager-plugin.deb \
 && rm -rf /tmp/tools/*

# 임시 폴더를 지우고 기본 작업 위치를 루트로 돌린다.
WORKDIR /
RUN rm -rf /tmp/tools
