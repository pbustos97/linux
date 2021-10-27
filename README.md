# CMPE 283 Assignment 1

## Question 1
I did the assignment myself

## Question 2 - Steps taken in order to complete assignment
1. Fork the main linux repository on [Github](https://github.com/torvalds/linux)
2. Clone the repository you forked into a Linux machine or VM that has an Intel CPU
3. Go into the cloned repository
    - Directory should be called 'linux'
4. Make sure you have the Linux Kernel development dependencies installed
    - flex, bison, bc, cmake, openssl, libssl-dev, libelf-dev, dwarves, clang
    - use the package linux-headers-$(uname -r) to install linux headers if you don't have them inside of the /usr/src directory
