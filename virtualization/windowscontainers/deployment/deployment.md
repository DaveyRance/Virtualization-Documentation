﻿---
title: Deploy Windows Containers on Windows Server
description: Deploy Windows Containers on Windows Server
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: ba4eb594-0cdb-4148-81ac-a83b4bc337bc
---

# Container Host Deployment - Windows Server

Deploying a Windows container host has different steps depending on the operating system and the host system type (physical or virtual). This document details deploying a Windows container host to either Windows Server 2016 or Windows Server Core 2016 on a physical or virtual system.

## Install Container Feature

The container feature needs to be enabled before working with Windows containers. To do so run the following command in an elevated PowerShell session.

```none
Install-WindowsFeature containers
```

When the feature installation has completed, reboot the computer.

```none
Restart-Computer -Force
```

## Install Docker

Docker is required in order to work with Windows containers. Docker consists of the Docker Engine, and the Docker client. For this exercise, both will be installed.

Download the Docker engine and client as a zip archive.

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

Expand the zip archive into Program Files, the archive contents is already in docker directory.

```none
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Run the following two commands to add the Docker directory to the system path.

```none
# For quick use, does not require shell to be restarted.
$env:path += ";c:\program files\docker"

# For persistent use, will apply even after a reboot. 
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

To install Docker as a Windows service, run the following.

```none
dockerd --register-service
```

Once installed, the service can be started.

```none
Start-Service Docker
```

## Install Base Container Images

Before working with Windows Containers, a base image needs to be installed. Base images are available with either Windows Server Core or Nano Server as the underlying operating system. For detailed information on Docker container images, see [Build your own images on docker.com](https://docs.docker.com/engine/tutorials/dockerimages/).

To install the Windows Server Core base image run the following:

```none
docker pull microsoft/windowsservercore
```

To install the Nano Server base image run the following:

```none
docker pull microsoft/nanoserver
```

> Please read the Windows Containers OS Image EULA which can be found here – [EULA](../Images_EULA.md).

## Hyper-V Container Host

In order to run Hyper-V containers, the Hyper-V role is required. If the Windows container host is itself a Hyper-V virtual machine, nested virtualization will need to be enabled before installing the Hyper-V role. For more information on nested virtualization, see [Nested Virtualization]( https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting).

### Nested Virtualization

The following script will configure nested virtualization for the container host. This script is run on the parent Hyper-V machine. Ensure that the container host virtual machine is turned off when running this script.

```none
#replace with the virtual machine name
$vm = "<virtual-machine>"

#configure virtual processor
Set-VMProcessor -VMName $vm -ExposeVirtualizationExtensions $true -Count 2

#disable dynamic memory
Set-VMMemory $vm -DynamicMemoryEnabled $false

#enable mac spoofing
Get-VMNetworkAdapter -VMName $vm | Set-VMNetworkAdapter -MacAddressSpoofing On
```

### Enable the Hyper-V role

To enable the Hyper-V feature using PowerShell, run the following command in an elevated PowerShell session.

```none
Install-WindowsFeature hyper-v
```
