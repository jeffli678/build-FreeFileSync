# build-FreeFileSync
FreeFileSync is a great open source file synchronization tool. It is one of my favorite tools. However, despite its open source nature, there is no build instruction for it. This repo records my own way of building FreeFileSync 10.12 on Ubuntu 18.04. 

## 0. Download and extract the source code

As of writing, the latest version of FreeFileSync is 10.12 and it can be downloaded from: 

https://freefilesync.org/download/FreeFileSync_10.12_Source.zip. 

Note wget does not work with this URL. You can manually specify a version number to get the source code of an earlier version

You also need to get a Resources.zip from an earlier version of the source code, e.g., 10.11,  https://freefilesync.org/download/FreeFileSync_10.11_Source.zip. This is probably a bug that the developer forgets to include it. Without it, the program cannot find the icons needed for the UI. The Resouces.zip is in FreeFileSync_10.11_Source/FreeFileSync/Build. Simply copy and paste the zip to the same place in the 10.12 folder. 

## 1. Install a newer version of gcc

FreeFileSync requires a c++ compiler that supports c++2a. The default version of gcc on Ubuntu is 7.4.0 and does not work. I followed the instruction at: https://solarianprogrammer.com/2016/10/07/building-gcc-ubuntu-linux/ to build and install the gcc 9.1.0.

If you follow the steps correctly, you should be able to run "gcc-9.1 -v" without any error. 

## 2. Install wxWidgets

FreeFileSync 10.12 requires at least wxWidgets 3.1.1 to compile -- because I find it uses several functions newly added in the version 3.1.1. I choose to install 3.1.2, the latest as of writing. Building wxWidgets should be relatively easy:

```
wget https://github.com/wxWidgets/wxWidgets/releases/download/v3.1.2/wxWidgets-3.1.2.tar.bz2
tar xf wxWidgets-3.1.2.tar.bz2
cd wxWidgets-3.1.2/
mkdir gtk-build # or any other name you like
cd gtk-build
../configure --disable-shared --enable-unicode
make
make install # use sudo if necessary
```

## 3. Install Boost

FreeFileSync requries Boost to compile. However, it does not require a very recent version, so simply install the system package to get version 1.65.1:

```
sudo apt install libboost-all-dev
```

## 4. Modify the Makefile

The Makefile is at FreeFileSync_10.12_Source/FreeFileSync/Source/Makefile. We need to change all occurrances (there should be two) of "g++" to "g++-9.1" . In the original file, the C++ compiler is hardcoded to g++. However, we installed the gcc-9.1 in the first step and it is called g++-9.1. 

## 5. Tweak the code:

Now everything is almost ready. However, depending on your version of libssh2 and libcurl, you may encounter several errors. The best way to fix this is to install the latest version of these two libraries. However, I am not very familiar with them and do not want to change the system default version, so I fix the code when I get compilation errors. 

In "afs/libcurl/curl_wrap.h", comment the line 78 and 120. Basically, we wipe out 
```
ZEN_CHECK_CASE_FOR_CONSTANT(CURLE_OBSOLETE51); 
and ZEN_CHECK_CASE_FOR_CONSTANT(CURLE_RECURSIVE_API_CALL);. 
```

In "afs/sftp.cpp", add at line 58
```
#define MAX_SFTP_OUTGOING_SIZE 30000
#define MAX_SFTP_READ_SIZE 30000
```

In "afs/sftp.cpp", add at line 1662
```
#define LIBSSH2_SFTP_DEFAULT_MODE      -1
```

I found the above consts from header files in newer versions of libssh2 and libcurl. 

## 6. Compile:

Run "make" in folder FreeFileSync_10.12_Source/FreeFileSync/Source. It will take roughly 10 minutes to compile on a i7 machine. 

The binary should be waiting for you in FreeFileSync_10.12_Source/FreeFileSync/Build/Bin. 

Hopefully, you make it!
