# direktiv-dev
 Sample dev env on Windows connecting to Direktiv Server on Multipass Ubuntu instance

## 1. Install Multipass on Windows
- Follow https://multipass.run/install
- Check
```
multipass launch --name MyUbuntu
multipass exec MyUbuntu -- lsb_release -a
multipass list
multipass stop MyUbuntu
multipass delete MyUbuntu
multipass purge
multipass find
```

## 2. Run Direktiv on Multipass:
- Launch Ditectiv linux image
```
multipass launch --cpus 6 --disk 20G --memory 12G --name direktiv --cloud-init https://raw.githubusercontent.com/direktiv/direktiv/main/build/docker/all/multipass/init.yaml
```

- Start bash shell on the VM
```
multipass exec direktiv -- /bin/bash
```

- Check IP
```
multipass info direktiv
```
-->
```
PS C:\Users\Andy.Nguyen> multipass info direktiv
Name:           direktiv
State:          Running
IPv4:           172.19.192.81
                10.42.0.0
                10.42.0.1
Release:        Ubuntu 22.04.3 LTS
Image hash:     5bed3f233c24 (Ubuntu 22.04 LTS)
CPU(s):         6
Load:           0.78 0.68 0.35
Disk usage:     5.4GiB out of 48.4GiB
Memory usage:   2.4GiB out of 23.5GiB
Mounts:         --
```

- Test
```
# create namespace 'test'
curl -X PUT http://172.19.192.81/api/namespaces/test

# create the workflow file
cat > helloworld.yml <<- EOF
direktiv_api: workflow/v1
functions:
- id: get
  type: reusable
  image: gcr.io/direktiv/functions/bash:1.0
states:
- id: getter
  type: action
  action:
    function: get
    input:
      commands:
      - command: echo Hello
EOF

# upload flow
curl -X PUT  --data-binary @helloworld.yaml "http://172.19.192.81/api/namespaces/test/tree/test.yaml?op=create-workflow"

# execute flow (initial call will be slightly slower than subsequent calls)
curl "http://172.19.192.81/api/namespaces/test/tree/test?op=wait"
```

## 3. Install Direktive Cli
- Download
https://github.com/direktiv/direktiv/releases/download/v0.8.0/direktivctl-windows.exe

- Change file name to direktivctl.exe and move to the location that we like.
- Set PATH to point to it.

## 4. Create a dev working folder on Windows
- Create a protect folder such as hello-echo
- cd inside it
- Create .direktive.yaml file. my-direktiv.server is the ip of direktive multipass instance
```
addr: "http://my-direktiv.server"
namespace: "hello-echo"
```
- Create namespace "hello-echo" (in bash shell to direktiv instance)
```
curl -X PUT http://172.19.192.81/api/namespaces/hello-echo
```
-->
```
ubuntu@direktiv:~$ curl -X PUT http://172.19.192.81/api/namespaces/hello-echo
{
  "namespace": {
    "createdAt": "2023-10-28T00:01:59.050111Z",
    "updatedAt": "2023-10-28T00:01:59.050111Z",
    "name": "hello-echo",
    "oid": "baf0b2b3-8775-4eef-b697-7de2210684cb",
    "notes": {}
  }
}
```

## 5.Work with working folder (hello-echo)
- In hello-echo folder create hello-echo.yaml with this content
```
direktiv_api: workflow/v1
functions:
- id: get
  type: reusable
  image: gcr.io/direktiv/functions/bash:1.0
states:
- id: getter
  type: action
  action:
    function: get
    input:
      commands:
      - command: echo Hello from echo
```

- Use direktivctl to push the hello-echo workflow to the Direktiv server
```
direktivctl workflows push hello-echo.yaml -a http://172.19.192.81 -n hello-echo
```
- Execute the workflow
```
direktivctl workflows exec hello-echo.yaml -a http://172.19.192.81 -n hello-echo
```
```
curl "http://172.19.192.81/api/namespaces/hello-echo/tree/hello-echo?op=wait"
```