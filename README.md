# CMPE 283 Assignment 1

## Question 1
I did the assignment myself

## Question 2 - Steps taken in order to complete assignment
1. Fork the main linux repository on [Github](https://github.com/torvalds/linux)
2. Clone the repository you forked into a Linux machine or VM that has an Intel CPU made from 2015 onwards
3. Go into the cloned repository
    - Directory should be called 'linux'
4. Make sure you have the Linux Kernel development dependencies installed
    - Install the dependencies from this [URL](https://www.kernel.org/doc/html/latest/process/changes.html#current-minimal-requirements) if you are missing packages
    - install the package linux-headers-$(uname -r) to install linux headers if you don't have them inside of the /usr/src directory
    - If you are still missing dependencies later after doing the previous steps, install those dependencies
5. Inside a terminal, type `uname -r` in order to find the Linux Kernel release version that you have installed
6. Inside of the cloned linux directory, copy the Kernel release boot configuration from the /boot/ directory
    - `cp /boot/config-$(uname -r) .config`
7. Type `make oldconfig` and just hold enter until finished
8. Type `make prepare`
9. Find out how many CPUs your Machine has by typing `cat /proc/cpuinfo` or `htop` if you have htop installed.
10. Type `make -j CPUcount modules` which will build the Kernel modules
    - CPUcount should be replaced with the number of CPUs your machine can see ex. `make -j 12 modules`
11. Type `make -j CPUcount` which will build the Linux Kernel
    - You might have to edit .config by commeting out the line containing `CONFIG_SYSTEM_TRUSTED_KEYS="debian/certs/..."` if building on Debian
12. Type `sudo make INSTALL_MOD_STRIP=1 modules_install` to package a the Kernel without debugging information
13. Type `sudo make install` to install the Kernel
14. Type `sudo reboot` in order to use the new Kernel
15. Type `uname -a` to verify that the Machine is using the new Kernel that was built
    - The date listed should be whenever you built the kernel
    - The Kernel release version should be a higher number than from earlier
16. Copy the files from Canvas into the directory containing the /linux directory
    - Files should be on the same level as the /linux directory when typing `ls`
17. Inside of cmpe283-1.c do the following
    - At the end of the file, add a module license with the line
      - `MODULE_LICENSE("GPL_v2");`
    - Where the line `#define IA32_VMX_PINBASED_CTLS 0x481` is, add definitions for
      - `IA32_VMX_PROCBASED_CTLS` mapped to 0x482
      - `IA32_VMX_PROCBASED_CTLS2` mapped to 0x48B
      - `IA32_VMX_EXIT_CTLS` mapped to 0x483
      - `IA32_VMX_ENTRY_CTLS` mapped to 0x484
    - Copy the pinbased struct and rename the structs to procbased, procbased2, vmxexit, vmxentry. Paste the information from the Virtual Machine Control Structures from Volume 3 of the Intel SDM into the appropriate structs following the pinbased struct layout
      - The structs should have 22, 27, 14, and 12 entries as of October 2021.
      - Example is [here](https://github.com/pbustos97/linux/blob/master/cmpe283/cmpe283-1.c)
    - Inside of the `detect_vmx_features` function, copy the lines from `/* Pinbased controls */`to the `report_capability` line and paste it below the original lines until there are 5 copies of that function
    - Replace the parameters of `rdmsr` with the definitions for the copies
    - Replace the parameters of `report_capability` with the structs and their size
18. Type `make` inside of the directory holding the `cmpe281-1.c` file
    - If there are errors make sure to fix them
19. Make sure a `cmpe283-1.ko` file is in the directory and type `sudo insmod cmpe283-1.ko`
20. Check if it installed properly with `sudo lsmod | grep cmpe283`
21. Check the output with the command `sudo dmesg`
