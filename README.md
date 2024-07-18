# NPSS Bootcamp


## Accompanying Slidware: 

The lab material contained here is part of NPSS bootcamp. Slideware for the bootcamp is posted [here](https://docs.google.com/presentation/d/1byvAY0zb2PeCS7mtFymNdtL6b1vvPI72FRLVt4xdBP8/edit?usp=drive_link)

## Labs for Hub Cluster Installation: 

These lab uses m3.large.x86 metal server from Equinix. This can be requested through [RHDP](https://demo.redhat.com/catalog?search=equinix+metal+baremetal+blank)

[!NOTE] The lab environment takes around 8-10 minutes to be ready and available.

| Lab ID | Description | URL |
|--------|-------------|------|
ABI1 | Disconnected Cluster Installation |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/abi.md |
ABI2 | Installing Operators in a disconnected Hub cluster |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/hub_operators.md |

## Labs for ZTP Concepts:

These lab requires either a lightweight RHEL server (could be a VM) or a SNO (for the 2nd lab). Recommended to use [this]https://demo.redhat.com/catalog/babylon-catalog-prod/order/openshift-cnv.ocpmulti-single-node-cnv.prod) environment with OCP 4.14. 


| Lab ID | Description | URL |
|--------|-------------|------|
1 | Kustomization Lab |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/kustomization.md | 
2 | Understanding OpenShift GitOps |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/gitops.md | 
3 | Lab Module to Translate SiteConfig |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/siteconfig.md | 
4 | Lab module to experience PGT Translation | https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/policies.md | 


## Labs for ZTP: 

The following labs use a [this](https://demo.redhat.com/catalog/babylon-catalog-prod/order/equinix-metal.ocp4-ran.prod) RHDP Environment. 

[!NOTE] The lab may take around 2 hours to spin up and ready to be used

Once the environment is ready, use [these stesp](https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_prep.md) to prefer it for the labs. 

| Lab ID | Description | URL |
|--------|-------------|------|
|5 |  Verifying ZTP readiness |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_verify.md | 
|6 |  Prepare Git Structure |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_0_git_structure.md  | 
|7 |  Prepare SiteConfig |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_1_siteconfig_sno2.md |
|8 |  Prepare PGT | https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_2_pgt.md |
|9 | Run ZTP Deployment GitOps |  https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_3_running_app.md |
|10| Importing a pre-deployed Cluster | https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_4_adding_sno1.md |
|11| Applying Day2 policy | https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_4_day2_policies.md |
|12| Upgrading a ZTP Deployed Clusters | https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_5_upgrade.md |
|13| A Worker Node to a Cluster | https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_6_adding_worker.md |
|14| Deploy SNO3 with Extra Manifests | https://gitlab.consulting.redhat.com/shassan/bootcamp/-/blob/main/lab_build_X_deploy_sno3.md |
