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
3. Open the x86.c file and modify it to have 3 arrays that have 1025 values, one array for number of exits, the other two are for cpu cycle counts (one is for the high bits, the other for the low bits)
4. When the arrays are declared, make sure they are exported using `EXPORT_SYMBOL` because these need to be called in different modules.
5. Open cpuid.c and import the exported symbols using the `extern` keyword.
6. Inside of the `kvm_emulate_cpuid` function at the end of the file, declare two u32 variables and one int variable.
    - The two u32 variables are for counting the number of clock cycles and the int variable is for the for loop.
7. After the eax and ecx assignment lines, create a switch statement with the cases for `0x4fffffff`, `0x4ffffffe`, `0x4ffffffd`, `0x4ffffffc`
    - These are for the custom CPUID leaf values that we will create.
8. Inside of the case `0x4fffffff`, assign one of the u32 variables as 0 and create a for loop from 0 to 1024. On each iteration, add the value read from the array that is used for counting exits to the u32 variable.
    - The value is 1024 because AMD uses VM Exit numbers from 0 to 1024
9. Assign eax the value of the u32 variable that was going through the for loop.
10. Inside of the case `0x4ffffffe`, assign both of the u32 variables as 0 and create a for loop from 0 to 1024. On each iteration, add the value of the array that represents high bits into a variable that will represent the high bits and the value of the other array into the other variable which will represent the low bits.
11. Assign the ebx value the high bits and the ecx value the low bits.
12. Inside of the default case, copy and paste the kvm_cpuid line.
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
23. Run the newly installed VM, install CPUID if the OS uses the apt package manager, and run `cpuid -l 0x4fffffff` and `cpuid -l 0x4ffffffe`.
    - The results should be increasing each time you run the commands.
    - For the 0x4ffffffe leaf, ebx will stay 0 for a long time until the VM reaches the number of clock cycles required to max out the lower 32 bits of the timestamp counter

## Assignment 3

### Question 1
I did the assignment myself

### Question 2 - Steps taken in order to complete assignment
0. Make sure [Assignment 2](#assignment-2) is working.
1. Above the `kvm_emulate_cpuid` function, create a helper function that will check if an input ecx value is valid with the arguments of eax, ebx, ecx, edx. All passed as reference.
    - I made this function a boolean function.
2. For Intel machines, look at Intel SDM Volume 3 Appendix C, this will have all the valid VM exits that Intel allows on their CPUs.
3. Create an if statement that does not include the range between the min and max values presented in the SDM. As of November 2021, the min and max values are 0 and 69.
4. Inside of that if statement, assign eax, ebx, and ecx the values of `0x00000000` and edx the value of `0xffffffff` and return false. These are set as because these values will not be in the SDM
5. If the value is inside of the SDM range, create an if statement for the missing values from the SDM and return the same as step 4.
    - Check the SDM for values that aren't defined.
6. Create an if statement for values that are disabled by KVM. These values are `5, 6, 11, 17, 66, 69` and assign the value `0x00000000` on all four registers and return false.
    - The values are found in the vmx.h file inside of `arch/x86/include/uapi/asm/vmx.h`
7. At the end of the function, return true because ecx would be a valid value for both the SDM and KVM.
8. Inside of the case `0x4ffffffd` and `0x4ffffffc`, assign eax, ebx, and edx the value of `0x00000000` and create an if statement that calls the function created earlier in step 1.
9. Depending on where the if statement is, if the statement passes, it should assign the appropriate values and break from the switch statement.
    - The if statement for `0x4ffffffd` should assign eax the current value of the exits for the requested exit type when the user types in `cpuid -l 0x4ffffffd -s exit_number`
    - The if statement for `0x4ffffffc` should assign ebx the current value of the high bits for the CPU cycles of the requested exit number and assign ecx the current value of the low bits for the CPU cycles of the requested exit number
10. Run the same build commands from assignment 2 starting at step 20.
11. Test using `cpuid -l 0x4ffffffd -s exit_number` and `cpuid -l 0x4ffffffc -s exit_number` 

### Question 3 - Frequency of exits and how many exits for a full VM boot
There are more exits performed during certain VM operations, for example, if I go to the root directory and do a ls -R, starting from 0x00154b6f exits, it turns into 0x001631b5 which is around 59,000 additional exits compared to something like exit 67 which is at 0 exits before and after doing the ls command from the root directory.
The number of exits for a full VM boot after the VMX module is reloaded is between 0x000f06aa and 0x000f06b2 (which is around 980,000+ exits) including login with a 7 character username, 12 character password, and typing in the cpuid command for 0x4fffffff.

### Question 4 - Which are the most frequent and least frequent exits
Leaving an Ubuntu server 18.04 VM open overnight for at least 8 hours
    -The most frequent exit is exit 30 which is the I/O instruction exit.
    - There were multiple least frequent exits, exits 2, 3, 4, 8, 9, 13, 14, 15, 16, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 33, 34, 36, 37, 38, 39, 41, 42, 43, 44, 45, 46, 48, 48, 50, 51, 53, 56, 57, 58, 59, 60, 61, 63, 64, 65, 67, and 68. All of these had 0 exits.
    - The least frequent exit with a value greater than 0 is exit 62 which is the page-modification log full exit with 1 exit. Another one with a small amount of exits is exit 55 with 3 exits which is the XSETBV exit. The next one had 9 exits which was the WBINVD or WBNOINVD exit.
