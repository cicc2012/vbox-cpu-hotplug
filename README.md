# vbox-cpu-hotplug

## Prerequisites

We assume you have created a VirtualBox virtual machine, so that we can try to adjust virtual hardware resources on the fly. More specially, we will adjust the CPU capacity while the virtual machine is running, which is hard for the traditional physical machine.

### 1. Prepare CPU hot plug feature for your VM.

By default, CPU Hot-Plugging feature is not enabled for your VM, once it's created. We need to make some preparations for it.

We can't make use of this feature in the graphical interface of VirtualBox. Instead, we can use the command line tool, VBoxManage, to manipulate the VM.

This tool may not be usable directly. If that happens, you can include the installation path of VirtualBox into the environment variables in your operating system, to fix it.

Once this is done, keep your VM powered off for a moment, and you can use the following commands in your prompt / terminal to set the CPU capacity:

```bash
    VBoxManage modifyvm VM1 --cpu-hotplug on
    VBoxManage modifyvm VM1 --cpus 4
```

VM1 is the name of the VM you created, and this name is assigned and controlled by you. The number of CPUs can also be changed based on the condition of your CPU.

You only need to do this once.

### 2. Monitor the CPU usage of your VM.

Now let's start the VM.

```bash
    VBoxManage startvm VM1
```

To have resource provisioning, we need to first collect the performance data in term of usages of system resources. You can use the following commands to collect the target performance data:

```bash
    VBoxManage metrics enable VM1 Guest/CPU/Load/Idle
    VBoxManage metrics collect –period 2 –samples 5 VM1 Guest/CPU/Load/Idle
    VBoxManage metrics query VM1 Guest/CPU/Load/Idle > monitor.dat
```

There is no output after the first command.

The performance metric used here is Guest/CPU/Load/Idle. The data collection period is 2 seconds. For each record, there will be 5 data points (samples). These can all the be changed based on your design.

The "VBoxManage metrics collect" command will keep generating some outputs, and you can press Ctrl+C to stop (change it based on your OS). Once this is done, the data will be saved periodically in the background (not showed on the screen), so that you can repeat "VBoxManage metrics query" command to get the data again and again.

This is the third command in the table above, and the output is redirected to a file named monitor.dat, which can be changed as you need. 

Based on the collected performance data, once your program finds that the idle CPU resource is limited, e.g., lower than 10%, you can consider performing the CPU Hot-Plugging with the following command:

```
$ VBoxManage controlvm VM1 plugcpu 1
```

The number at the end is the index of the CPU. If there are totally 4 CPUs reserved for your VM, the indices will be from 0 to 3. Each time, you just plug in one more CPU with the corresponding index. You can plug in more CPUs when needed.

You can keep watching the performance data in your program. Once a new CPU core is plugged in, you can observe the change. If the idle CPU is constantly above a certain threshold, e.g., 85%, you can consider removing some CPU by:

```
$ VBoxManage controlvm VM1 unplugcpu 1
```

Change the VM name and index of CPU accordingly. But `CPU 0 can't be removed`.

There might be some errors after running this command for unplugging, which is because of missing VirtualBox Guest Additions for your VM. If that happens, go and install the guest additions.
