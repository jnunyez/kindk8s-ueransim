# Running UERANSIM in OCP

Running UERANSIM 5G-NR emulator and connect to Open5Gs core in OCP 4.8.X 

## Rationale

This repo leverages [ueransim](https://github.com/aligungr/UERANSIM) to deploy an emulated 5G-SA gNB and a 5G-SA UE implementing 3GPP Release 15 and connect to Open5Gs core, all running on top of [OCP](https://github.com/openshift). We will have:

	* 5G-SA UE running in a OCP pod
	* 5G-SA gNB running in a OCP pod
	* 5GC running in OCP pods

Radio links connecting UE and gNB OCP pods are simulated over UDP protocol. More information regarding the implemented 3GPP features can be found [at UERANSIM Feature-Set](https://github.com/aligungr/UERANSIM/wiki/Feature-Set).

## Quickstart

1. Build the UERANSIM image. The repo to build this image is available in [here](https://github.com/jnunyez/build-ueransim). 
	- Image are also publicly available:

		```console
    	podman pull quay.io/jnunez/ueransim
		```

2. Build open5gs and webui base image, as detailed [here](http://github.com/jnunyez/build-open5gs). 
	
	- Images are also publicly available, download them from:

		```console
    	podman pull quay.io/jnunez/open5gs
    	podman pull quay.io/jnunez/webui
		```

3. Deploy open5gs in OCP. The manifests along with detailed instructions for deploying open5gs in OCP are available [here](https://github.com/jnunyez/kindk8s-open5gs). Assure that all open5gs pods are running properly.

4. Assure the 5G-Core has registered the subscriber profiles planned to be subsequently used with the 5G-NR simulator. 
	
	- SIM provisioning can be done manually by opening the webui dashboard. The `webui` service is externally accessible using a NodePort to the default port in the app (i.e., port 3000). See screenshot of webui dashboard below:

		![Subscriber](./images/webui.png?raw=true)

	- Automated SIM provisioning (**TBA**)

5. Deploy a 5G gNB that attaches to the AMF and subsequently a 5G UE that will attach to the gNB. 

	- The UE must be already subscribed in the UDM as detailed in step 5.
 
	- Create the sriov interfaces attachments for n2 and n3 ifaces

		```console
		kustomize assets/sriov | oc apply -f -
		```

	- Run the ueransim deployment:

   		```console
   		deploy-ueransim
   		```

6. Check that the UE is subscribed and gets IP info by checking the logs of the UE or gNB pod e.g., `kubectl logs -f ue-pod-identifier`. The output of this command should confirm the succesful registration of the UE.

	```console
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

7. Test 5G UE uplink and downlink connectivity through ue interface.

	```console
	ping -I uesimtun0 8.8.8.8
	PING 8.8.8.8 (8.8.8.8) from 45.45.0.2 uesimtun0: 56(84) bytes of data.
	64 bytes from 8.8.8.8: icmp_seq=1 ttl=116 time=13.6 ms
	64 bytes from 8.8.8.8: icmp_seq=2 ttl=116 time=13.3 ms
	```

	![Wireshark](./images/wireshark.png?raw=true)


## ToDo

- Generalize `deploy-ueransim` to deploy UEs and gNodeB at scale
