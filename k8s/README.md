# volbench on k8s
Standardized benchmarking for volumes hosted on kubernetes

## how does it look like
To guarantee the simplest usage possible, the k8s version is composed of two files:
- ```volbench.sh``` which is the actual bash script calling ```fio``` with the relevant benchmarking profile
- ```volbench.yaml``` which is a standard yaml configuration file 

Note: ```volbench.sh``` has very few changes from the CLI version to run smoothly as a pod on k8s.

## what does ```volbench.yaml``` do
When applying the file towards a k8s cluster, it will create: 
- a namespace called ```volbench```
- a persistent volume claim called ```volbenchtemp1``` linked to the created namespace
- a pod called ```volbench-runner``` within the created namespace consuming the persistent volume

Here is the full yaml file:
```yaml 
apiVersion: v1
kind: Namespace
metadata:
  name: volbench
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: volbenchtemp1
  namespace: volbench
spec:
  storageClassName: "fast"
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: volbench-runner
  namespace: volbench
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["/bin/sh"]
      args: ["-c", 'apk update && apk add git fio bash --no-cache; git clone --single-branch --branch containerized http://github.com/rovandep/volbench.git; /volbench/k8s/volbench.sh; sleep 36000']
      volumeMounts:
        - mountPath: /tmp
          name: tmp1
      env:
      - name: FIO_files
        value: "/tmp/volbenchtest1 /tmp/volbenchtest2"
      - name: FIO_size
        value: "1MB"
      - name: FIO_ramptime
        value: "1"
      - name: FIO_runtime
        value: "5"
      - name: FIO_rwmixread
        value: "75"
      - name: FIO_fdatasync
        value: "0"
  volumes:
    - name: tmp1
      persistentVolumeClaim:
        claimName: volbenchtemp1
```

Note: there are environment variables defined under ```env:``` within the YAML configuration file defining the ```fio``` benchmarking profile.

Here is an overview of the ones shipped in the configuraiton file that allows a very fast run to check that all is working as expected.


## how to use it
In a nutshell, from the above YAML output:
- the ```storageClassName``` might need to be change to match the existing ```storageClassName``` on the targeted k8s.  
- the ```env``` field might need to be change to match the desired ```fio``` benchmarking profile.

Then:
[![asciicast](https://asciinema.org/a/407266.svg)](https://asciinema.org/a/407266)

<script id="asciicast-407266" src="https://asciinema.org/a/407266.js" async></script>