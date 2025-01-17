# ShellWasp 2.0


ShellWasp is a new tool to faciliate creating shellcode utilizing syscalls, released at DEF CON 30 on August 12, 2022. It will also soon be a briefing at Black Hat Middle East and Africa this November, 2022! On April 19, 2023, ShellWasp 2.0 is released with massive updates, including alternative ways to get OSBuild (via User_Shared_Data, PEB via r12), and three new ways to invoke the syscall via WoW64 (one for Win7 and two for Win10/11).

ShellWasp automates building templates of syscall shellcode. The template is intended to be just that - a template. The user still must determine what parameter values to use and how to generate them. The intent is to simplify the process for those desiging to create syscall shellcode by hand. Nearly all user-mode syscalls supported, including all the ones I could find function prototypes for. ShellWasp also solves the syscall portability problem for syscalls. It identifies the OS build, and ShellWasp creates a syscall arrray in response to user input, allowing the current syscall values (SSNs) to be found at runtime, rather than having to be hardcoded, which can limit how you can use them across OS builds. ShellWasp takes care of managing the syscall array, so if a syscall is used multiple times, there will only be one entry in the syscall array. Thus, ShellWasp will allows syscall values (SSNs) to be obtained dynamically.

ShellWasp supports Windows 7/10/11. It uses both existing syscall tables, as well as newly created syscall tables for newer versions of Windows 10 & 11. These are created through an original process, parsing WinDbg results. To achieve a more compact shellcode size, ShellWasp utilizes precomputed syscall tables in JSON format. This allows us to keep the shellcode size minimal. 

The shellcode size created by ShellWasp is relatively small in size. Users can select the OS builds to support, and it is recommend to use perhaps just some of the most recent ones from Windows 7/10/11, rather than every possible one. This can help keep size more manageable. Additionall, the way in which syscalls are called differs from Windows 7 and Windows 10/11. ShellWasp will automatically take care of that based on the selections the user makes. We have created syscall shellcode that works across all three OS, using our technique.

ShellWasp 2.0 includes some  alternative ways to discover the OSBuild. ShellWasp 2.0 additionally provides three new ways to invoke the syscall from WoW64, all without syscall, int 0x2e or fs:0xc0 - two for Windows 10/11 and one for Windows 7. These two new methods have not been seen before (see below images).

## Using ShellWasp
The Assembly generated by ShellWasp is compact in size. Additionally, as many people have automatic Windows update, it may be desirable to select only more recent OS builds, rather than every possible one, and this helps reduce size as well. Users can easily and quickly rearrange syscalls in shellcode. 

ShellWasp only supports Windows 7/10/11 at the moment, as a desing choice. It is easy to select desired Windows releases via config file or UI. Changes can also be saved to the to config.


![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/shellwasp1.png?raw=true)

![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/shellwasp3.png?raw=true)

![image](https://user-images.githubusercontent.com/49998815/201258739-bc8e4f11-d737-4a1f-a8e5-7f827f701717.png)
Note: You select the OS builds to target--it is not necessary to target every single build--and you select the syscalls to use. The above is just a random illustration. ShellWasp takes care of a lot of the details, but you still need to build out the parameters and required structures.



![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/osbuild3.png?raw=true)

![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/fsyscall.png.png?raw=true)

![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/osbuild2.png?raw=true)
![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/multWays.png?raw=true)
![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/alt_invoke.png?raw=true)
![image](https://github.com/Bw3ll/ShellWasp/blob/main/images/altinvoke2.png?raw=true)





## Usage
Download via GitHub and run it on the command line, e.g. `py shellWasp.py`

Desired settings for selected OS builds/releases as well as Windows syscalls can be added to the config file. They also may be added or changed in the UI. 

## Updates
* On Nov. 1, 2022, support was added for Windows 10 22H2 and Windows 11 22H2. These are the newest Windows releases. Note: we do not support Insider preview builds nor Server. 
* On Nov. 1, 2022, the mechanism by which the pointer to the syscall array is preserved has been changed. In testing shellcode with chains of several Windows syscalls, some stability issues were noted with values on the stack. In order to avoid those issues, it was decided to change the stack cleanup (`add esp, 0xXX`) and `pop edi`,  to `mov edi, [esp+0xYY]` - YY being the number of bytes that would have been "cleaned" from the stack. The `push edi` that follows is retained. ShellWasp maintains a pointer to the syscall array at edi, and since the actual syscall itself destroys the value contained in edi, there needs to be a way to restore it, after the return from the far jump to kernel-mode. It was felt this new Assembly would be a more stable way to accomplish this. Of course, another option could be to have a pointer to the syscall array stored at some location on ebp or other memory, and then that could be used to restore EDI. That would in some ways be simpler, as it would be possible to avoiding needing to count the number of bytes to go back. However, it was felt that `mov edi, [esp+0xYY]` would be safer for novices. If it was stored elsewhere in memory at a fixed location, such as the stack, it could be possible to accidentally overwrite it. Both approaches take minimal time and effort. 
* April 19, 2023 - ShellWasp 2.0 is released with masssive changes, including alternative ways to identify to the OSBuild, and three previously undocumented ways to invoke the syscall via WoW64 (one for Windows 7 and two for Windows 10/11).

## Installation
A setup file is provided to help ensure the correct libraries are installed. 
To set up ShellWasp, simply use the following command: `py setup.py install`

You may substitute *py* with *python* as need be. This will make sure that Colorama and keystone-engine are installed. Keystone is used to compile the Assembly, which ensures it is valid Assembly. However, the generated bytes are not currently displayed, as ShellWasp is intended to produce a template, whose parameters need to be customized. This may be enabled as an optional setting in the future.

If you do not wish to use the setup.py file, you can also install these manually. `pip install keystone-engine` and `pip install colorama` may be used for that purpose.


## Correction
Please note that comments I made regarding sorting by address techniques no longer working were incorrect. I apologize for the error. Keep in mind this tool is geared for WoW64, 32-bit shellcode, not as a replacement for other syscall techniques. Our efforts remain in that WoW64 realm.

## License
This project is released under the terms of the MIT license.

## Note
Please note that this GitHub is currently incomplete, and it will be updated with more information at a later date, including install instructions.
