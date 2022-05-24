# First update Linux kernel on Ubuntu
sudo apt-get update

# Download and install "CMake"
Go to https://cmake.org/download/
Download stable release such as "cmake-3.23.1-linux-x86_64.tar.gz" for 18.05.2022
Create a folder in /opt named "CMake" with "sudo mkdir -m 755 /opt/CMake"
Extract CMake archive file to /opt/CMake with
   "sudo tar -xf ~/Downloads/cmake-3.23.1-linux-x86_64.tar.gz -C /opt/CMake/"
Remove downloaded archive file with "rm -r ~/Downloads/cmake-3.23.1-linux-x86_64.tar.gz"
Don't forget to add CMake binary to $PATH. We will do it with a bash script later.

# Download and install "Modelsim"
Go to INTEL website and find "Modelsim-Intel Lite" or "Questa-Intel Lite" setup and download. I downloaded "ModelsimSetup-19.1.0.670-linux.run" setup file. 
    INTEL states that: "Starting with Intel® Quartus® Prime version 21.3, the ModelSim*-Intel® FPGA edition software has been discontinued and replaced by the Questa*-Intel® FPGA Edition software. Refer to this customer advisory for details."
In the downloads page, I downloaded ID: 664524 and DATE: 03/31/19 and VERSION:19.1
This is the webpage as for now 18.05.2022:
https://www.intel.com/content/www/us/en/software-kit/664524/intel-quartus-prime-lite-edition-design-software-version-19-1-for-linux.html
Here in the "Individual Files" tab you can donwload Modelsim setup. It's size is just below 1 GB.
In the console run this script: "sudo mkdir -m 755 /opt/Modelsim"
Then run the .run file with "sudo ~/Downloads/ModelSimSetup-19.1.0.670-linux.run"
Select "Installation Directory" as "/opt/Modelsim" and finish the install.
Remove downloaded .run file with "rm -r ~/Downloads/ModelSimSetup-19.1.0.670-linux.run"
Don't forget to add "/opt/Modelsim/modelsim_ase/linuxaloem" to $PATH variable. We will do it with a bash script. 

# Download and install "ri5cy-toolchain"
There are 3 different PULP risc-v toolchain repos:
https://github.com/pulp-platform/ri5cy_gnu_toolchain
https://github.com/pulp-platform/riscv-gnu-toolchain
https://github.com/pulp-platform/pulp-riscv-gnu-toolchain

Before cloning and building one of these toolchains, some packages have to be installed with:
"sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev"

When I run "dpkg -l | grep gcc | awk '{print $2}'" console outputs:
gcc
gcc-10-base:amd64
gcc-10-base:i386
gcc-9
gcc-9-base:amd64
libgcc-9-dev:amd64
libgcc-s1:amd64
libuno-cppuhelpergcc3-3
libuno-purpenvhelpergcc3-3
libuno-salhelpergcc3-3

ri5cy_gnu_toolchain README.md says it works with gcc 5.2:
"This will download a specific version of the RISCV toolchain based on gcc 5.2, patch it with extensions for Pulpino and compile it."
gcc version can cause problems, I am not sure. 

When I run "gcc --version" console says:
"gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0"

In riscv-gnu-toolchain repo in the Installation section of README.md it says:
"./configure --prefix=/opt/riscv --with-arch=rv32imfcxpulpv3 --with-abi=ilp32 --enable-multilib
make"

Maybe I can try pulpv2 ??? !!!
But If we use zero-riscy -march=IM option should be selected ??? !!! 

I just cloned ri5cy_gnu_toolchain and run "make" as suggested in Kula's master thesis.
When running make, I saw that the script downloaded gcc-5.2.0, newlib-2.2.0, binutils-2.25
Sth went wrong on building toolchain. It seems that I need to run "make" with gcc-5.2. I found a way to download gcc-5.2:
https://askubuntu.com/questions/1235819/ubuntu-20-04-gcc-version-lower-than-gcc-7

   Add xenial to sources.list:
     deb http://dk.archive.ubuntu.com/ubuntu/ xenial main
     deb http://dk.archive.ubuntu.com/ubuntu/ xenial universe
   sudo apt update
   sudo apt install g++-5 gcc-5
   sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-5 5
   sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-5 5
   sudo update-alternatives --config gcc
   sudo update-alternatives --config g++

Then I rerun "make" in toolchain.
Seems OK after 30-40 min :)

Create a folder in /opt as "riscv" and copy the content of "ri5cy_gnu_toolchain/install/" folder to there:
"sudo mkdir -m 755 /opt/riscv"
"sudo cp -RT ~/Desktop/ri5cy_gnu_toolchain/install /opt/riscv"

Later we will add "/opt/riscv/bin" to PATH variable

# Other installations
"sudo apt-get install tcsh"
"sudo apt install python3-pip"
"sudo apt-get install python-yaml"

# Clone pulpino
"git clone https://github.com/pulp-platform/pulpino"
run "./update-ips.py"
I got error "usr/bin/env ‘python’ no such file or directory"
Answer is in: https://stackoverflow.com/questions/3655306/ubuntu-usr-bin-env-python-no-such-file-or-directory
   "sudo apt install python-is-python3"
But we need Python2 for this script to run.
Actually I have python2.7 but script automatically use python3. So run the script with:
   "python2 ./update-ips.py"
It seems OK but still some errors. I changed last line with: 
   "execute("python2 ./generate-scripts.py")" and now no error.
IMPORTANT: Continue reading this manual, there is an option to change default Pyhthon version, just read in patience!

Okay okay :) just write these scripts to console and select Pyhthon2

   sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
   sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
   sudo update-alternatives --config python

Now we have our IPs in "/pulpino/ips" folder.

# Create /build folder and script
Create build folder with: "mkdir -m 755 xxx/pulpino/sw/build"
Copy build script to /build folder with: "cp xxx/pulpino/sw/cmake_configure.zeroriscy.gcc.sh xxx/pulpino/sw/build/cmake_configure.zeroriscy.gcc.sh"
Then run "./cmake_configure.zeroriscy.gcc.sh" inside /build folder
This script creates necessary folders and files inside /build folder
Now pulpino is readily built to be used with Modelsim

# Compile pulpino for Modelsim
Compile pulpino project for Modelsim with: "make vcompile" inside /build folder
This gives error. Go to Modelsim setup and open vsim in /bin folder with sudo
   Change vco="linux_rh60" to vco="linux" in line 210
   Change mode=${MTI_VCO_MODE:-""} to mode=${MTI_VCO_MODE:-"32"} in line 13
Retry "make vcompile"
It seems that running Modelsim 19.1 in Ubuntu 20.04 is an issue !!!
I will try downloading a higher version like Questasim and retry !!!

I downloaded Questa-Intel FPGA Edition 21.1.0.842 and remove older Modelsim form /opt
"sudo rm -r /opt/Modelsim"
"sudo mkdir -m 755 modelsim"
This time I created modelsim folder with small 'm'
I tried to run vsim but thist time Questa gave license error:

Unable to find the license file.  It appears that your license file environment variable (e.g., LM_LICENSE_FILE) is not set correctly.
Unable to checkout a license.  Vsim is closing.
** Error: Invalid license environment. Application closing.

Go to Intel webpage and generate free Questa-Intel license by signing in.
I downloaded LR-082206_License.dat file. Copy the file to the /modelsim folder with: "sudo cp LR-082206_License.dat /opt/modelsim/license/LR-082206_License.dat"
Don't forget to add license file to PATH

None worked. But find a way !!! I downloaded 19.1 again and build it. This time I followed vhdlwhiz.com's steps for building modelsim to ubuntu 20.04 and it worked:
   https://vhdlwhiz.com/modelsim-quartus-prime-lite-ubuntu-20-04/#comment-300597

Now it is time to compilo pulpino with Modelsim.
Go to sw/build/ folder and run "make vcompile"

I got tons of errors. All about 64-bit Modelsim support in Ubuntu. I opened pulpino folder in VSCODE and search for -64 recursively, then deleted it in the scripts, be careful, DONT naively replace every "-64" found in search with space! There are numbers started with -64 in different files !!!

This compiles RTL found in /rtl, /ips and /tb folders.

After successfully compiling pulpino, console gives the output:
   "--> PULPino platform compilation complete!"

# Behavioral Simulation
There are example SW applications inside sw/apps folder. I will first try to compile "helloworld" SW application.
In order to compile this SW application, we need to call "make helloworld" inside sw/build folder. This will call:

   helloworld: cmake_check_build_system
      $(MAKE) $(MAKESILENT) -f CMakeFiles/Makefile2 helloworld
   .PHONY : helloworld

in Makefile2

   helloworld: apps/helloworld/CMakeFiles/helloworld.dir/rule
   .PHONY : helloworld

Again we got errors related to Python version. Let's make Python2 our default Pyhton version with:

   sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
   sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 2
   sudo update-alternatives --config python

Now retry "make helloworld", this time it will compile withour error. Executable files such as "helloworld.elf" is generated after compiling the SW.

Then we will run simulation on Modelsim with "make helloworld.vsim"
I got error:
   "env: ‘/opt/Modelsim/modelsim_ase/linuxaloem/vsim’: No such file or directory"
Somehow (maybe I build with older PATH variable) folder name is Uppercase "Modelsim" instead of "modelsim". I just changed with lowercase.
Retry "make helloworld.vsim"
Classical error for 64-bit Modelsim issues. Again search for -64 and remove them, be careful again not to replace all !!!
I am encountering "Modelsim" fplder path error and -64 again and again. I will restart steps for building pulpino from the beginning.

Now because of gcc version differences I got some error:
   "Error: (vsim-3828) Could not link 'vsim_auto_compile.so"

Google suggests removing all the folders starting with gcc. 
When I run inside /opt/modelsim/modelsim_ase/ 
   "ls -d gcc*"
   it gives:
   "gcc-4.3.3-linux  gcc-4.5.0-linux  gcc-4.7.4-linux"
So I run script below to remove these directories
   "rm -rf /opt/modelsim/modelsim_ase/gcc*/"

# Add necessary binaries to $PATH
NOTE: You can edit .bashrc or .profile files and add paths to $PATH variable in order not to run script each time you open a terminal. However, I will not do it. I will create a bash script and run this script each time when working on pulpino in a terminal.

source ~/Desktop/pulpino_help/pulpino_env_setup.sh

# STEP BY STEP EASY BUILD
1)  source source ~/Desktop/pulpino_help/pulpino_env_setup.sh
2)  git clone https://github.com/pulp-platform/pulpino
3)  cd pulpino
4)  ./update-ips.py
5)  mkdir -m 755 sw/build
6)  cp sw/cmake_configure.zeroriscy.gcc.sh sw/build/cmake_configure.zeroriscy.gcc.sh
7)  cd sw/build
8)  Delete -64 from CMakeSim.txt and other scripts
9)  ./cmake_configure.zeroriscy.gcc.sh
10) make vcompile
11) make helloworld
12) sudo rm -rf /opt/modelsim/modelsim_ase/gcc*/
13) make helloworld.vsim
14) Now you can run simulation in Modelsim