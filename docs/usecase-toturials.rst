.. |date| date::

.. _Current GOLD images: nrec-gold-images.html#current-gold-images
.. _How do I set the root password for my Linux instance?: faq.html#how-do-i-set-the-root-password-for-my-linux-instance

Use case toturials
==================

Last changed: |date|

.. contents::
.. :local:

.. note::

    Please be advised that these are merely examples of use cases, and they may be outdated.

Changing network interface for a running instance
-------------------------------------------------

It is possible to change the network interface (i.e., from dualStack to IPv6)
without rebooting or rebuilding your VM instance. This is possible between all
available networks in the dashboard. Changing network interface will change the IP
addresses of the instance. This toturial demonstrates how to change the network
interface from dualStack to IPv6.

.. TIP::

   **Set root password!**

   It is a good idea to set the root password prior to doing any network changes.
   The instance can then be accessed using the "Console" view in the dashboard.

In the Dashboard:

1. In the drop-down menu of your running instance, select "Detach Interface" (Figure 1).

   .. figure:: images/uc-if-1.png
      :align: center
      :figwidth: image

      Figure 1: Drop-down menu of the running instance (in Compute -> Instances). The first three options are shown. We will use all three options in this toturial.

2. Select the network to detach under "Port". In Figure 2, a dualStack network configuration that is currently used by the running VM instance, is selected for detachment.

   .. figure:: images/uc-if-2.png
      :align: center
      :figwidth: image

      Figure 2: Selecting existing network to detach.

3. In the drop-down menu of your running instance, select "Attach Interface" (Figure 1).

4. Select the new (suggested) network to attach. In Figure 3, a new IPv6 network is selected.

   .. figure:: images/uc-if-3.png
      :align: center
      :figwidth: image

      Figure 3: Selecting new network to attach.

.. TIP::

   **Automatic removal of security groups**

   Note that all security groups that used the network that you detached were removed
   from the instance as a result of detaching the interface. Because of this, you need to
   re-add the affected security group(s). In the drop-down menu of the running instance (Figure 1), select "Edit Instance". In this toturial, a security group for SSH access
   is re-added as shown in Figure 4.

   .. figure:: images/uc-if-4.png
      :align: center
      :figwidth: image

      Figure 4: Adding security group for SSH access.

Linux VM user management
------------------------

Your Linux VM will come with a root user in addition to a cloud user as described in `Current GOLD images`_.

User management follows standard Linux procedures. Below are some useful commands:

.. list-table:: Table 1: Linux commands for basic user management.
   :widths: 50 25 25
   :header-rows: 1

   * - Command
     - Description
     - Use case
   * - ``openssl rand -base64 6``
     - Generate random 6-digit password
     - Interactive
   * - ``sudo adduser <username>``
     - Create a new user with a password
     - Interactive
   * - ``sudo adduser --disabled-password <username>``
     - Create a new user with disabled password
     - When creating users with key-based login only. The user cannot be authenticated using password
   * - ``sudo useradd -m -s /bin/bash <username>``
     - Create a new user without specifying its password
     - -"-, non-interactive
   * - ``sudo passwd <username>``
     - Set password for user
     - Interactive
   * - ``sudo gpasswd -a <username> <groupname>``
     - Add user <username> to the group <groupname>
     - Ex. adding user admin to group sudo
   * - ``sudo passwd -l <username>``
     - Disable (lock) the password for <username>
     - The user can no longer be authenticated using password. Login to the user with other authentication methods will still work
   * - ``google-authenticator``
     - Run 2FA setup for scanning QR code with OTP mobile app
     - When having set up SSH to use the Google Authenticator PAM

Example:

Creating the additional user student with disabled password. From root:

.. code-block:: console

   adduser --disabled-password student

This should also create the home directory ``/home/student``.

SSH keys

The public SSH key you selected in the in the wizard when creating the VM instance, was installed for the cloud and root user. However, by default, public key-based SSH login is only enabled for the cloud user. If you like to enable this for root (not recommended), you need to edit the correspondig settings in ``/etc/ssh/sshd_config``, followed by a restart of the ssh service.

Example:

The manual process of installing the public SSH key for student is the following (from root):

.. code-block:: console

   mkdir -p /home/student/.ssh
   # Substitute KEY with the public SSH key received from user student
   echo KEY >> /home/student/.ssh/authorized_keys

2FA/MFA

You may want to setup the VM to use a pluggable authentication module (PAM) with your public SSH key `and` mobile one-time-password (OTP) app. Google Authenticator provides such a setup. The installation may vary with Linux distribution. For Debian-based systems, the package to install is ``libpam-google-authenticator`` and configuration is performed in ``/etc/pam.d/sshd`` and ``/etc/ssh/sshd_config``.

Example: Assuming that Google Authenticator PAM is setup correctly with the SSH server in the VM. From root:

.. code-block:: console

   su - student
   google-authenticator

A good default is to say yes ('y') to all options. A QR code should be printed. The student needs to somehow scan this QR code using any mobile OTP app. Additionally, the file ``/home/student/.google_authenticator`` will be created together with the generated QR code. This file can be deleted if you wish to re-run the ``google-authenticator`` command to get a new QR code.

Sudo

Passwordless sudo right is granted to the cloud user. This means that you may want to use sudo to set the root password while logged in with the cloud user, as described in `How do I set the root password for my Linux instance?`_. The config file enabling passwordless sudo for the cloud user should be located in ``/etc/sudoers.d/``. If you want passwordless sudo right for additional users, you can edit this file accordingly.

Example: To grant sudo right to user student, add user student to the group sudo. Then, find and edit the file where the cloud user is granted sudo right. For Ubuntu, the file is ``/etc/sudoers.d/90-cloud-init-users``. From root:

.. code-block:: console

   gpasswd -a student sudo
   echo -e '# User rules for ubuntu\nstudent ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers.d/90-cloud-init-users

If you followed the examples in this toturial, note that student can change to any user in the VM (using sudo su - <username>).

To prevent student from accessing other users, student and any other users in the VM should not have sudo rights, as well as a disabled password.

Any user logged into the VM may change to another user with password enabled (using su - <username>). This is a reason to create users with the --disabled-password option.

Shared account:

A shared user group1 may be created with password, and the password can be shared within the group. All members of the group should then be able to login to the VM using user group1 and shared password simultaneously. Shared accounts may also be accomplished by sharing the full (private+public) SSH key and possibly OTP app. However, this use case would go against introducing these increased security measures in the first place.

Lightweight Linux DE - LXDE + XRDP
-----------------------------------

This is a tutorial on how you may setup a minimal graphical desktop environment (DE) in your linux VM, and access it remotely using the Remote Desktop Protocol (RDP) over a Secure Shell (SSH) tunnel.

- LXDE: Lightweight X Desktop Environment
- xrdp, a VDI server using the Remote Desktop Protocol (RDP) protocol, and that starts isolated X sessions
- Web browser (firefox)
- File browser (pcmanfm)
- File de-compress/compress tool (xarchiver)
- Text processor (mousepad)
- Terminal emulator (lxterminal)
- Decent theme (shimmer themes).

Abbreviations:

RDP: Remote Desktop Protocol, SSH: Secure Shell, GUI: Graphical User Interface, VDI: Virtual Desktop Infrastructure, DE: Desktop Environment

.. Note::

   The specific steps required for GUI to your linux VM instance depend heavily on the software and distribution. The steps in this toturial are likely to change in the future. The last edit was 2024-09-04.

1. Launch a new linux VM instance

  - Image: GOLD Ubuntu 24.04 LTS
  - Flavor: m1.medium (4 GB RAM, 20 GB OS disk)
  - Network: IPv6
  - Add a security group that allows SSH to the instance for IPv6
  - Add your SSH key

  In this, toturial the instance is named ``vdi``

2. SSH login with TCP tunnel for RDP connection

   .. code-block:: console

      ssh ubuntu@<IPv6 address> -L 45000:localhost:3389

   where we choose a high numbered port that we want to use to access our DE on ``localhost`` on our local machine.

   If you are on a IPv4 only network such as eduroam, you can connect through ``login.uio.no`` or ``login.uib.no``, e.g., for UiO users

   .. code-block:: console

      ssh -J <username>@login.uio.no ubuntu@<IPv6 address> -L 45000:localhost:3389

   where <username> is your UiO username. This requires that your SSH key is installed on the login host.

3. Set password for the cloud user (will be asked with VDI login)

   .. code-block:: console

      sudo passwd ubuntu

4. Install software

   .. code-block:: console

      sudo apt update -y && sudo apt install -y xrdp openbox-lxde-session lxappearance lxterminal xarchiver mousepad shimmer-themes firefox

5. First VDI login

   Use a RDP Client to connect to ``localhost:45000``. The client to use on Windows is the built-in Windows Remote Desktop. A good Linux client is Remmina.

   You will be asked to login as user ubuntu with the password you set previously.

5. Necessary fixes

   - Fix lxpanel bug for Ubuntu 24.04 LTS [#f1]_ [#f2]_

     ``Right click on (the visible part of the) panel -> Panel Settings -> Panel Applets, select Desktop Pager, then click Remove``

   - Set decent theme

     ``Preferences -> Customize Look and Feel, select Greybird-dark -> Apply -> Close``

     ``Preferences -> Openbox Configuration Manager, select Numix -> Close``

     ``Right click on panel -> Panel Settings -> Appearance, under Background, select System theme -> Close``

   - Disable screensaver to avoid unwanted CPU consumption

     ``Preferences -> XScreenSaver Settings -> Mode: Disable Screen Saver -> Close``

   - (Windows only) Fix Windows Remote Desktop specific issues [#f3]_

     Enable shared clipboard as well as drive redirection in Windows Remote Desktop client (to ``thinclient_drives`` mount): Make sure Windows Remote Desktop client is configured properly by unchecking Printers and Smart cards. Select the drive(s) to redirect, as well as Clipboard, then save the profile.

6. Finish

   This toturial used the Remmina RDP client with custom screen resolution set to 1920x1080 (Figure 5).

   .. figure:: images/uc-vdi-1.png
      :align: center
      :figwidth: image

      Figure 5: Screenshot of the virtual DE with the GUI tools installed in this toturial.

.. rubric:: Footnotes

.. [#f1] https://askubuntu.com/questions/1518705/lxde-panel-gets-cut-off-on-ubuntu-24-04

.. [#f2] https://sourceforge.net/p/lxde/bugs/968/

.. [#f3] https://github.com/neutrinolabs/xrdp/issues/308

VirtualGL Linux DE - GNOME + TurboVNC (Terraform)
-------------------------------------------------

This tutorial demonstrates how to deploy a ready-to-use Ubuntu 24.04 LTS VM with a GNOME desktop and TurboVNC remote access on NREC OpenStack, using the one-click deployment scripts from the `nrec-oneclick-vps <https://github.com/norcams/nrec-oneclick-vps/>`_ repository.

.. TIP::

   **Prerequisites**

   - Terraform >= 1.5
   - NREC OpenStack credentials (``OS_USERNAME``, ``OS_PASSWORD``, ``OS_PROJECT_NAME``, ``OS_REGION_NAME``)
   - SSH client
   - TurboVNC viewer
   - Git (to clone the repository)

1. Clone the repository

   .. code-block:: console

      git clone https://github.com/norcams/nrec-oneclick-vps.git
      cd nrec-oneclick-vps

2. Create and fill in the environment file

   .. code-block:: console

      cp env.sh.template env.sh

   Edit ``env.sh`` and set your OpenStack API credentials:

   - ``OS_USERNAME``: your username (e.g. ``user@institution.no``)
   - ``OS_PASSWORD``: your password
   - ``OS_PROJECT_NAME``: your project name
   - ``OS_REGION_NAME``: your region (e.g. ``bgo``)

   The ``OS_AUTH_URL`` is pre-set to ``https://identity.api.bgo.nrec.no:5000/v3``.

.. TIP::

   **Windows**

   Windows users: copy ``env.ps1.template`` to ``env.ps1`` and set the same OpenStack credentials there. Run ``deploy.ps1`` instead of ``deploy.sh``.

3. Deploy the VM

   .. code-block:: console

      ./deploy.sh

   The script will:

   - Auto-detect your public IPv4/IPv6 address
   - Generate a ``terraform.tfvars`` with default flavor (``c1.xlarge``) and image (``GOLD Ubuntu 24.04 LTS``). These can be changed directly in ``deploy.sh``.
   - Generate a TLS private key and save it to ``keys/vps-<deployment-id>.pem``
   - Create an OpenStack keypair
   - Create a security group with SSH-only ingress
   - Launch a VM with cloud-init (installs TurboVNC, GNOME desktop, Google Chrome)
   - Print the VM IP addresses and SSH command

   Credentials are saved to:

   - VNC password: ``keys/vps-<deployment-id>.vncpass``
   - On VM: ``cat /home/ubuntu/.vnc-passwd``

4. SSH login with VNC tunnel

   .. code-block:: console

      ssh -L 55901:localhost:5901 -i keys/vps-<deployment-id>.pem ubuntu@<VM_IP>

   If you are connecting from IPv6-only:

   .. code-block:: console

      ssh -L 55901:localhost:5901 -i keys/vps-<deployment-id>.pem ubuntu@<VM_IPv6>

5. Start a VNC session and connect

   .. code-block:: console

      vncserver :1

   Then connect with TurboVNC to ``localhost:55901``, using the password from ``/home/ubuntu/.vnc-passwd``.

   .. TIP::

      **Desktop session**

      The default session starts with GNOME Flashback (Metacity). For the full modern GNOME session:

      .. code-block:: console

         vncserver :1 -wm gnome

6. Tear down the VM

   When finished, destroy all provisioned resources (including the VM, security groups, keypair, and local key files):

   .. code-block:: console

      terraform destroy

Fast Qwen3.6 inference on L40s flavor for agentic tasks
-------------------------------------------------------

This tutorial demonstrates how to run the `Qwen3.6-35B-A3B <https://unsloth.ai/docs/models/qwen3.6#mtp-qwen3.6-35b-a3b>`_ LLM with decent inference speed on an NREC L40s instance using llama.cpp and multi-token prediction (MTP).

.. TIP::

   **Instance requirements**

   - Flavor: ``gr1.L40S.24g.4xlarge`` (24 GB NVIDIA L40S vGPU)
   - Image: vGPU Ubuntu 24.04 LTS
   - Model: `unsloth/Qwen3.6-35B-A3B-MTP-GGUF <https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF>`_ with UD-Q2_K_XL dynamic 2-bit quantization

   The UD-Q2_K_XL quantization is a dynamic 2-bit format from Unsloth that reduces memory usage and increases inference speed. The A3B suffix indicates a Mixture of Experts (MoE) variant; no equivalent MoE variant exists yet for Qwen3.8.

1. Create and prepare the instance

   Create a new instance with the flavor and image above. After login, install required packages:

   .. code-block:: console

      sudo apt update
      sudo apt install -y nvidia-cuda-toolkit git python3-venv python3-pip pciutils build-essential cmake curl libcurl4-openssl-dev nvtop nload

      sudo timedatectl set-timezone Europe/Oslo

   Follow the "Upgrading the instance drivers" section from the `NREC vGPU documentation <https://docs.nrec.no/vgpu.html#upgrading-the-instance-drivers>`_ to install the latest NVIDIA drivers.

2. Verify GPU

   .. code-block:: console

      nvidia-smi

   You should see the NVIDIA L40S GPU listed.

3. Build llama.cpp

   .. code-block:: console

      git clone https://github.com/ggml-org/llama.cpp
      cd llama.cpp
      cmake -B build -DBUILD_SHARED_LIBS=OFF -DGGML_CUDA=ON
      cmake --build build --config Release -j --clean-first --target llama-cli llama-mtmd-cli llama-server llama-gguf-split
      cp build/bin/llama-* .

4. Create a Python environment and download the model

   .. code-block:: console

      cd /home/ubuntu/llama.cpp
      python3 -m venv hf-llama
      source hf-llama/bin/activate
      pip install -U "huggingface_hub"

   .. code-block:: console

      hf download unsloth/Qwen3.6-35B-A3B-MTP-GGUF \
          --local-dir unsloth/Qwen3.6-35B-A3B-MTP-GGUF \
          --include "*mmproj-F16*" \
          --include "*UD-Q2_K_XL*"

5. Start the inference server

   .. code-block:: console

      ./llama-server \
          --model unsloth/Qwen3.6-35B-A3B-MTP-GGUF/Qwen3.6-35B-A3B-UD-Q2_K_XL.gguf \
          --mmproj unsloth/Qwen3.6-35B-A3B-MTP-GGUF/mmproj-F16.gguf \
          --temp 0.6 --top-p 0.95 --min-p 0.00 --top-k 20 \
          --ctx-size 262144 --port 8001 \
          --spec-type draft-mtp --spec-draft-n-max 2 \
          --chat-template-kwargs '{"preserve_thinking":true}' \
          --no-mmap --image-min-tokens 1024

   Key options explained:

   - ``--spec-type draft-mtp --spec-draft-n-max 2``: enables multi-token prediction, a speculative decoding technique that speeds up inference significantly
   - ``--mmproj``: enables image recognition capability (the multimodal projector); agentic frameworks with built-in image tools such as Hermes desktop should auto-detect and use it
   - ``--chat-template-kwargs '{"preserve_thinking":true}'``: adds extra reasoning tokens that improve the model's reasoning quality
   - ``--no-mmap``: avoids memory mapping for better GPU performance
   - ``--image-min-tokens 1024``: minimum tokens allocated for image processing

   The server exposes an OpenAI-compatible API at ``http://127.0.0.1:8001/v1``.

   To use the CLI instead of the server, run ``llama-cli`` with the same arguments (omit ``--port``).

   .. NOTE::

      The ``--spec-type draft-mtp --spec-draft-n-max 2`` flags cause a CUDA kernel
      crash with very short inputs (1-2 characters) in ``llama-cli``. These flags are
      safe to use with ``llama-server`` (which handles longer context), but should be
      omitted when running ``llama-cli`` interactively.

6. Connect an agentic framework

   Configure your agentic framework (e.g., agentic frameworks with built-in image tools such as Hermes desktop) to use the local endpoint:

   .. code-block:: console

      # In your agent config:
      # provider: custom
      # endpoint: http://127.0.0.1:8001/v1

   Stop the server with ``Ctrl+C``.

Fast Qwen3.6 inference on L40s flavor for agentic tasks (Ubuntu 26.04 LTS)
--------------------------------------------------------------------------

This is an adaptation of the `Fast Qwen3.6 inference on L40s flavor for agentic tasks`_ tutorial for Ubuntu 26.04 LTS (Resolute Raccoon).

.. TIP::

   **Instance requirements**

   - Flavor: ``gr1.L40S.24g.4xlarge`` (24 GB NVIDIA L40S vGPU)
   - Image: ``vGPU Ubuntu 26.04 LTS``
   - Model: `unsloth/Qwen3.6-35B-A3B-MTP-GGUF <https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF>`_ with UD-Q2_K_XL dynamic 2-bit quantization

   The UD-Q2_K_XL quantization is a dynamic 2-bit format from Unsloth that reduces memory usage and increases inference speed. The A3B suffix indicates a Mixture of Experts (MoE) variant; no equivalent MoE variant exists yet for Qwen3.8.

1. Create and prepare the instance

   Create a new instance with the flavor and image above. After login, install
   required packages:

   .. code-block:: console

      sudo apt update
      sudo apt install -y cuda-toolkit-13 gcc-14 g++-14 git cmake nvtop btop tree nvitop nload python3.14-venv

      sudo timedatectl set-timezone Europe/Oslo

   .. NOTE::

      **Two pre-built fixes are required on Ubuntu 26.04 LTS.**

      **NVML version mismatch:** The vGPU image ships with kernel module ``580.159.03``,
      but ``cuda-toolkit-13`` installs userspace ``580.173.02``. This breaks ``nvidia-smi``.
      Fix by re-linking the NVML symlink:

      .. code-block:: console

         sudo ln -sf libnvidia-ml.so.580.159.03 /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1

      Substitute the actual kernel module version with
      ``ls /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.580.* | head -1`` to find the correct version.

      **CUDA/GCC incompatibility:** CUDA 13.1 does not support GCC 15 (the default on
      Ubuntu 26.04 LTS). nvcc reads GCC 15's ``bits/mathcalls.h`` which conflicts with
      CUDA 13.1's ``math_functions.h`` (``noexcept(true)`` vs no ``noexcept``).
      Patch before building:

      .. code-block:: console

         MATH_F=/usr/local/cuda/targets/x86_64-linux/include/crt/math_functions.h
         sudo sed -i 's/extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ double                 rsqrt(double x);/extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ double                 rsqrt(double x) noexcept(true);/' $MATH_F
         sudo sed -i 's/extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ float                  rsqrtf(float x);/extern __DEVICE_FUNCTIONS_DECL__ __device_builtin__ float                  rsqrtf(float x) noexcept(true);/' $MATH_F

      These cmake flags alone cannot fix these issues — both fixes are mandatory.

2. Verify GPU

   .. code-block:: console

      nvidia-smi

   You should see the NVIDIA L40S GPU listed.

3. Build llama.cpp

   .. code-block:: console

      git clone https://github.com/ggml-org/llama.cpp
      cd llama.cpp
      export PATH=/usr/local/cuda/bin:\$PATH
      cmake -B build \
        -DBUILD_SHARED_LIBS=OFF \
        -DGGML_CUDA=ON \
        -DCMAKE_CUDA_HOST_COMPILER=g++-14 \
        -DCMAKE_CXX_COMPILER=g++-14 \
        -DCMAKE_CUDA_COMPILER=/usr/local/cuda/bin/nvcc \
        -DCMAKE_CUDA_ARCHITECTURES=86
      cmake --build build --config Release -j
      cp build/bin/llama-* .

   Key build flags:

   - ``-DCMAKE_CUDA_HOST_COMPILER=g++-14``: tells nvcc to use GCC 14 (required because CUDA 13.1 does not support GCC 15)
   - ``-DCMAKE_CUDA_ARCHITECTURES=86``: targets sm_86 (L40S Ampere compute capability)

4. Create a Python environment and download the model

   .. code-block:: console

      python3 -m venv hf-llama
      source hf-llama/bin/activate
      pip install -U "huggingface_hub"

   .. code-block:: console

      hf download unsloth/Qwen3.6-35B-A3B-MTP-GGUF \
          --local-dir unsloth/Qwen3.6-35B-A3B-MTP-GGUF \
          --include "*mmproj-F16*" \
          --include "*UD-Q2_K_XL*"

5. Start the inference server

   .. code-block:: console

      ./llama-server \
          --model unsloth/Qwen3.6-35B-A3B-MTP-GGUF/Qwen3.6-35B-A3B-UD-Q2_K_XL.gguf \
          --mmproj unsloth/Qwen3.6-35B-A3B-MTP-GGUF/mmproj-F16.gguf \
          --temp 0.6 --top-p 0.95 --min-p 0.00 --top-k 20 \
          --ctx-size 262144 --port 8001 \
          --spec-type draft-mtp --spec-draft-n-max 2 \
          --chat-template-kwargs '{"preserve_thinking":true}' \
          --load-mode mmap --image-min-tokens 1024

   Key options explained:

   - ``--spec-type draft-mtp --spec-draft-n-max 2``: enables multi-token prediction, a speculative decoding technique that speeds up inference significantly
   - ``--mmproj``: enables image recognition capability (the multimodal projector); agentic frameworks with built-in image tools such as Hermes desktop should auto-detect and use it
   - ``--chat-template-kwargs '{"preserve_thinking":true}'``: adds extra reasoning tokens that improve the model's reasoning quality
   - ``--load-mode mmap``: memory-mapped model loading (replaces the deprecated ``--no-mmap``)
   - ``--image-min-tokens 1024``: minimum tokens allocated for image processing

   The server exposes an OpenAI-compatible API at ``http://127.0.0.1:8001/v1``.

   To use the CLI instead of the server, run ``llama-cli`` with the same arguments (omit ``--port``).

   .. NOTE::

      The ``--spec-type draft-mtp --spec-draft-n-max 2`` flags cause a CUDA kernel
      crash with very short inputs (1-2 characters) in ``llama-cli``. These flags are
      safe to use with ``llama-server`` (which handles longer context), but should be
      omitted when running ``llama-cli`` interactively.

   Benchmarks measured on the L40S:

   - Model load time: ~4.6s
   - Prompt eval: ~10.2ms/token (98 t/s)
   - MTP generation: ~5.0ms/token (200 t/s), 100% draft acceptance, 12 drafts accepted

6. Connect an agentic framework

   Configure your agentic framework (e.g., agentic frameworks with built-in image tools such as Hermes desktop) to use the local endpoint:

   .. code-block:: console

      # In your agent config:
      # provider: custom
      # endpoint: http://127.0.0.1:8001/v1

   Stop the server with ``Ctrl+C``.

Hermes Agent with XRDP on NREC
-------------------------------

This tutorial demonstrates how to deploy a full virtual desktop infrastructure (VDI) workflow on NREC OpenStack: provision a GNOME VM with XRDP remote access, then install and run Hermes Agent inside the virtual desktop. The workflow uses the UiO-managed VDI as a management hub to deploy and manage OpenStack resources.

.. TIP::

   **Prerequisites**

   - A Feide-connected university login (e.g. ``@uio.no``, ``@uib.no``)
   - Access to the Omnissa Horizon client
   - RDP client installed on your local machine (Windows Remote Desktop, Remmina on Linux, Microsoft Remote Desktop on macOS)
   - SSH client
   - Git and Terraform

1. Sign up for NREC access

   Log in to `access.nrec.no <https://access.nrec.no>`_ with your Feide credentials. This will automatically create a demo project sufficient for running the nrec-oneclick-vps provisioning. If you need more resources, submit a request at `request.nrec.no <https://request.nrec.no>`_.

   On the same page, click **API password** to generate an NREC OpenStack API password. Save it securely — avoid storing it in multiple note-taking apps or third-party cloud services.

.. TIP::

   **API password scope**

   The API password is shared across all NREC regions. You only need to generate it once.

2. Verify dashboard access

   Log in to `dashboard.nrec.no <https://dashboard.nrec.no>`_ using your Feide credentials. You should see your project quota and resources.

3. Connect to the UiO Linux Desktop (VDI)

   Open the **Omnissa Horizon** client and connect to server ``view.uio.no``. Log in with your UiO Microsoft 2FA credentials, then select **UiO Linux Desktop**.

   Open **Activities → Terminal**.

4. Prepare the Python environment

   .. code-block:: console

      python3 -m venv nrec-venv
      source nrec-venv/bin/activate
      which python
      which pip

   This creates a local Python virtual environment in your shared home directory — the same directory you can access at ``login.uio.no``.

   .. code-block:: console

      pip install terraform-install python-openstackclient python-glanceclient

5. Clone the deployment repository

   .. code-block:: console

      git clone https://github.com/norcams/nrec-oneclick-vps.git
      cd nrec-oneclick-vps
      git checkout xrdp

6. Configure environment variables

   .. code-block:: console

      cp env.sh.template env.sh
      nano env.sh

   Fill in the following fields:

   - ``OS_USERNAME``: your username (e.g. ``username@uio.no``)
   - ``OS_PASSWORD``: the API password from step 1
   - ``OS_PROJECT_NAME``: your project name (e.g. ``DEMO-username.uio.no``)
   - ``OS_REGION_NAME``: your region (e.g. ``osl`` or ``bgo``)

   Save and exit with ``Ctrl+X``, then ``Y``, then ``Enter``.

.. TIP::

   **Flavor selection**

   - Using the **DEMO** project: use a small flavor such as ``c1.medium``
   - Using a **PRIVATE** project (granted via request.nrec.no): use the default ``c1.xlarge``

   To change the flavor, edit ``deploy.sh`` and update the ``flavor`` variable before running.

7. Deploy the VM

   .. code-block:: console

      ./deploy.sh

   While the script runs, you can monitor progress in the dashboard under **Compute → Instances → vps-* → Log and Console**.

   You can also get the the console log using the openstack cli client just installed:

   .. code-block:: console

      openstack console log show vps-* | less +G

   Replace the ``*`` with the randomly generated instance ID printed by the script (e.g. ``vps-d2a440``).

8. Connect via RDP

   SSH into the VM with a local port forwarded to the XRDP port. This can be done either directly from your local machine (private or UiO-managed, or from UiO-managed VDI). The autogenerated ssh security group needs to be modified to allow additional SSH logins from machines with public IP other than the specific VDI host used to deploying the VPS. Here, login is demonstrated from the same machine as where deploy were run:

   .. code-block:: console

      ssh -L 33389:localhost:3389 -i keys/vps-<deployment-id>.pem ubuntu@<VM_IP>

   If you do not want to work with the autogenerated (admin) cloud password at ~/.admin-password, set the password for the cloud user (will be requested by the RDP client):

   .. code-block:: console

      sudo passwd ubuntu

   Open your local RDP client and connect to ``localhost:33389``. On Windows, use the built-in **Windows Remote Desktop**. On Linux, use **Remmina**. On macOS, use **Windows App**. Log in as ``ubuntu`` with the cloud user password.

.. TIP::

   **Note**

   - If you have a non-US keyboard, add your layout via **Settings → Keyboard → Add Input Source** (e.g. Norwegian or Norwegian (Macintosh)).
   - On high-resolution displays, increase scaling: **System Settings → Displays → Scale: 200% → Apply**.
   - You can set dark mode in the Settings app at Appearance -> Style -> Dark

9. Install Hermes Agent

   Right-click on the desktop and select **Open Terminal**. It is conventient to also open **System Monitor** to check available resources:

   .. code-block:: console

      gnome-system-monitor &

   Launch Google Chrome:

   .. code-block:: console

      google-chrome &

   You do not need to sign into Chrome. Re-open Chrome to get greeted with an option to select a default search engine (e.g. Brave).

   Search for **Hermes Agent** and visit the website. Copy the Linux install command, make sure system is updated and install:

   .. code-block:: console

      sudo apt update
      curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

   If prompted, run:

   .. code-block:: console

      source ~/.bashrc

   Then install Hermes desktop:

   .. code-block:: console

      hermes desktop

10. Manage OpenStack from the UiO VDI

    Open a second terminal on the UiO VDI (not inside the NREC VM) to manage your OpenStack resources:

    .. code-block:: console

       source nrec-venv/bin/activate
       source nrec-oneclick-vps/env.sh

    List servers:

    .. code-block:: console

       openstack server list

    The regular backup method for VMs in NREC (shutoff or running) is taking a snapshot:

    .. code-block:: console

       openstack server image create --name vps-snapshot-1 --wait vps-*

    Note that this will not backup the volume attached to the VM. There are separate openstack commands to take backup of volumes that are not yet fully implemented in NREC. The working method as of today is to create an image snapshot of the volume, then download the snapshot with glance. The volume can then be mounted to a linux machine using the nbd (network block device) driver.

    Download the snapshot to the UiO VDI:

    .. code-block:: console

       glance image-download --file vps-snapshot-1.qcow2 --progress $(openstack image show -f value -c id vps-*)

    Transfer the file to your local machine through a login jump host:

    .. code-block:: console

       rsync --progress username@login.uio.no vps-snapshot.qcow2 .

    The snapshot can be run locally (e.g. in VirtualBox) by converting from qcow2 to vmdk:

    .. code-block:: console

       qemu-img convert -p -f qcow2 -O vmdk vps-snapshot-1.qcow2 vps-snapshot-1.vmdk

    A VirtualBox VM created from this disk can use these specs: 16 GB RAM, 8 vCPU, EFI enabled, using the existing disk.

11. Clean up

    Destroy all NREC resources:

    .. code-block:: console

       terraform destroy

.. rubric:: Tips

**Direct RDP access from your local machine**

For better desktop performance, you can connect directly to the NREC VM from your local machine:

1. Allow SSH access from your public IP. In the dashboard, go to **Network → Security Groups → vps-***, click **Manage Rules**, then **Add Rule**: SSH, CIDR ``<your public IPv4>/32`` or ``<your public IPv6>/128``.

2. Add your SSH public key (from ``~/.ssh``) to ``~/.ssh/authorized_keys`` in the NREC VM.

3. Connect with an SSH + RDP tunnel (note that here the generated vps private key (-i ...) is not used):

   .. code-block:: console

      ssh -L 33389:localhost:3389 ubuntu@<VM_IPv4_or_IPv6>

   Then connect your local RDP client to ``localhost:33389``.

**Clipboard integration**

If you use the full Omnissa Horizon client with clipboard enabled, you can copy/paste between your local system and the VDI clipboard in UiO Linux VDI terminal directly:

.. code-block:: console

   # Copy from VM to local clipboard
   echo "text" | xclip -in -selection clipboard

   # Paste from local clipboard into VM
   xclip -out -selection clipboard

If text is not copying correctly, make sure you click into the target window before performing copy/paste actions. This is an additional way for copy paste than the regular ctrl shortcuts and mouse clicks. In Linux terminals, use ``Ctrl+Shift+C`` (copy) and ``Ctrl+Shift+V`` (paste).

**Persistent tmux sessions**

Create persistent tmux sessions in the UiO VDI or NREC VM:

.. code-block:: console

   tmux new-session -t persistent-vdi-session-1
   ctrl + b, d (will detach current session)
   tmux attach-session -t persistent-vdi-session-1

A way to overcome the 1-week reboot cycle in UiO VDI, is to save tmux sessions with the Tmux Resurrect plugin.

