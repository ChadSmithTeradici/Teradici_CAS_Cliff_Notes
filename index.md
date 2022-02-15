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
 
     ![image](https://github.com/ChadSmithTeradici/Teradici_CAS_Cliff_Notes/blob/main/images/EULA_Agree.png?raw=true)  
