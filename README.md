# Deploy Application to K8s with Jenkins, ArgoCD on Windows
## 1. Install Docker Desktop and K8s single-node
- Link hướng dẫn cài đặt: [Docker Desktop](https://docs.docker.com/desktop/install/windows-install/)
- Sau khi cài đặt Docker Desktop thành công thì sẽ tiến hành Enable Kubernet trên giao diện Docker Desktop. Quá trình sẽ hoàn tất sau vài phút
- Có thể verify bằng command:
```
kubectl get node
```
- Với trường hợp trên máy cá nhân đã config thông tin của 1 k8s cluster khác (ví dụ dự án của công ty,..) thì cần switch sang context khác cho đúng với cluster trên máy cá nhân để thao tác
    <ul>
Liệt kê các context hiện có:
```
kubectl get context
```
Switch sang context cần check:
```
kubectl config use-context {context-name}
```
</ul>

## 2. Install Jenkins by Docker on Windowns
- Tạo Dockerfile có nội dung như sau
```
FROM jenkins/jenkins:lts
USER root
RUN apt-get update -qq \
    && apt-get install -qqy apt-transport-https ca-certificates curl gnupg2 software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update  -qq \
    && apt-get -y install docker-ce
RUN usermod -aG docker jenkins
```
- Build Images Jenkins:
```
docker image build -t custom-jenkins-docker .
```
- Run container Jenkins:
```
docker run -it  --name jenkins -d -p 8080:8080 -p 50000:50000 -v //var/run/docker.sock:/var/run/docker.sock -v jenkins_home:/var/jenkins_home --restart unless-stopped custom-jenkins-docker
```
- Sau khi container chạy xong thì bạn có thể truy cập vào web thông qua địa chỉ : *http://localhost:8080/*
    <ul>
Để lấy password cho lần đầu tiên đăng nhập:
```
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
</ul>

## 3. Install ArgoCD
- Cài đặt ArgoCD sử dụng manifest file
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
- Kiểm tra tất cả các tài nguyên được tạo trên namespace argocd
```
 kubectl get all -n argocd
```
- Để truy cập được vào argocd thì mình sử dụng expose service argocd-server type NodePort:
```
kubectl edit service argocd-server -n argocd
```
<ul>
Chỉnh sửa spec.type từ ClusterIP thành NodePort
</ul>

- Lấy thông tin Port để truy cập vào ArgoCD:
```
kubectl describe svc argocd-server -n argocd |grep NodePort
```
- Lấy mật khẩu truy cập vào web ArgoCD với link: http://localhost:{port} với user mặc định là *admin*
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode ; echo
```
<ul>
Sau khi login thì nên đổi mật khẩu admin.
</ul>