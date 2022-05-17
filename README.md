## Install minikube from: https://github.com/kubernetes/minikube/releases

```
$ curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.25.2/minikube-darwin-arm64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 70.6M  100 70.6M    0     0  7785k      0  0:00:09  0:00:09 --:--:-- 8124k
```
```
$ chmod +x minikube
```
```
$ sudo mv minikube /usr/local/bin
```
```
$ minikube version
minikube version: v1.25.2
commit: 362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
```

## Start minikube

```
$ minikube start --cpus=4 --memory=8Gi --driver=docker
😄  minikube v1.25.2 on Darwin 12.3.1 (arm64)
    ▪ MINIKUBE_ACTIVE_DOCKERD=minikube
✨  Using the docker driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.23.3 preload ...
    > preloaded-images-k8s-v17-v1...: 419.07 MiB / 419.07 MiB  100.00% 9.21 MiB
🔥  Creating docker container (CPUs=4, Memory=8192MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## Enable ingress
```
$ minikube addons enable ingress
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

## Install kubectl (Apple Silicon) https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-kubectl-binary-with-curl-on-macos

It is better to install the same version as Kubernetes cluster, in our case it is v1.23.3

```
$ curl -LO https://dl.k8s.io/release/v1.23.3//bin/darwin/arm64/kubectl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   154  100   154    0     0    858      0 --:--:-- --:--:-- --:--:--   890
100 52.8M  100 52.8M    0     0  9713k      0  0:00:05  0:00:05 --:--:-- 10.1M
```
```
$ chmod +x kubectl 
```
```
$ sudo mv kubectl /usr/local/bin
```
```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:25:17Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"darwin/arm64"}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:19:12Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/arm64"}
```

## Install helm 
```
$ curl -Ls https://get.helm.sh/helm-v3.8.2-darwin-arm64.tar.gz | tar x --strip-components 1 darwin-arm64/helm
```
```
$ chmod +x helm
```
```
$ sudo mv helm /usr/local/bin
```
```
$ helm version
version.BuildInfo{Version:"v3.8.2", GitCommit:"6e3701edea09e5d55a8ca2aae03a68917630e91b", GitTreeState:"clean", GoVersion:"go1.17.5"}
```

## Install moon https://aerokube.com/moon/latest/

Choose moon domain name and add it to /etc/hosts file, by default it is moon.aerokube.local

```
$ sudo bash -c "echo 127.0.0.1 moon.aerokube.local >>/etc/hosts"
```

Prerare helm values.yaml file with moon configuration
```
$ cat >values.yaml<< EOF
deployment:
  moonRepo: aandryashin/moon
  moonConfRepo: aandryashin/moon-conf
  moonUIRepo: aandryashin/moon-ui
configs:
  default:
    containers:
      ca-certs:
        repository: aandryashin/ca-certs
      defender:
        repository: aandryashin/defender
      video-recorder:
        repository: aandryashin/video-recorder
      vnc-server:
        repository: aandryashin/vnc-server
      x-server:
        repository: aandryashin/x-server
browsers:
  default:
    selenium:
      chromium:
        default: 100.0.4896.127-0
        repository: quay.io/browser/chromium
      firefox:
        default: 100.0.0-0
        repository: quay.io/browser/firefox
      safari:
        default: 15.0-0
        repository: quay.io/browser/safari
      chrome: null
      MicrosoftEdge: null
      opera: null
    playwright:
      chromium:
        repository: aandryashin/playwright-chromium
      firefox:
        repository: aandryashin/playwright-firefox
      webkit:
        repository: aandryashin/playwright-webkit
      chrome: null
    cypress:
      electron: null
      chromium: null
      chrome: null
      firefox: null
      edge: null
    devtools:
      chrome: null
EOF
```

Add helm aerokube repository
```
$ helm repo add aerokube https://charts.aerokube.com/
"aerokube" has been added to your repositories
```
```
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "aerokube" chart repository
Update Complete. ⎈Happy Helming!⎈
```

Install moon
```
$ helm upgrade --install --create-namespace --set ingress.host=moon.aerokube.local -f values.yaml -n moon moon aerokube/moon2
Release "moon" does not exist. Installing it now.
NAME: moon
LAST DEPLOYED: Tue May 17 17:57:01 2022
NAMESPACE: moon
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

Start minikube tunnel in separate terminal and keep it running
```
$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress moon requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service moon.
```

Now you are able to open http://moon.aerokube.local in browser

Web interface url
  http://moon.aerokube.local

Selenium tests url, please note for Apple Silicon only chromium, firefox, and webkit browsers are available.
  http://moon.aerokube.local/wd/hub

Playwright tests urls
  ws://moon.aerokube.local/playwright/chromium/playwright-1.22.0?headless=false
  ws://moon.aerokube.local/playwright/firefox/playwright-1.22.0?headless=false
  ws://moon.aerokube.local/playwright/webkit/playwright-1.22.0?headless=false


## Examples (look to capabilities tab for sample code)

New selenium session, first time session creation takes a time due browser image downloading

```
$ curl -H'Content-Type: application/json' http://moon.aerokube.local/wd/hub/session -d'{"capabilities":{"alwaysMatch":{"browserName":"chromium", "moon:options":{"sessionTimeout":"5m", "name": "session created by curl command", "labels":{"manual":"true"}}}}}'
{"value":{"capabilities":{"acceptInsecureCerts":false,"browserName":"chromium","browserVersion":"100.0.4896.127","chrome":{"chromedriverVersion":"100.0.4896.127 (ff0d0695743e65305d7194f9bd309e5e1c824aa0-refs/branch-heads/4896_88@{#4})","userDataDir":"/tmp/.org.chromium.Chromium.PmpFyl"},"goog:chromeOptions":{"debuggerAddress":"localhost:44625"},"networkConnectionEnabled":false,"pageLoadStrategy":"normal","platformName":"linux","proxy":{},"se:cdp":"ws://moon.aerokube.local/wd/hub/session/chromium-100-0-4896-127-0-b5122743-ec88-46ed-9ec2-f0c3032ead0b/aerokube/devtools","se:cdpVersion":null,"setWindowRect":true,"strictFileInteractability":false,"timeouts":{"implicit":0,"pageLoad":300000,"script":30000},"unhandledPromptBehavior":"dismiss and notify","webauthn:extension:credBlob":true,"webauthn:extension:largeBlob":true,"webauthn:virtualAuthenticators":true},"sessionId":"chromium-100-0-4896-127-0-b5122743-ec88-46ed-9ec2-f0c3032ead0b"}}
```
```
curl -H'Content-Type: application/json' http://moon.aerokube.local/wd/hub/session -d'{"capabilities":{"alwaysMatch":{"browserName":"firefox", "moon:options":{"sessionTimeout":"5m", "name": "session created by curl command", "labels":{"manual":"true"}}}}}'
{"value":{"capabilities":{"acceptInsecureCerts":false,"browserName":"firefox","browserVersion":"100.0","moon:options":{"labels":{"manual":"true"},"name":"session created by curl command","sessionTimeout":"5m"},"moz:accessibilityChecks":false,"moz:buildID":"20220428192727","moz:geckodriverVersion":"0.30.0","moz:headless":false,"moz:processID":23,"moz:profile":"/tmp/rust_mozprofilesfk993","moz:shutdownTimeout":60000,"moz:useNonSpecCompliantPointerOrigin":false,"moz:webdriverClick":true,"pageLoadStrategy":"normal","platformName":"linux","platformVersion":"5.10.104-linuxkit","proxy":{},"setWindowRect":true,"strictFileInteractability":false,"timeouts":{"implicit":0,"pageLoad":300000,"script":30000},"unhandledPromptBehavior":"dismiss and notify"},"sessionId":"firefox-100-0-0-0-4c48256f-4152-4851-9b98-652dea95ee70"}}
```

Simplest playwright tests:

```
const { chromium } = require('playwright');

(async () => {
    const browser = await chromium.connect({ timeout: 0, wsEndpoint: `ws://moon.aerokube.local/playwright/chromium/playwright-1.22.0?headless=false` });
    const page = await browser.newPage();
    await page.goto('https://aerokube.com/moon/');
    await page.screenshot({ path: `screenshot.png` });
    await browser.close();
})();
```
```
const { firefox } = require('playwright');

(async () => {
    const browser = await firefox.connect({ timeout: 0, wsEndpoint: `ws://moon.aerokube.local/playwright/firefox/playwright-1.22.0?headless=false` });
    const page = await browser.newPage();
    await page.goto('https://aerokube.com/moon/');
    await page.screenshot({ path: `screenshot.png` });
    //await browser.close();
})();
```
```
const { webkit } = require('playwright');

(async () => {
    const browser = await webkit.connect({ timeout: 0, wsEndpoint: `ws://moon.aerokube.local/playwright/webkit/playwright-1.22.0?headless=false` });
    const page = await browser.newPage();
    await page.goto('https://aerokube.com/moon/');
    await page.screenshot({ path: `screenshot.png` });
    //await browser.close();
})();
```

## Configuration objects
Add -o yaml ooption to see content
```
$ kubectl get licenses
NAME   LICENSEE   SESSIONS   EXPIRES   STATUS   NAMESPACE
moon   Default    4          Never     Ok       moon
```
```
$ kubectl get config -n moon
NAME      AGE
default   37m
```
```
$ kubectl get browsers -n moon
NAME      AGE
default   38m
```
```
$ kubectl get devices -n moon
NAME      AGE
default   39m
```

## Troubleshutting

Verify installation
```
$ kubectl get ingress -n moon
NAME   CLASS   HOSTS                 ADDRESS        PORTS   AGE
moon   nginx   moon.aerokube.local   192.168.49.2   80      47s
```
```
$ kubectl get deploy -n moon
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
moon   2/2     2            2           74s
```
```
$ kubectl get po -n moon
NAME                    READY   STATUS    RESTARTS   AGE
moon-7c9f78f96b-4sfll   3/3     Running   0          99s
moon-7c9f78f96b-cgqwp   3/3     Running   0          99s
```

Look to moon logs
```
$ kubectl logs -n moon -c moon -l app=moon --tail=-1
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: license controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: license: Default [4], expires: Never
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: parallel sessions limit: [4]
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: deviceset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: deviceset controller: devices "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: browserset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: browserset controller: browsers "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: config controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: config controller: config "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: quota controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: quota controller: quota "moon": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: started browser counter for namespace "moon"
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: listening on :4444
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: license controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: license: Default [4], expires: Never
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: parallel sessions limit: [4]
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: deviceset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: deviceset controller: devices "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: browserset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: browserset controller: browsers "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: config controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: config controller: config "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: quota controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: quota controller: quota "moon": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: started browser counter for namespace "moon"
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: listening on :4444
```

Follow moon logs
```
$ kubectl logs -n moon -c moon -l app=moon -f
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: deviceset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: deviceset controller: devices "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: browserset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: browserset controller: browsers "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: config controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: config controller: config "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: quota controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: quota controller: quota "moon": added
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: started browser counter for namespace "moon"
2022/05/17 14:57:03 moon-7c9f78f96b-cgqwp: listening on :4444
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: deviceset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: deviceset controller: devices "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: browserset controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: browserset controller: browsers "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: config controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: config controller: config "default": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: quota controller: started
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: quota controller: quota "moon": added
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: started browser counter for namespace "moon"
2022/05/17 14:57:03 moon-7c9f78f96b-4sfll: listening on :4444
```

Session lifecicle
```
$ kubectl logs -n moon -c moon -l app=moon -f
2022/05/17 15:22:18 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: starting new session
2022/05/17 15:22:18 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: creating config map
2022/05/17 15:22:18 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: config map created
2022/05/17 15:22:18 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: creating pod
2022/05/17 15:22:18 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: pod created
2022/05/17 15:22:18 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: waiting pod
2022/05/17 15:22:21 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: pod is up
2022/05/17 15:22:21 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: waiting driver
2022/05/17 15:22:21 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: driver is up
2022/05/17 15:22:21 moon-7c9f78f96b-cgqwp: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: session created: 3.209s
2022/05/17 15:22:31 moon-7c9f78f96b-4sfll: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: deleting session
2022/05/17 15:22:31 moon-7c9f78f96b-4sfll: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: deleting pod
2022/05/17 15:22:31 moon-7c9f78f96b-4sfll: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: pod deleted
2022/05/17 15:22:31 moon-7c9f78f96b-4sfll: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: deleting configmap
2022/05/17 15:22:31 moon-7c9f78f96b-4sfll: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: configmap deleted
2022/05/17 15:22:31 moon-7c9f78f96b-4sfll: moon: chromium-100-0-4896-127-0-6f441d90-0f45-46e7-9a1e-ad7a4f3767b8: session deleted
```










































