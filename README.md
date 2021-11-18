# CMPE 283 Assignments
 * [Assignment 1](#assignment-1)
 * [Assignment 2](#assignment-2)
 * [Assignment 3](#assignment-3)

## Assignment 1

### Question 1
I did the assignment myself

### Question 2 - Steps taken in order to complete assignment
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
12. Type `sudo make INSTALL_MOD_STRIP=1 modules_install` to package the Kernel modules without debugging information
13. Type `sudo make install` to install the Kernel
14. Type `sudo reboot` in order to use the new Kernel
15. Type `uname -a` to verify that the Machine is using the new Kernel that was built
    - The date listed should be whenever you built the kernel
    - The Kernel release version should be a higher number than from earlier
16. Copy the Makefile and cmpe283-1.c files from Canvas into the directory containing the /linux directory
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
18. Type `make` inside of the directory holding the `cmpe281-1.c` and 'Makefile' file
    - If there are errors make sure to fix them
19. Make sure a `cmpe283-1.ko` file is in the directory and type `sudo insmod cmpe283-1.ko`
20. Check if it installed properly with `sudo lsmod | grep cmpe283`
21. Check the output with the command `sudo dmesg`
22. Uninstall the module with `sudo rmmod cmpe281-1`

## Assignment 2

### Question 1
I did the assignment myself

### Question 2 - Steps taken in order to complete assignment
0. Install another virtual machine that's using KVM and have CPUID installed on the VM.
    - An easy way to install a VM via GUI is to install the `virt-manager` package since it uses KVM for the VMs it creates.
1. Figure out what CPU architecture you are using.
    - Use the command `cat /proc/cpuinfo` to find the vendorID
2. Figure out where CPUID is defined in the Linux Kernel
    - It is located in the `linux/arch/x86/kvm` directory for x86 systems.
3. Open the x86.c file and modify it to have 3 arrays that have 1024 values, one array for number of exits, the other two are for cpu cycle counts.
4. When the arrays are declared, make sure they are exported using `EXPORT_SYMBOL` because these need to be called in different modules.
5. Open cpuid.c and import the exported symbols using the `extern` keyword.
6. Inside of the `kvm_emulate_cpuid` function at the end of the file, declare two u32 variables and one int variable.
    - The two u32 variables are for counting the number of clock cycles and the int variable is for the for loop.
7. After the eax and ecx assignment lines, create a switch statement with the cases for `0x4fffffff`, `0x4ffffffe`, `0x4ffffffd`, `0x4ffffffc`
    - These are for the custom CPUID leaf values that we will create.
8. Inside of the case `0x4fffffff`, assign one of the u32 variables as 0 and create a for loop from 0 to 1024. On each iteration, add the value read to the u32 variable.
    - The value is 1024 because AMD uses VM Exit numbers from 0 to 1024
9. Assign eax the value of the u32 variable that was going through the for loop.
10. Inside of the case `0x4ffffffe`, assign both of the u32 variables as 0 and create a for loop from 0 to 1024. On each iteration, add the value of one array into a variable that will represent the high bits and the value of the other array into the other variable which will represent the low bits.
11. Assign the ebx value the high bits and the ecx value the low bits.
12. Inside of the default case, copy and pase the kvm_cpuid line.
13. Inside of svm/svm.c and vmx/vmx.c, extern the exported variables and find the functions that will handle exits.
    - In svm.c it will be called `handle_exit`
    - In vmx.c it will be called `__vmx_handle_exit`
14. In the functions that handles exits, figure out the exit code and create an if statement that will increment the array that represents the number of exits and use the exit_code as the array index.
15. At the top of the function that handles exits, create two u32 arrays of size 2. These will be used to count how many CPU cycles it takes for an exit to be handled. One will be for the start time of an exit and the other will be for the ending time of an exit.
    - One of the indices for both arrays will be representing the high bits and the other index will represent the low bits.
16. Above the function that handles exits, create a void function that will modify the arrays created. 
    - This function will use assembly to read the timestamp counter. And assign the high and low bits of the input array.
    - This StackOverflow [link](https://stackoverflow.com/questions/17401914/why-should-i-use-rdtsc-differently-on-x86-and-x86-x64) will help with creating the function.
17. Inside of the same if statement created earlier, call the newly created function and pass in the array that represents the start time.
18. At the end of the function that handles exits, create the same if statement from earlier and call the new function but pass in the array that represents the end time instead.
19. After calling the new function, with the corresponding externed array representing high and low bits of the cycle counter, add to the exit_code index the time it took to handle the exit.
    - An example `total_exits_time_hi[exit_code] = total_exits_time_hi[exit_code] + (end_time[1] - start_time[1]);`
20. After finishing the code, in the Linux Kernel root directory, build and install the modules using `make -j CPUcount modules` and `make INSTALL_MOD_STRIP=1 modules_install`
21. Remove the modules related to kvm using `rmmod`. You have to remove the architecture based module first, then do `rmmod kvm` to fully uninstall the kvm module.
    - For AMD remove the module `kvm_amd`
    - For Intel remove the module `kvm_intel`
22. Reinstall the modules by reversing the removal process with the `modprobe` command

## Assignment 3

### Question 1
I did the assignment myself

### Question 2 - Steps taken in order to complete assignment
