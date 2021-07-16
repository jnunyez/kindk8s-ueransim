# kindk8s-ueransim

config maps and deployments of 5G-NR emulated networks in k8s 

## Rationale

This repo leverages [ueransim](https://github.com/aligungr/UERANSIM) to deploy 5G gNB and a 5G UE inside k8s using [kind](https://kind.sigs.k8s.io).

## Quickstart

1. Generate open5gs and webui base image, as detailed [here](http://github.com/jnunyez/build-open5gs).

2. Deploy open5gs and prerequisites in k8s. The manifests for deploying open5gs in k8s are available [here](https://github.com/jnunyez/kindk8s-open5gs).

3. Build the UERANSIM image. The repo to build this image is available in [here](https://github.com/build-ueransim).

4. Deploy a 5G gNB and a 5G UE on k8s to start testing: 

   ```
   deploy-ueransim
   ```

## ToDo

- Generalize `deploy-ueransim` to deploy UEs and gNodeB at scale
