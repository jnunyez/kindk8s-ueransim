# kindk8s-ueransim

config maps and deployments of 5G-NR emulated networks in k8s 

## Rationale

This repo leverages [ueransim](https://github.com/aligungr/UERANSIM) to deploy 5G gNB and a 5G UE inside k8s using [kind](https://kind.sigs.k8s.io).

## Quickstart

1. Generate open5gs and webui base image, as detailed [here](http://github.com/jnunyez/build-open5gs).

2. Build the UERANSIM image. The repo to build this image is available in [here](https://github.com/build-ueransim).

3. Deploy open5gs and prerequisites in k8s. The manifests and detailed instructions for deploying open5gs in k8s are available [here](https://github.com/jnunyez/kindk8s-open5gs).

4. Build the UERANSIM image. The repo to build this image is available in [here](https://github.com/build-ueransim).

5. Assure the 5G-Core has registered the subscriber profiles planned to register with the 5G-NR simulator. This can be done manually by opening the webui dashboard or automated via a script. The `webui` service is externally accessible using a NodePort to the default port in the app (i.e., port 3000). See screenshot of webui dashboard below:

![Subscriber](./images/webui.png?raw=true)



6. Deploy a 5G gNB that attaches to the AMF and subsequently a 5G UE already subscribed in the UDM: 

   ```
   deploy-ueransim
   ```

7. Check that the UE is subscribed and gets IP info by checking the logs of the UE pod e.g., `kubectl logs -f ue-7798cbd9b4-2ffl9`. The output of this command should confirm the succesful registration of the UE.

```
UERANSIM v3.2.2
[2021-07-23 16:24:04.136] [nas] [info] UE switches to state [MM-DEREGISTERED/PLMN-SEARCH]
[2021-07-23 16:24:04.137] [rrc] [debug] New signal detected for cell[1], total [1] cells in coverage
[2021-07-23 16:24:04.137] [nas] [info] Selected plmn[001/01]
[2021-07-23 16:24:04.137] [rrc] [info] Selected cell plmn[001/01] tac[12345] category[SUITABLE]
[2021-07-23 16:24:04.137] [nas] [info] UE switches to state [MM-DEREGISTERED/PS]
[2021-07-23 16:24:04.137] [nas] [info] UE switches to state [MM-DEREGISTERED/NORMAL-SERVICE]
[2021-07-23 16:24:04.137] [nas] [debug] Initial registration required due to [MM-DEREG-NORMAL-SERVICE]
[2021-07-23 16:24:04.137] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2021-07-23 16:24:04.137] [nas] [debug] Sending Initial Registration
[2021-07-23 16:24:04.137] [nas] [info] UE switches to state [MM-REGISTER-INITIATED]
[2021-07-23 16:24:04.137] [rrc] [debug] Sending RRC Setup Request
[2021-07-23 16:24:04.138] [rrc] [info] RRC connection established
[2021-07-23 16:24:04.138] [rrc] [info] UE switches to state [RRC-CONNECTED]
[2021-07-23 16:24:04.138] [nas] [info] UE switches to state [CM-CONNECTED]
[2021-07-23 16:24:04.142] [nas] [debug] Authentication Request received
[2021-07-23 16:24:04.143] [nas] [debug] Sending Authentication Failure due to SQN out of range
[2021-07-23 16:24:04.148] [nas] [debug] Authentication Request received
[2021-07-23 16:24:04.153] [nas] [debug] Security Mode Command received
[2021-07-23 16:24:04.153] [nas] [debug] Selected integrity[2] ciphering[0]
[2021-07-23 16:24:04.164] [nas] [debug] Registration accept received
[2021-07-23 16:24:04.164] [nas] [info] UE switches to state [MM-REGISTERED/NORMAL-SERVICE]
[2021-07-23 16:24:04.164] [nas] [debug] Sending Registration Complete
[2021-07-23 16:24:04.164] [nas] [info] Initial Registration is successful
[2021-07-23 16:24:04.164] [nas] [debug] Sending PDU Session Establishment Request
[2021-07-23 16:24:04.164] [nas] [debug] UAC access attempt is allowed for identity[0], category[MO_sig]
[2021-07-23 16:24:04.367] [nas] [debug] Configuration Update Command received
[2021-07-23 16:24:04.381] [nas] [debug] PDU Session Establishment Accept received
[2021-07-23 16:24:04.381] [nas] [info] PDU Session establishment is successful PSI[1]
[2021-07-23 16:24:04.392] [app] [info] Connection setup for PDU session[1] is successful, TUN interface[uesimtun0, 45.45.0.2] is up.
```

## ToDo

- Generalize `deploy-ueransim` to deploy UEs and gNodeB at scale
