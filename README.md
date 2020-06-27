# fedora-silverblue-custom

#### Build image

From upstream: https://github.com/coreos/coreos-assembler

```
cosa() {
   env | grep COREOS_ASSEMBLER
   set -x
   podman run --rm -ti --security-opt label=disable --privileged -w /srv                            \
              --uidmap=10000:0:1 --uidmap=0:1:10000 --uidmap 10001:10001:55536                      \
              -v ${PWD}:/srv/ --device /dev/kvm --device /dev/fuse                                  \
              --tmpfs /tmp -v /var/tmp:/var/tmp --name cosa-silverblue                              \
              ${COREOS_ASSEMBLER_CONFIG_GIT:+-v $COREOS_ASSEMBLER_CONFIG_GIT:/srv/src/config/:ro}   \
              ${COREOS_ASSEMBLER_GIT:+-v $COREOS_ASSEMBLER_GIT/src/:/usr/lib/coreos-assembler/:ro}  \
              ${COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS}                                            \
              ${COREOS_ASSEMBLER_CONTAINER:-quay.io/coreos-assembler/coreos-assembler:latest} "$@"
   rc=$?; set +x; return $rc
}
```

Check out config
```
cosa init https://github.com/randomcoww/fedora-silverblue-custom.git --force
```

Run build
```
cosa clean && cosa fetch && cosa build metal && cosa buildextend-live --no-pxe
```

Embed ignition from https://github.com/randomcoww/terraform-infra
```
curl http://127.0.0.1:8080/ignition?ign=desktop \
  | sudo coreos-installer iso embed builds/latest/x86_64/fedora-silverblue-*-live.x86_64.iso -o desktop.iso
```

Replace on running system
```
curl http://127.0.0.1:8080/ignition?ign=desktop \
   | sudo coreos-installer iso embed /dev/sda --force
```
