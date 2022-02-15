---
title: Teradici CAS Cliff Notes
description: A guide deployed to quickly set-up CAS on a workstation without having to consult official documentation
author: chad-m-smith
tags: CAS, CAS-Plus, Windows
date_published: 2022-02-15
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

While Teradici's [official documentation](https://docs.teradici.com/find/product/cloud-access-software) is the recommended for deploying CAS, this “cliff noted” guide is a condensed version, highlighting recommendations, troubleshooting and observations 'in-line' as a part of this deployment process. This removes the need to open several  guides and ‘cuts the fat’ and illustrates the most common deployment scenarios seen out in the field over the past two years at Teradici.

**A brief description of deployment:**
1. Review of system requirement to ensure compatibility
1. Administrator of workstation logs into Teradici portal to download CAS agent.
1. Install CAS agent on workstation
1. Configure CAS agent
1. End-user logs into Teradici portal to download CAS client on their remote system
.
# Decide on the version of CAS agent that is needed #
**An assumption on type of CAS agent and purchase has been made in pre-sales processes**

There are two CAS agent SKUs availble depending on the requirements of the end-user. Each of these SKUs have a difference price point that is assoicated to the purchase/ registration key at time of sale. It is important that right agent is selected, there have been situations where end-users paid for Graphics Agents but downloaded Standard Agent. Our licensing server would permit this senerio but it would NOT prevent the opposite (paying for Standard but installing Graphics) from happening.

**Overview of CAS Agents:**

|  CAS Version    |  Performance    |         Encoder        |       Hardware      |    Use Case    |
| --------------- | --------------- |------------------------|---------------------|----------------|
| CAS Graphics    |    60FPS -4K    | multi-core AVX2 -NVENC | CPU AVX2 /GPU-Nvidia|Content Creation|
| CAS Standard    |    30FPS -2K    |        CPU             |       x86           |  Task Worker   |

# Ensure the requirements are met for each OS #
**The next step is to decide on the OS for the CAS agent**

Both CAS Graphics and Standard isn't locked to a particular OS. CAS Graphics has the option to be installed on various flavors of Windows, Linux and MAC. CAS Standard can be installed on Windows and Linux.  Each possible deployment seneraio has a system requirement page that should be verified before an installation begins. 

Before continuing consult the requirement page for the applicable deployment:
- [Graphics Agent for Windows](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/windows/22.01/admin-guide/requirements/system-requirements/)
- [Graphics Agent for Linux](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/linux/22.01/admin-guide/requirements/system-requirements/)
- [Graphics Agent for MAC](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/macos/22.01/admin-guide/requirements/system-requirements/)
- [Standard Agent for Windows](https://www.teradici.com/web-help/pcoip_agent/standard_agent/windows/22.01/admin-guide/requirements/system-requirements/)
- [Standard Agent for Linux](https://www.teradici.com/web-help/pcoip_agent/standard_agent/linux/22.01/admin-guide/requirements/system-requirements/)

# CAS Agent installation on host #
**This deployment guide we will focus on CAS Graphics on Windows, which will be the most common use case for this group.**

1. From the [Teradici portal](https://docs.teradici.com/find/product/cloud-access-software), select the CAS version and type of host systems OS. ) Then select the **‘Log in to download’** button.

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Select%20Download.png)

1. You must have a registered Teradici account in order to download the software, You can “Create an account”, if you don’t have an account. This account isn’t tied to a purchase or sale. HP or personal email address are allowed.  Note: Teradici Sales and Support employees are tied our old **@teradici.com** email addresses (not) **@HP.com**
     
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/LogintoPortal.png)

 1. &#x1F536;**(Optional)** &#x1F536; the Teradici portal will default to the latest GA build. By selecting the drop-down menu, you have access to the previous quarters build, as well has the next to upcoming beta and development builds as well. 

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Access2OptionalBuilds.png?raw=true)
      
 1. Next Scroll through the EULA until the “Agree button ”  isn't greyed button is available to “click on”

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/EULA_Agree.png?raw=true)     
 
 1. Once EULA is agreed you have two options a download (or) a download using a script via curl. (Download via script does include as user session token to programmatically authenticate to download page and download. Note: curl is primarly used for Linux, windows server does have curl pre-installed, windows 10)
 
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/DownLoadOptions.png?raw=true)
     
 1. Once the agent installer is downloaded, run installer under a local /AD administrator account. Administrator permission will be required for USB passthrough driver, audio driver and a PowerShell script to modify Windows Defender Firewall. Many times the installer launches and near the end receives a **PowerShell Execution Policy** error, the user at a minimum must set **AllSigned** when running outside of administrator account. 

      ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/RUNASADMIN.png?raw=true)
 
1. Select a language, English by default

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Selectlanguage.png?raw=true)
      
1. &#x1F536;**(Caution)** &#x1F536; You may receive an “unsupported graphics adapter cannot be found”, many times the GPU is supported, although the driver is not.
     - **If you have a supported GPU, but not the correct driver** - You have the option to select "No" to continue the CAS installation and fix afterwards.
     - **If you don't have a supported GPU, thus wrong driver as well** - You can still select "No", but will get a "installation aborted", in essence as selecting "Yes"

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/Unsupported%20Graphics.png?raw=true)
 
 1. If you have a supported GPU and driver, you will go straight to the installation wizard screen.
     
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/WelcomeScreen.png)
     
 1. There is a CAS Agent EULA section, that can just be "clicked" **I Agree** to continue.

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/CAS_EULA.png)
     
 1. Choose an Installation path, Next to continue.
 
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/InstallPath.png)
     
 1. The installation process should take under a minute to complete. Rarely, if there any installation errors will show up as &#x1F536; (RED) &#x1F536; text by the installer. You can scroll back to error messages can copy directly from installer. 
 
     Also installation logs are kept in *%Appdata%/roaming/Teradici/*

     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/InstallerProgress.png?raw=true)
 
 1. When purchasing Teradici CAS, you will receive a registration code that will have to be entered in the registration code window. All registration code follow the same format of a having a *ABCDEFGH12@AB12-C345-D67E-89FG* with @ in the middle of the code. (A registation code is tied to a pool of floating seats)
  
     You can select the “not now” and run a registration script after installation. Generally, this is for building “golden images”. Typically, images and will be deployed and have a start-up script for registration. For more information see [PowerShell Licensing Script](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/windows/22.01/admin-guide/licensing/licensing/)
