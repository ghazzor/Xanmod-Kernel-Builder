### **Xanmod-Kernel-Builder**
An extremely simple basic workflow to build xanmod kernel for your device
This builds a xanmod kernel with the multiple versions **Clang/LLVM** **`LLVM=1`** and **`LLVM_IAS=1`** for debian/ubuntu distros , can be easily edited to support arch linux


It uses this script to setup LLVM 
https://apt.llvm.org/llvm.sh

It builds with Clang/LLVM 19 , its the latest as the time of writing


The config in repo build a kernel specifically for my laptop , and this workflow has been made for non-generic kernel builds (hardware-specfic)
but it can be easily modified to make generic builds too , just specify your config in the workflow

#### Notice

1. Github Actions service is NOT unlimited so to avoid waste , use a config that works and is stable (build locally first and then edit the config
file in repo)

2. Before you make any changes, make sure that the repository you are operating in belongs to you. "Fork" if you want to commit code, otherwise use
"Use this template"

 shamelessly kanged instructions from **[here](https://github.com/azwhikaru/Action-TWRP-Builder)**


#### **To easily create a device-specfic config here is a noob-friendly guide**

1. Install the xanmod kernel you want build , lets say you want to build kernel 6.1 lts , so you would install the official **xanmod 6.1** build , boot into it

2. Clone the xanmod source


```shell
git clone --depth=1 https://gitlab.com/xanmod/linux -b 6.1 kernel/xanmod  
```

 the build puts the binaries in the kernel dir while the tree is in xanmod dir

3. Install Build Dependencies

 For Ubuntu 22.04

```shell
sudo apt update
sudo apt upgrade
sudo apt install gcc clang llvm lld g++ build-essential bison flex pkg-config qtcreator qtbase5-dev qt5-qmake qttools5-dev-tools libssl-dev libncurses-dev libelf-dev elfutils -y 
```
4. If you less than 16 gb ram then add some disk swap or zram , you can google it
   i have 12 gb of DDR4 , so i added some zram , which made the build process slightly faster 
5. We are going to use `make localmodconfig` to make a stripped down config, It Create a config based on current config and loaded modules (lsmod). Disables any module option that is not needed for the loadedmodules. Make sure to connect all the usb devices and turn on bluetooth and be connected to internet then cd to `kernel/xanmod` and run

```shell
make distclean
rm -rf vmlinux-gdb.py
make localmodconfig
```
 then 
```shell
make CC=clang LD=ld.lld LLVM=1 LLVM_IAS=1 xconfig

or

make CC=clang LD=ld.lld LLVM=1 LLVM_IAS=1 menuconfig
``` 

 both `xconfig` and `menuconfig` have gui , which can be used to edit the config genrated by `make localmodconfig` , you can enable some stuff here like **lto** , **kvm support** , weather to optmize for **size** or **performance** , **cpu specfic optmizations** etc and other stuff that you are going to use

 **NEVER** enable **`CONFIG_MNATIVE_AMD`** or **`CONFIG_MNATIVE_INTEL`** because it optmizes for the processor the kernel was built on , rather specify your processor specfically because you are going to build on a **workflow** , if you are going to build locally and use the kernel on the same machine then enable them

 lets say you have a **intel skylake** or **kabylake processor** 
 you can set `CONFIG_MSKYLAKE=y`
 you can use xconfig and menuconfig for that as they allow searching these configs and automatically adapt the config , now save your config and lets start the build

```shell
make CC=clang LD=ld.lld LLVM=1 LLVM_IAS=1 LOCALVERSION=-xanmod-clang deb-pkg -j$(nproc)
```
 the **`deb-pkg`** neatly packs the kernel into .deb files which are very easy to install

 you will find the .deb files in the `kernel` dir just beside the `xanmod` dir
 to install them run

```shell
sudo dpkg -i *.deb
```

 boot into the kernel and check it with `uname -r`
 it should be something like `xxx-xanmod-clang-xxx`

 test some stuff and if all goes well then edit the `config` file in repo and paste content from your .config which you will find in the `xanmod` dir

#### Tips

 Make sure to enable **Full LTO** in your configs and if you have 4 gb ram or less then cosider enabling **Thin LTO**

 You dont need to enable LTO in your local configs when you build locally , cuz it increases the build time rather just build without LTO but enable **Full LTO** in your config when you commit the .config to the repo , so that you get all the benifits of LTO