# COEN 241 HW 1
## System Vs OS Virtualization
#### Project Report
#### Name : Venkata Rahul Yalavarthi
---
---
### Detailed configurations

Detailed configurations of the experimental setup I have used for the homework are as follows:
- **Processor:** 12th Gen Intel® Core™ i5-12400F × 12
- **Memory:** 16.0 GiB
- **OS Name:** Fedora Linux 37 (Workstation Edition)
- **OS Type:** 64-bit
- **Disk Capacity:** 500.1 GB
- **Hardware Model:** Gigabyte Technology Co., Ltd. B660M DS3H AX DDR4
- **Graphics:** NVIDIA GeForce RTX™ 3050

---
## System Virtualization (QEMU) Setup
### QEMU Virtual Machine Deployment

#### Introduction
This report details the steps taken to enable a QEMU virtual machine (VM) on a Fedora Linux 37 system. The goal was to install an Ubuntu Server (provided as an ISO file) using QEMU commands and appropriate VM configurations. 

#### QEMU Virtual Machine Configuration

In this section, we will go through the steps to enable a QEMU virtual machine on a Fedora Linux 37 operating system. The virtual machine will run an Ubuntu Server, using the ISO file provided in the assignment.

The following are the commands and configurations used to install the QEMU virtual machine:

1. **Installing QEMU**:
To install QEMU, we run the following command in the terminal:sh
```sh
$ sudo dnf install qemu
```

2. **Creating a QEMU Image**:
Once QEMU is installed, we create a QEMU Image by running the following command in the terminal: (This command creates a virtual image with 10 GiB of disk space.)
```sh
$ sudo qemu-img create ubuntu.img 10G -f qcow2
```
3. **Installing Ubuntu Server VM**:
Once the Ubuntu image is created, we install the virtual machine by running the following command in the terminal: (This command boots the virtual machine and installs the Ubuntu Server.)
```sh
$ sudo qemu-system-x86_64 -hda ubuntu.img -boot d -cdrom ./ubuntu.img -m 2046 -boot strict=on
```

The following are the explanations of the configurations used in the third command:
- ```qemu-system-x86_64```: represents the normal QEMU command for an x86_64 machine.
- ```-hda [file]```: specifies the image file for the virtual IDE hard disk. In this case, it is ```ubuntu.img```.
- ```-boot [a | c | d | n]```: boot from floppy disk (a), hard disk (c), CD-ROM (d), or etherboot (n). In this case, we boot from the CD-ROM.
- ```-cdrom – use iso image as cdrom to install ubuntu```: specifies that we are using the ISO image as a CD-ROM to install Ubuntu.
- ```-m 2048```: allocates 2048 MB of RAM for the virtual machine.
- ```-accel [accel]```: specifies the accelerator for virtual machine execution. In this case, it is not specified, so the default value is used. QEMU supports multiple accelerator options, including KVM, Xen, and TCG. Here, by default qemu uses TCG.

---
## OS Virtualization (Docker) Setup

In this assignment, the setup of Docker was performed using the following two documentations:

- https://docs.docker.com/engine/install/fedora/
- https://developer.fedoraproject.org/tools/docker/docker-installation.html

The first step involved uninstalling any older versions of Docker along with associated dependencies. The latest version of Docker was then installed from its repository. To verify the installation, the hello-world image was executed to ensure that Docker was functioning as expected.

#### Setting up the Docker Container

To set up the Docker container, the following steps were taken:

- Verification of image creation was performed using the command `sudo docker images`.
- The Docker container was created using the command `sudo docker run -it ubuntu:20.04`. This pulled the ubuntu:20.04 image, created a container from it, and ran the Bash shell inside it.
- The base ubuntu image was utilized to create a new image with Sysbench installed by updating the base image using the command `apt update` and installing Sysbench using the command `apt install sysbench`.
- The new image was created using the command `sudo docker commit <container_id> my_image_with_sysbench`, where `<container_id>` is the ID of the running container.
- The history of the newly created image was inspected using the command `docker history my_image_with_sysbench`.

The following commands were used to list the running containers and verify the images available locally:

- `sudo docker ps`: To list running containers
- `sudo docker images`: To see the available images locally.

![docker_start_01](C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_start_01.png)*docker_start_01*

![docker_start_02](C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_start_02.png)*docker_start_02*

---
## Benchmarking
---
Prior to initiating the benchmarking process, it was ensured that both QEMU and Docker were utilizing the same versions of Ubuntu 20.04.5 LTS and Sysbench 1.0.18.

#### QEMU setup experiments:
After conducting a series of tests with various arguments, it was found that the use of the -accel kvm option significantly improved the performance of the virtual machine(attached the screenshots below). The use of the Kernel-based Virtual Machine (KVM) accelerator provided a significant increase in the speed and responsiveness of the virtual machine compared to the default accelerator tcg tested. Even the boot times were blazingly fast after using the kvm accelerator.


| VM Configuration          | Events per second |
| ------------------------- | ----------- |
| with kvm accelerator      | 37803.80       |
| with default(tcg) accelerator   | 7530.89      |
| with kvm accelerator and smp set to max=12     | 37558.62      |

Due to minimal change observed in increasing the number of hot-pluggable CPUs to its maximum of 12 using SMP for KVM, we will proceed with utilizing the KVM accelerator for the remaining benchmark experiments. We use the following command to start the Ubuntu VM
```sh
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img -accel kvm
```

![qemu_kvm](C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_kvm.png)*qemu_kvm*

![qemu_without_kvm](C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_without_kvm.png)*qemu_without_kvm*

![qemu_kvm_smp_max_12](C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_kvm_smp_max_12.png)*qemu_kvm_smp_max_12*

---

#### Scenarios and Test cases:

Performance data collection can be performed using the sysbench tool, which provides various modes for measuring CPU utilization and I/O performance.

To measure CPU utilization, we can use the following sysbench commands:

- ```sysbench cpu```: Measures user-level CPU performance by executing mathematical operations. Results include the number of events executed per second and the total time taken to execute the events.
- ```sysbench fileio```: Measures kernel-level CPU utilization during file I/O operations. Results include information about the CPU utilization during these operations.

To measure I/O performance, we can use the following sysbench command:

- ```sysbench fileio```: Measures file I/O performance. Results include I/O throughput, latency, and disk utilization, with I/O throughput being the number of I/O operations per second, latency being the average time taken to complete an I/O operation, and disk utilization being the amount of disk space used during the test.

In conclusion, sysbench provides valuable information for performance analysis and optimization through its various modes for measuring CPU utilization and I/O performance. The code below contains the test cases for the first scenario, and the remaining test cases and scenarios can be found in the GitHub repository.

---

#### Testcases for Scenarios
```sh
#test-case-01-cpu-2000
sysbench --test=cpu --cpu-max-prime=2000 --time=30 run  
#test-case-02-cpu-20000
sysbench --test=cpu --cpu-max-prime=20000 --time=30 run
#test-case-03-cpu-100000
sysbench --test=cpu --cpu-max-prime=100000 --time=30 run 
#test-case-04-io-rndrw
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw cleanup
#test-case-05-io-seqrewr
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr cleanup
```
---

#### Scenarios

Shell scripts have been developed to automate the repetition of necessary commands. Each test case must be executed five times, and the corresponding shell scripts can be found in the GitHub repository. The use of these scripts streamlines the process of repeating the tests multiple times and ensures consistency in the data collected.

In QEMU, virtual scenarios will be established during the start-up of the virtual machine (VM), using the following commands:
```sh
#scenario-1 : 2 GiB of RAM allocated with kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img -accel kvm
#scenario-2 : 2 GiB of RAM allocated without kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img
#scenario-3 : 8 GiB of RAM allocated with kvm accelerator and smp set to max=12 
$ sudo qemu-system-x86_64 -m 8192 -hda ubuntu.img -accel kvm -smp 12
```
Whereas for Docker Virtualizations, the script files innately allocate the set amounts of memory(RAM), and CPUs for their respective scenarios. The following commands are used to start-up the three different docker container scenarios.
```sh
#scenario-1 : 2 GiB of RAM, 2 CPUs allocated
$ sudo docker run -it --cpus="2" --memory="2g" my_image_with_sysbench:latest 
#scenario-2 : 4 GiB of RAM, 4 CPUs allocated
$ sudo docker run -it --cpus="4" --memory="4g" my_image_with_sysbench:latest 
#scenario-3 : 8 GiB of RAM, 8 CPUs allocated
$ sudo docker run -it --cpus="8" --memory="8g" my_image_with_sysbench:latest 
```

---

# Screenshots QEMU:

The following report depicts screenshots with tables to showcase average values for each scenario and testcase in QEMU Virtualization.

---

# Scenario - 1 & Testcase - 1

These screenshots depict five iterations of testcase 1 in scenario 1 in QEMU VM

```sh
#scenario-1 : 2 GiB of RAM allocated with kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img -accel kvm
#test-case-01-cpu-2000
sysbench --test=cpu --cpu-max-prime=2000 --time=30 run  
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_111.png" alt="qemu_kvm_smp_max_12" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_112.png" alt="qemu_scenario_01" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_113.png" alt="qemu_scenario_01" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_114.png" alt="qemu_scenario_01" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_115.png" alt="qemu_scenario_01" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second  |
| ------------------------- | ------------------ |
| 1                         | 34629.34           |
| 2                         | 34513.20 (**MIN**) |
| 3                         | 34699.11 (**MAX**) |
| 4                         | 34655.88           |
| 5                         | 34664.02           |
| Average Events per second | 34632.31           |

---

# Scenario - 1 & Testcase - 2

These screenshots depict five iterations of testcase 2 in scenario 1 in QEMU VM

```sh
#scenario-1 : 2 GiB of RAM allocated with kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img -accel kvm
#test-case-02-cpu-20000
sysbench --test=cpu --cpu-max-prime=20000 --time=30 run
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_121.png" alt="qemu_121" style="zoom:50%;" /> <img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_122.png" alt="qemu_122" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_123.png" alt="qemu_123" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_124.png" alt="qemu_124" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_125.png" alt="qemu_125" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 1330.53           |
| 2                         | 1338.58 (**MAX**) |
| 3                         | 1336.07           |
| 4                         | 1336.40           |
| 5                         | 1328.61 (**MIN**) |
| Average Events per second | 1334.038          |

---

# Scenario - 1 & Testcase - 3

These screenshots depict five iterations of testcase 3 in scenario 1 in QEMU VM

```sh
#scenario-1 : 2 GiB of RAM allocated with kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img -accel kvm
#test-case-03-cpu-100000
sysbench --test=cpu --cpu-max-prime=100000 --time=30 run 
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_131.png" alt="qemu_131" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_132.png" alt="qemu_132" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_133.png" alt="qemu_133" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_134.png" alt="qemu_134" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_135.png" alt="qemu_135" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 145.37            |
| 2                         | 145.76            |
| 3                         | 145.88 (**MAX**)  |
| 4                         | 145.36 (**MIN**)  |
| 5                         | 145.49            |
| Average Events per second | 145.57            |

---

# Scenario - 1 & Testcase - 4

These screenshots depict five iterations of testcase 4 in scenario 1 in QEMU VM

```sh
#scenario-1 : 2 GiB of RAM allocated with kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img -accel kvm
#test-case-04-io-rndrw
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_141.png" alt="qemu_141" style="zoom: 50%;" /> <img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_142.png" alt="qemu_142" style="zoom: 50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_143.png" alt="qemu_143" style="zoom: 50%;" /> <img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_144.png" alt="qemu_144" style="zoom: 50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_145.png" alt="qemu_145" style="zoom: 50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | reads/s | writes/s | fsyncs/s | throughput (read) | written |
| -------------- | ------- | -------- | -------- | ----------------- | ------- |
| 1              | 1268.01 | 845.23   | 2772.79  | 19.81             | 13.21   |
| 2              | 1307.25 | 871.78   | 2856.65  | 20.43             | 13.62   |
| 3              | 1363.35 | 908.84   | 2975.71  | 21.30             | 14.20   |
| 4              | 1343.62 | 895.58   | 2933.65  | 20.99             | 13.99   |
| 5              | 1303.92 | 869.67   | 2847.20  | 20.37             | 13.59   |
| Average values | 1317.23 | 878.22   | 2877.2   | 20.57             | 13.71   |

---

# Scenario - 1 & Testcase - 5

These screenshots depict five iterations of testcase 5 in scenario 1 in QEMU VM

```sh
#scenario-1 : 2 GiB of RAM allocated with kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img -accel kvm
#test-case-05-io-seqrewr
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_151.png" alt="docker_151" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_152.png" alt="docker_152" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_153.png" alt="docker_153" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_154.png" alt="docker_154" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_155.png" alt="docker_155" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | writes/s | fsyncs/s | Throughput (written) |
| -------------- | -------- | -------- | -------------------- |
| 1              | 6165.17  | 7958.97  | 96.33                |
| 2              | 6216.73  | 8022.13  | 97.14                |
| 3              | 6148.91  | 7934.55  | 96.08                |
| 4              | 5681.68  | 7336.67  | 88.78                |
| 5              | 6163.76  | 7954.70  | 96.31                |
| average values | 6075.25  | 7841.4   | 93.76                |

---

# Scenario - 2 & Testcase - 1

These screenshots depict five iterations of testcase 1 in scenario 2 in QEMU VM

```sh
#scenario-2 : 2 GiB of RAM allocated without kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img
#test-case-01-cpu-2000
sysbench --test=cpu --cpu-max-prime=2000 --time=30 run
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_211.png" alt="docker_211" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_212.png" alt="docker_212" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_213.png" alt="docker_213" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_214.png" alt="docker_214" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_215.png" alt="docker_215" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 6851.15 (**MAX**) |
| 2                         | 6839.86           |
| 3                         | 6845.34           |
| 4                         | 6819.65           |
| 5                         | 6459.04 (**MIN**) |
| Average Events per second | 6763              |

---

# Scenario - 2 & Testcase - 2

These screenshots depict five iterations of testcase 2 in scenario 2 in QEMU VM

```sh
#scenario-2 : 2 GiB of RAM allocated without kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img
#test-case-02-cpu-20000
sysbench --test=cpu --cpu-max-prime=20000 --time=30 run
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_221.png" alt="qemu_221" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_222.png" alt="qemu_222" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_223.png" alt="qemu_223" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_224.png" alt="qemu_224" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_225.png" alt="qemu_225" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 345.15 (**MAX**)  |
| 2                         | 341.38            |
| 3                         | 340.14            |
| 4                         | 340.64            |
| 5                         | 338.87 (**MIN**)  |
| Average Events per second | 341.23            |

---

# Scenario - 2 & Testcase - 3

These screenshots depict five iterations of testcase 3 in scenario 2 in QEMU VM

```sh
#scenario-2 : 2 GiB of RAM allocated without kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img
#test-case-03-cpu-100000
sysbench --test=cpu --cpu-max-prime=100000 --time=30 run 
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_231.png" alt="qemu_231" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_232.png" alt="qemu_232" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_233.png" alt="qemu_233" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_234.png" alt="qemu_234" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_235.png" alt="qemu_235" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 38.39             |
| 2                         | 39.21             |
| 3                         | 38.75 (**MAX**)   |
| 4                         | 39.37 (**MIN**)   |
| 5                         | 39.03             |
| Average Events per second | 38.95             |

---

# Scenario - 2 & Testcase - 4

These screenshots depict five iterations of testcase 4 in scenario 2 in QEMU VM

```sh
#scenario-2 : 2 GiB of RAM allocated without kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img
#test-case-04-io-rndrw
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_241.png" alt="qemu_241" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_242.png" alt="qemu_242" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_243.png" alt="qemu_243" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_244.png" alt="qemu_244" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_245.png" alt="qemu_245" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | reads/s | writes/s | fsyncs/s | throughput (read) | written |
| -------------- | ------- | -------- | -------- | ----------------- | ------- |
| 1              | 1115.30 | 743.49   | 2445.82  | 17.43             | 11.62   |
| 2              | 1139.14 | 759.15   | 2493.97  | 17.80             | 11.86   |
| 3              | 1074.11 | 715.80   | 2357.69  | 16.78             | 11.18   |
| 4              | 1175.85 | 783.74   | 2575.63  | 18.37             | 12.25   |
| 5              | 1147.09 | 764.39   | 2510.23  | 17.92             | 11.94   |
| average values | 1139.42 | 759.36   | 2481.57  | 17.71             | 11.86   |

---

# Scenario - 2 & Testcase - 5

These screenshots depict five iterations of testcase 5 in scenario 2 in QEMU VM

```sh
#scenario-2 : 2 GiB of RAM allocated without kvm accelerator
$ sudo qemu-system-x86_64 -m 2048 -hda ubuntu.img
#test-case-05-io-seqrewr
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_251.png" alt="qemu_251" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_252.png" alt="qemu_252" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_253.png" alt="qemu_253" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_254.png" alt="qemu_254" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_255.png" alt="qemu_255" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | writes/s | fsyncs/s | Throughput (written) |
| -------------- | -------- | -------- | -------------------- |
| 1              | 4044.71  | 5240.81  | 63.20                |
| 2              | 4304.29  | 5575.04  | 67.25                |
| 3              | 4272.63  | 5532.64  | 66.76                |
| 4              | 4184.37  | 5423.13  | 65.38                |
| 5              | 3973.49  | 5152.30  | 62.09                |
| average values | 4121.52  | 5368.68  | 64.36                |

---

# Scenario - 3 & Testcase - 1

These screenshots depict five iterations of testcase 1 in scenario 3 in QEMU VM

```sh
#scenario-3 : 8 GiB of RAM allocated with kvm accelerator and smp set to max=12 
$ sudo qemu-system-x86_64 -m 8192 -hda ubuntu.img -accel kvm -smp 12
#test-case-01-cpu-2000
sysbench --test=cpu --cpu-max-prime=2000 --time=30 run  
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_311.png" alt="qemu_311" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_312.png" alt="qemu_312" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_313.png" alt="qemu_313" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_314.png" alt="qemu_314" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_315.png" alt="qemu_315" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second  |
| ------------------------- | ------------------ |
| 1                         | 37580.52 (**MAX**) |
| 2                         | 36778.92           |
| 3                         | 35798.67           |
| 4                         | 34213.62 (**MIN**) |
| 5                         | 34251.01           |
| Average Events per second | 35724.55           |

---

# Scenario - 3 & Testcase - 2

These screenshots depict five iterations of testcase 2 in scenario 3 in QEMU VM

```sh
#scenario-3 : 8 GiB of RAM allocated with kvm accelerator and smp set to max=12 
$ sudo qemu-system-x86_64 -m 8192 -hda ubuntu.img -accel kvm -smp 12
#test-case-02-cpu-20000
sysbench --test=cpu --cpu-max-prime=20000 --time=30 run
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_321.png" alt="docker_321" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_322.png" alt="docker_322" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_323.png" alt="docker_323" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_324.png" alt="docker_324" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_325.png" alt="docker_325" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 1324.87           |
| 2                         | 1324.12           |
| 3                         | 1321.18 (**MIN**) |
| 4                         | 1325.20 (**MAX**) |
| 5                         | 1324.87           |
| Average Events per second | 1323.89           |

---

# Scenario - 3 & Testcase - 3

These screenshots depict five iterations of testcase 3 in scenario 3 in QEMU VM

```sh
#scenario-3 : 8 GiB of RAM allocated with kvm accelerator and smp set to max=12 
$ sudo qemu-system-x86_64 -m 8192 -hda ubuntu.img -accel kvm -smp 12
#test-case-03-cpu-100000
sysbench --test=cpu --cpu-max-prime=100000 --time=30 run 
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_331.png" alt="qemu_331" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_332.png" alt="qemu_332" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_333.png" alt="qemu_333" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_334.png" alt="qemu_334" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_335.png" alt="qemu_335" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 144.31            |
| 2                         | 144.35 (**MAX**)  |
| 3                         | 144.11 (**MIN**)  |
| 4                         | 144.20            |
| 5                         | 144.22            |
| Average Events per second | 144.24            |

---

# Scenario - 3 & Testcase - 4

These screenshots depict five iterations of testcase 4 in scenario 3 in QEMU VM

```sh
#scenario-3 : 8 GiB of RAM allocated with kvm accelerator and smp set to max=12 
$ sudo qemu-system-x86_64 -m 8192 -hda ubuntu.img -accel kvm -smp 12
#test-case-04-io-rndrw
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_341.png" alt="qemu_341" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_342.png" alt="qemu_342" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_343.png" alt="qemu_343" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_344.png" alt="qemu_344" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\qemu_345.png" alt="qemu_345" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | reads/s | writes/s | fsyncs/s | throughput (read) | written |
| -------------- | ------- | -------- | -------- | ----------------- | ------- |
| 1              | 629.51  | 419.67   | 1407.03  | 9.84              | 6.56    |
| 2              | 654.50  | 436.33   | 1459.90  | 10.23             | 6.82    |
| 3              | 792.35  | 528.34   | 1755.16  | 12.38             | 8.26    |
| 4              | 762.83  | 508.55   | 1692.39  | 11.92             | 7.95    |
| 5              | 757.54  | 505.08   | 1681.78  | 11.84             | 7.89    |
| average values | 719.98  | 492.17   | 1592.25  | 11.05             | 7.45    |

---

# Scenario - 3 & Testcase - 5

These screenshots depict five iterations of testcase 5 in scenario 3 in QEMU VM

```sh
#scenario-3 : 8 GiB of RAM allocated with kvm accelerator and smp set to max=12 
$ sudo qemu-system-x86_64 -m 8192 -hda ubuntu.img -accel kvm -smp 12
#test-case-05-io-seqrewr
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_351.png" alt="docker_351" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_352.png" alt="docker_352" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_353.png" alt="docker_353" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_354.png" alt="docker_354" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_355.png" alt="docker_355" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | writes/s | fsyncs/s | Throughput (written) |
| -------------- | -------- | -------- | -------------------- |
| 1              | 5376.45  | 6945.66  | 84.01                |
| 2              | 5478.02  | 7079.74  | 85.59                |
| 3              | 5337.46  | 6899.28  | 83.40                |
| 4              | 4820.56  | 6235.70  | 75.32                |
| 5              | 5197.70  | 6717.29  | 81.21                |
| average values | 5223.56  | 6776.57  | 81.19                |

---

# Screenshots Docker:

The following report depicts screenshots with tables to showcase average values for each scenario and testcase in Docker Virtualization.

# Scenario - 1 & Testcase - 1

These screenshots depict five iterations of testcase 1 in scenario 1 in Docker.

```sh
#scenario-1 : 2 GiB of RAM, 2 CPUs allocated
$ sudo docker run -it --cpus="2" --memory="2g" my_image_with_sysbench:latest 
#test-case-01-cpu-2000
sysbench --test=cpu --cpu-max-prime=2000 --time=30 run  
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_111.png" alt="docker_111" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_112.png" alt="docker_112" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_113.png" alt="docker_113" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_114.png" alt="docker_114" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_115.png" alt="docker_115" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second  |
| ------------------------- | ------------------ |
| 1                         | 35322.89           |
| 2                         | 35298.54           |
| 3                         | 35433.27 (**MAX**) |
| 4                         | 35279.87 (**MIN**) |
| 5                         | 35406.51           |
| Average Events per second | 35328.82           |

---

# Scenario - 1 & Testcase - 2

These screenshots depict five iterations of testcase 2 in scenario 1 in Docker.

```sh
#scenario-1 : 2 GiB of RAM, 2 CPUs allocated
$ sudo docker run -it --cpus="2" --memory="2g" my_image_with_sysbench:latest
#test-case-02-cpu-20000
sysbench --test=cpu --cpu-max-prime=20000 --time=30 run
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_121.png" alt="docker_121" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_122.png" alt="docker_122" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_123.png" alt="docker_123" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_124.png" alt="docker_124" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_125.png" alt="docker_125" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 1372.64           |
| 2                         | 1370.77 (**MIN**) |
| 3                         | 1371.07           |
| 4                         | 1371.45           |
| 5                         | 1377.65 (**MAX**) |
| Average Events per second | 1372.7            |

---

# Scenario - 1 & Testcase - 3

These screenshots depict five iterations of testcase 3 in scenario 1 in Docker.

```sh
#scenario-1 : 2 GiB of RAM, 2 CPUs allocated
$ sudo docker run -it --cpus="2" --memory="2g" my_image_with_sysbench:latest
#test-case-03-cpu-100000
sysbench --test=cpu --cpu-max-prime=100000 --time=30 run 
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_131.png" alt="docker_131" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_132.png" alt="docker_132" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_133.png" alt="docker_133" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_134.png" alt="docker_134" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_135.png" alt="docker_135" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 149.38 (**MAX**)  |
| 2                         | 148.95            |
| 3                         | 148.79            |
| 4                         | 145.97            |
| 5                         | 144.90 (**MIN**)  |
| Average Events per second | 147.25            |

---

# Scenario - 1 & Testcase - 4

These screenshots depict five iterations of testcase 4 in scenario 1 in Docker.

```sh
#scenario-1 : 2 GiB of RAM, 2 CPUs allocated
$ sudo docker run -it --cpus="2" --memory="2g" my_image_with_sysbench:latest
#test-case-04-io-rndrw
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_141.png" alt="docker_141" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_142.png" alt="docker_142" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_143.png" alt="docker_143" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_144.png" alt="docker_144" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_145.png" alt="docker_145" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | reads/s | writes/s | fsyncs/s | throughput (read) | written |
| -------------- | ------- | -------- | -------- | ----------------- | ------- |
| 1              | 4014.51 | 2676.32  | 8632.03  | 62.73             | 41.82   |
| 2              | 4077.70 | 2718.30  | 8765.67  | 63.71             | 42.47   |
| 3              | 4155.93 | 2770.56  | 8930.60  | 64.94             | 43.29   |
| 4              | 4274.56 | 2849.55  | 9183.36  | 66.79             | 44.52   |
| 5              | 4283.82 | 2855.82  | 9205.61  | 66.93             | 44.62   |
| average values | 4159.50 | 2733.91  | 8603.25  | 64.82             | 43.34   |

---

# Scenario - 1 & Testcase - 5

These screenshots depict five iterations of testcase 5 in scenario 1 in Docker.

```sh
#scenario-1 : 2 GiB of RAM, 2 CPUs allocated
$ sudo docker run -it --cpus="2" --memory="2g" my_image_with_sysbench:latest
#test-case-05-io-seqrewr
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_151.png" alt="docker_151" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_152.png" alt="docker_152" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_153.png" alt="docker_153" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_154.png" alt="docker_154" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_155.png" alt="docker_155" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | writes/s | fsyncs/s | Throughput (written) |
| -------------- | -------- | -------- | -------------------- |
| 1              | 8369.80  | 10780.60 | 130.78               |
| 2              | 9007.50  | 11595.78 | 140.74               |
| 3              | 12171.40 | 15644.24 | 190.18               |
| 4              | 12664.69 | 16277.67 | 197.89               |
| 5              | 10215.55 | 13143.03 | 159.62               |
| average values | 10183.68 | 13449.26 | 163.66               |

---

# Scenario - 2 & Testcase - 1

These screenshots depict five iterations of testcase 1 in scenario 2 in Docker.

```sh
#scenario-2 : 4 GiB of RAM, 4 CPUs allocated
$ sudo docker run -it --cpus="4" --memory="4g" my_image_with_sysbench:latest
#test-case-01-cpu-2000
sysbench --test=cpu --cpu-max-prime=2000 --time=30 run  
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_211.png" alt="docker_211" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_212.png" alt="docker_212" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_213.png" alt="docker_213" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_214.png" alt="docker_214" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_215.png" alt="docker_215" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second  |
| ------------------------- | ------------------ |
| 1                         | 37998.84 (**MAX**) |
| 2                         | 37378.79           |
| 3                         | 36582.26           |
| 4                         | 34862.93           |
| 5                         | 34762.97 (**MIN**) |
| Average Events per second | 36697.35           |

---

# Scenario - 2 & Testcase - 2

These screenshots depict five iterations of testcase 2 in scenario 2 in Docker.

```sh
#scenario-2 : 4 GiB of RAM, 4 CPUs allocated
$ sudo docker run -it --cpus="4" --memory="4g" my_image_with_sysbench:latest
#test-case-02-cpu-20000
sysbench --test=cpu --cpu-max-prime=20000 --time=30 run
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_211.png" alt="docker_211" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_212.png" alt="docker_212" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_213.png" alt="docker_213" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_214.png" alt="docker_214" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_215.png" alt="docker_215" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 1341.13           |
| 2                         | 1341.25           |
| 3                         | 1343.26 (**MAX**) |
| 4                         | 1339.58           |
| 5                         | 1337.59 (**MIN**) |
| Average Events per second | 1340.56           |

---

# Scenario - 2 & Testcase - 3

These screenshots depict five iterations of testcase 3 in scenario 2 in Docker.

```sh
#scenario-2 : 4 GiB of RAM, 4 CPUs allocated
$ sudo docker run -it --cpus="4" --memory="4g" my_image_with_sysbench:latest
#test-case-03-cpu-100000
sysbench --test=cpu --cpu-max-prime=100000 --time=30 run 
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_231.png" alt="docker_231" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_232.png" alt="docker_232" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_233.png" alt="docker_233" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_234.png" alt="docker_234" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_235.png" alt="docker_235" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 145.87            |
| 2                         | 145.07 (**MIN**)  |
| 3                         | 145.88            |
| 4                         | 145.90 (**MAX**)  |
| 5                         | 145.63            |
| Average Events per second | 145.67            |

---

---

# Scenario - 2 & Testcase - 4

These screenshots depict five iterations of testcase 4 in scenario 2 in Docker.

```sh
#scenario-2 : 4 GiB of RAM, 4 CPUs allocated
$ sudo docker run -it --cpus="4" --memory="4g" my_image_with_sysbench:latest
#test-case-04-io-rndrw
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_241.png" alt="docker_241" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_242.png" alt="docker_242" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_243.png" alt="docker_243" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_244.png" alt="docker_244" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_245.png" alt="docker_245" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | reads/s | writes/s | fsyncs/s | throughput (read) | written |
| -------------- | ------- | -------- | -------- | ----------------- | ------- |
| 1              | 6064.81 | 4043.32  | 13005.05 | 94.76             | 63.18   |
| 2              | 6585.60 | 4390.62  | 14114.68 | 102.90            | 68.60   |
| 3              | 6043.09 | 4028.89  | 12960.24 | 94.42             | 62.95   |
| 4              | 6030.51 | 4020.51  | 12930.10 | 94.23             | 62.82   |
| 5              | 6862.09 | 4574.84  | 14706.28 | 107.22            | 71.48   |
| average values | 6317.22 | 4211.63  | 13543.27 | 98.70             | 65.80   |

---

# Scenario - 2 & Testcase - 5

These screenshots depict five iterations of testcase 5 in scenario 2 in Docker.

```sh
#scenario-2 : 4 GiB of RAM, 4 CPUs allocated
$ sudo docker run -it --cpus="4" --memory="4g" my_image_with_sysbench:latest
#test-case-05-io-seqrewr
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_251.png" alt="docker_251" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_252.png" alt="docker_252" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_253.png" alt="docker_253" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_254.png" alt="docker_254" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_255.png" alt="docker_255" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | writes/s | fsyncs/s  | Throughput (written) |
| -------------- | -------- | --------- | -------------------- |
| 1              | 77637.75 | 99442.19  | 1213.09              |
| 2              | 92047.52 | 117885.93 | 1438.24              |
| 3              | 70251.19 | 89988.60  | 1097.67              |
| 4              | 94544.84 | 121081.77 | 1477.26              |
| 5              | 71445.31 | 91514.42  | 1116.33              |
| average values | 83184.66 | 105870.74 | 1306.44              |

---

# Scenario - 3 & Testcase - 1

These screenshots depict five iterations of testcase 1 in scenario 3 in Docker.

```sh
#scenario-3 : 8 GiB of RAM, 8 CPUs allocated
$ sudo docker run -it --cpus="8" --memory="8g" my_image_with_sysbench:latest 
#test-case-01-cpu-2000
sysbench --test=cpu --cpu-max-prime=2000 --time=30 run  
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_311.png" alt="docker_311" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_312.png" alt="docker_312" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_313.png" alt="docker_313" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_314.png" alt="docker_314" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_315.png" alt="docker_315" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second  |
| ------------------------- | ------------------ |
| 1                         | 37396.15 (**MIN**) |
| 2                         | 37902.02 (**MAX**) |
| 3                         | 37542.24           |
| 4                         | 37713.42           |
| 5                         | 37474.99           |
| Average Events per second | 37605.76           |

---

# Scenario - 3 & Testcase - 2

These screenshots depict five iterations of testcase 2 in scenario 3 in Docker.

```sh
#scenario-3 : 8 GiB of RAM, 8 CPUs allocated
$ sudo docker run -it --cpus="8" --memory="8g" my_image_with_sysbench:latest 
#test-case-02-cpu-20000
sysbench --test=cpu --cpu-max-prime=20000 --time=30 run
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_321.png" alt="docker_321" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_322.png" alt="docker_322" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_323.png" alt="docker_323" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_324.png" alt="docker_324" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_325.png" alt="docker_325" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 1460.97 (**MAX**) |
| 2                         | 1449.83 (**MIN**) |
| 3                         | 1457.00           |
| 4                         | 1458.55           |
| 5                         | 1456.04           |
| Average Events per second | 1456.47           |

---

# Scenario - 3 & Testcase - 3

These screenshots depict five iterations of testcase 3 in scenario 3 in Docker.

```sh
#scenario-3 : 8 GiB of RAM, 8 CPUs allocated
$ sudo docker run -it --cpus="8" --memory="8g" my_image_with_sysbench:latest 
#test-case-03-cpu-100000
sysbench --test=cpu --cpu-max-prime=100000 --time=30 run 
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_331.png" alt="docker_331" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_332.png" alt="docker_332" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_333.png" alt="docker_333" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_334.png" alt="docker_334" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_335.png" alt="docker_335" style="zoom:50%;" />

The following table shows the Events per second for test case scenario.

| Iteration                 | Events per second |
| ------------------------- | ----------------- |
| 1                         | 159.17 (**MAX**)  |
| 2                         | 156.93            |
| 3                         | 154.49            |
| 4                         | 145.77 (**MIN**)  |
| 5                         | 146.14            |
| Average Events per second | 152.5             |

---

# Scenario - 3 & Testcase - 4

These screenshots depict five iterations of testcase 4 in scenario 3 in Docker.

```sh
#scenario-3 : 8 GiB of RAM, 8 CPUs allocated
$ sudo docker run -it --cpus="8" --memory="8g" my_image_with_sysbench:latest 
#test-case-04-io-rndrw
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=rndrw cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_341.png" alt="docker_341" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_342.png" alt="docker_342" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_343.png" alt="docker_343" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_344.png" alt="docker_344" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_345.png" alt="docker_345" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | reads/s | writes/s | fsyncs/s | throughput (read) | written |
| -------------- | ------- | -------- | -------- | ----------------- | ------- |
| 1              | 6531.31 | 4354.32  | 13998.97 | 102.05            | 68.04   |
| 2              | 6384.95 | 4256.63  | 13689.39 | 99.76             | 66.51   |
| 3              | 6079.78 | 4053.35  | 13035.87 | 95.00             | 63.33   |
| 4              | 6846.11 | 4564.13  | 14669.32 | 106.97            | 71.31   |
| 5              | 6596.95 | 4398.08  | 14140.01 | 103.08            | 68.72   |
| average values | 6386.72 | 4338.05  | 13875.28 | 100.51            | 67.52   |

---

# Scenario - 3 & Testcase - 5

These screenshots depict five iterations of testcase 5 in scenario 3 in Docker.

```sh
#scenario-3 : 8 GiB of RAM, 8 CPUs allocated
$ sudo docker run -it --cpus="8" --memory="8g" my_image_with_sysbench:latest 
#test-case-05-io-seqrewr
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr prepare
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr run
sysbench --num-threads=16 --test=fileio --file-total-size=3G --time=30 --file-test-mode=seqrewr cleanup
```

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_351.png" alt="docker_351" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_352.png" alt="docker_352" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_353.png" alt="docker_353" style="zoom:50%;" /><img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_354.png" alt="docker_354" style="zoom:50%;" />

<img src="C:\Users\Rachana Machines\Downloads\Screenshots\Screenshots\docker_355.png" alt="docker_355" style="zoom:50%;" />

The following table shows the performance evaluations for test case scenario.

| Iteration      | writes/s  | fsyncs/s  | Throughput (written) |
| -------------- | --------- | --------- | -------------------- |
| 1              | 145880.05 | 186790.89 | 2279.38              |
| 2              | 146650.91 | 187777.41 | 2291.42              |
| 3              | 145643.39 | 186490.56 | 2275.68              |
| 4              | 144980.11 | 185640.06 | 2265.31              |
| 5              | 144366.74 | 184857.52 | 2255.73              |
| average values | 145505.23 | 186244.62 | 2274.57              |

---

# Conclusions

After an extensive evaluation of the benchmarking results, it can be concluded that scenario 3 in both the system virtualization (QEMU) and OS virtualization (Docker) offered the highest level of performance when compared to the other two scenarios.

In terms of CPU utilization, the OS virtualization provided slightly better results than the system virtualization. However, when it came to disk utilization, specifically in terms of file I/O, the OS virtualization outperformed the system virtualization significantly.

These findings indicate that while both virtualization approaches have their strengths and weaknesses, in this particular scenario, the OS virtualization offered a higher level of performance.

It is important to note that this conclusion was drawn based on a specific set of criteria and benchmarking data. The results may differ in other scenarios, depending on the system specifications, configuration, and usage patterns. Thus, it is recommended to evaluate virtualization options on a case-by-case basis, considering the specific needs and requirements of the system.

Overall, this report serves as a valuable resource for those seeking to understand the relative performance of system virtualization and OS virtualization. The results can be used to make informed decisions regarding the selection and deployment of virtualization technology in different scenarios.

---

# Github Repository Information

Repository name : cloud-class

https://github.com/rahulyal/cloud-class.git
