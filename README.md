# Running Moon on Apple Silicon (M1)

This is a step-by-step guide showing how to run [Moon](https://aerokube.com/moon/) browser automation software on Apple Silicon computer. Detailed Moon documentation can be found [here](https://aerokube.com/moon/latest).

**Prerequisites:**
  * Apple Silicon computer
  * [Docker](https://docs.docker.com/desktop/mac/install/) installed 

**Installation steps:**

1. Install [minikube](https://github.com/kubernetes/minikube) by downloading the binary from Github [releases page](https://github.com/kubernetes/minikube/releases).

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

2. Check that you now have `minikube` command:
```
$ minikube version
minikube version: v1.25.2
commit: 362d5fdc0a3dbee389b3d3f1034e8023e72bd3a7
```

3. Start local Kubernetes cluster with Minikube.

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

4. Enable [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) support in Minikube:
```
$ minikube addons enable ingress
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

5. Install [kubectl](https://github.com/kubernetes/kubectl) (Kubernetes command line client) for Apple Silicon: https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-kubectl-binary-with-curl-on-macos. It is better to install the same `kubectl` version as Kubernetes cluster version. In our case we install version `1.23.3`:

```
$ curl -LO https://dl.k8s.io/release/v1.23.3/bin/darwin/arm64/kubectl
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

6. Check that you now have `kubectl` command:
```
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:25:17Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"darwin/arm64"}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.3", GitCommit:"816c97ab8cff8a1c72eccca1026f7820e93e0d25", GitTreeState:"clean", BuildDate:"2022-01-25T21:19:12Z", GoVersion:"go1.17.6", Compiler:"gc", Platform:"linux/arm64"}
```

7. Install [Helm](https://helm.sh/) package manager:
```
$ curl -Ls https://get.helm.sh/helm-v3.8.2-darwin-arm64.tar.gz | tar x --strip-components 1 darwin-arm64/helm
```
```
$ chmod +x helm
```
```
$ sudo mv helm /usr/local/bin
```

7. Check that you now have `helm` command:
```
$ helm version
version.BuildInfo{Version:"v3.8.2", GitCommit:"6e3701edea09e5d55a8ca2aae03a68917630e91b", GitTreeState:"clean", GoVersion:"go1.17.5"}
```

8. Choose a domain name for Moon cluster and add it to `/etc/hosts` file. Default name is `moon.aerokube.local`.

```
$ sudo bash -c "echo 127.0.0.1 moon.aerokube.local >> /etc/hosts"
```

9. Prerare `values.yaml` file with Moon installation configuration:
```
$ cat >values.yaml<< EOF
deployment:
  moonRepo: aandryashin/moon
  moonConfRepo: aandryashin/moon-conf
  moonUIRepo: aandryashin/moon-ui
  moonTag: 2.2.0
  moonConfTag: 2.2.0
  moonUITag: 2.0.2
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
      webkit:
        default: 613.1.6.1
        repository: quay.io/browser/webkit
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

10. Add Helm repository for Moon:
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

11. Install [Moon](https://aerokube.com/moon) with Helm:
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

12. Start [Minikube tunnel](https://minikube.sigs.k8s.io/docs/handbook/accessing/#using-minikube-tunnel) in a separate terminal and keep it running:
```
$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress moon requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service moon.
```

13. Open Moon user interface in browser: http://moon.aerokube.local/

14. Use the following URLs in your code:

* [Selenium](https://selenium.dev/) tests URL:

`http://moon.aerokube.local/wd/hub`

Only Chromium, Firefox, and Webkit browsers are available for Apple Silicon.

* [Playwright](https://playwright.dev/) tests URLs:

`ws://moon.aerokube.local/playwright/chromium/playwright-1.22.0?headless=false`

`ws://moon.aerokube.local/playwright/firefox/playwright-1.22.0?headless=false`

`ws://moon.aerokube.local/playwright/webkit/playwright-1.22.0?headless=false`


## Example Commands and Code

* How to create a new [Selenium](https://selenium.dev/) session with `curl`:

```
$ curl -H'Content-Type: application/json' http://moon.aerokube.local/wd/hub/session -d'{"capabilities":{"alwaysMatch":{"browserName":"chromium", "moon:options":{"sessionTimeout":"5m", "name": "session created by curl command", "labels":{"manual":"true"}}}}}'
{"value":{"capabilities":{"acceptInsecureCerts":false,"browserName":"chromium","browserVersion":"100.0.4896.127","chrome":{"chromedriverVersion":"100.0.4896.127 (ff0d0695743e65305d7194f9bd309e5e1c824aa0-refs/branch-heads/4896_88@{#4})","userDataDir":"/tmp/.org.chromium.Chromium.PmpFyl"},"goog:chromeOptions":{"debuggerAddress":"localhost:44625"},"networkConnectionEnabled":false,"pageLoadStrategy":"normal","platformName":"linux","proxy":{},"se:cdp":"ws://moon.aerokube.local/wd/hub/session/chromium-100-0-4896-127-0-b5122743-ec88-46ed-9ec2-f0c3032ead0b/aerokube/devtools","se:cdpVersion":null,"setWindowRect":true,"strictFileInteractability":false,"timeouts":{"implicit":0,"pageLoad":300000,"script":30000},"unhandledPromptBehavior":"dismiss and notify","webauthn:extension:credBlob":true,"webauthn:extension:largeBlob":true,"webauthn:virtualAuthenticators":true},"sessionId":"chromium-100-0-4896-127-0-b5122743-ec88-46ed-9ec2-f0c3032ead0b"}}
```
```
curl -H'Content-Type: application/json' http://moon.aerokube.local/wd/hub/session -d'{"capabilities":{"alwaysMatch":{"browserName":"firefox", "moon:options":{"sessionTimeout":"5m", "name": "session created by curl command", "labels":{"manual":"true"}}}}}'
{"value":{"capabilities":{"acceptInsecureCerts":false,"browserName":"firefox","browserVersion":"100.0","moon:options":{"labels":{"manual":"true"},"name":"session created by curl command","sessionTimeout":"5m"},"moz:accessibilityChecks":false,"moz:buildID":"20220428192727","moz:geckodriverVersion":"0.30.0","moz:headless":false,"moz:processID":23,"moz:profile":"/tmp/rust_mozprofilesfk993","moz:shutdownTimeout":60000,"moz:useNonSpecCompliantPointerOrigin":false,"moz:webdriverClick":true,"pageLoadStrategy":"normal","platformName":"linux","platformVersion":"5.10.104-linuxkit","proxy":{},"setWindowRect":true,"strictFileInteractability":false,"timeouts":{"implicit":0,"pageLoad":300000,"script":30000},"unhandledPromptBehavior":"dismiss and notify"},"sessionId":"firefox-100-0-0-0-4c48256f-4152-4851-9b98-652dea95ee70"}}
```

When running these commands for first time, session creation can takes more time. This is because of Moon browser images being downloaded.

* Example [Playwright](https://playwright.dev/) tests:

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

## Moon Configuration Objects

1. Listing license keys:
```
$ kubectl get licenses
NAME   LICENSEE   SESSIONS   EXPIRES   STATUS   NAMESPACE
moon   Default    4          Never     Ok       moon
```
2. Listing Moon global configuration objects:
```
$ kubectl get config -n moon
NAME      AGE
default   37m
```
3. Listing Moon browsers sets:
```
$ kubectl get browsers -n moon
NAME      AGE
default   38m
```
4. Listing Moon device sets:
```
$ kubectl get devices -n moon
NAME      AGE
default   39m
```
To view contents of every such object - append `-o yaml` to the command above.


## Troubleshooting

To verify everything is installed correctly:

1. Make sure you have an active Ingress object:
```
$ kubectl get ingress -n moon
NAME   CLASS   HOSTS                 ADDRESS        PORTS   AGE
moon   nginx   moon.aerokube.local   192.168.49.2   80      47s
```

2. Make sure you have an running [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/):
```
$ kubectl get deploy -n moon
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
moon   2/2     2            2           74s
```

3. Make sure Moon [pods](https://kubernetes.io/docs/concepts/workloads/pods/) are in `Running` state:
```
$ kubectl get po -n moon
NAME                    READY   STATUS    RESTARTS   AGE
moon-7c9f78f96b-4sfll   3/3     Running   0          99s
moon-7c9f78f96b-cgqwp   3/3     Running   0          99s
```

4. If something is still not working - take a look at Moon logs:
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

The same command for following Moon logs while running your tests:
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

An example Selenium session log looks like this:
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
