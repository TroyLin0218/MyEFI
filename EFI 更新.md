## OpenCore 更新
V2.0 2021.10.24更新 `OpenCore 0.7.4`

V1.0 2021.05.15 更新 `OpenCore 0.6.3`

### 1. EFI分区挂载
1. 创建挂载点

```shell
tianlin@iMac ~ % sudo mkdir /Volumes/EFI
```

2. 查看分区列表(EFI分区一般大于300MB)

```she
tianlin@iMac ~ % diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *250.1 GB   disk0
   1:                        EFI ⁨EFI⁩                     209.7 MB   disk0s1
   2:                 Apple_APFS ⁨Container disk2⁩         249.8 GB   disk0s2

/dev/disk1 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *240.1 GB   disk1
   1:       Microsoft Basic Data ⁨⁩                        239.4 GB   disk1s1
   2:                        EFI ⁨⁩                        607.7 MB   disk1s2

/dev/disk2 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +249.8 GB   disk2
                                 Physical Store disk0s2
   1:                APFS Volume ⁨TianLinMacOS - 数据⁩     56.3 GB    disk2s1
   2:                APFS Volume ⁨Preboot⁩                 363.2 MB   disk2s2
   3:                APFS Volume ⁨Recovery⁩                613.7 MB   disk2s3
   4:                APFS Volume ⁨VM⁩                      1.1 MB     disk2s4
   5:                APFS Volume ⁨TianLinMacOS⁩            15.1 GB    disk2s5
   6:              APFS Snapshot ⁨com.apple.os.update-...⁩ 15.1 GB    disk2s5s1

/dev/disk3 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *1.0 TB     disk3
   1:       Microsoft Basic Data ⁨新加卷⁩                  495.8 GB   disk3s1
   2:       Microsoft Basic Data ⁨新加卷⁩                  504.4 GB   disk3s2
```

3. 把目标EFI分区disk1s2挂载到刚刚创建的挂载点/Volumes/EFI上

```shell
tianlin@iMac ~ % sudo mount -t msdos /dev/disk1s2 /Volumes/EFI
Executing: /usr/bin/kmutil load -p /System/Library/Extensions/msdosfs.kext
```

4. 打开访达即可看到已经把EFI挂载上了

### 2. 更新引导文件

#### 1. 下载OpenCorePkg准备基础文件

   OC引导主要文件，解压后复制出`X64\EFI`文件夹到任意目录(如桌面)作为基础文件。下载地址[Releases · acidanthera/OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/releases)   

#### 2. 精简EFI目录(注意根据原有引导配置来操作删除)

   EFI|\OC\Tools目录保留OpenShell.efi 和ResetSystem.efi，其他文件删除。

   EFI\OC\Drivers目录保留OpenRuntime.efi 和OpenCanopy.efi，其他文件删除。

   EFI\OC\Resources目录删除。

#### 3. 复制config.plist文件

   复制OpenCorePkg解压后的Docs\Sample.plist文件到刚刚准备的基础文件的EFI\OC\目录下，~~重命名为config.plist~~。

> 2021.10.24
>
> 为了在使用vscode比较时方便区分新旧版本的config.plist文件，不建议修改名称

#### 4. 下载OcBinaryData

   OcBinaryData主要用来更新EFI\OC\Resources文件夹内容和HfsPlus.efi

   ~~HfsPlus.efi文件放到EFI\OC\Drivers下~~

> 2021.10.24更新
>
> HfsPlus.efi同2中的OpenRuntime.efi 和OpenCanopy.efi放一起。即：EFI\OC\Drivers目录下

   ~~Resources文件夹直接放到EFI\OC\下即可。也可以不更新，在2中不删除Resources即可~~

> 2021.10.24更新
>
> 可以不用更新Resources文件夹

   下载地址[acidanthera/OcBinaryData](https://github.com/acidanthera/OcBinaryData)

   注意：根据Github的最后更新时间来确定是否需要更新

> 2021.11.24修改：
>
> HfsPlus.efi需要根据原repo文件的修改时间来判断是否需要修改
>
> 本次更新OC(0.7.4)未更新HfsPlus.efi文件



#### 5. 更新必要Kext

   注意：每次更新时使用最新Release来更新

   AppleALC.kext [Releases · acidanthera/AppleALC (github.com)](https://github.com/acidanthera/AppleALC/releases)

   IntelMausi.kext [Releases · acidanthera/IntelMausi (github.com)](https://github.com/acidanthera/IntelMausi/releases)

   Lilu.kext [Releases · acidanthera/Lilu (github.com)](https://github.com/acidanthera/Lilu/releases)

   SATA-unsupported.kext [khronokernel/Legacy-Kexts](https://github.com/khronokernel/Legacy-Kexts/tree/master/Injectors/Zip) 

> 2021.10.24修改：
>
> SATA-unsupported.kext需要根据原repo文件的修改时间来判断是否需要修改
>
> 2021.10.24本次更新OC未更新SATA-unsupported.kext文件

   VirtualSMC.kext [Releases · acidanthera/VirtualSMC (github.com)](https://github.com/acidanthera/VirtualSMC/releases)

> 包含了多个kext，仅仅需要以下几个即可 
>    	｜--Kexts
>    		｜--SMCProcessor.kext
>    		｜--SMCSuperIO.kext
>    		｜--VirtualSMC.kext

   WhateverGreen.kext [Releases · acidanthera/WhateverGreen (github.com)](https://github.com/acidanthera/WhateverGreen/releases)

   USBPorts.kext (暂未找到下载地址，复制旧版继续使用)

   以上Kext均放到`EFI\OC\Kexts`目录下



#### 6. 沿用旧版的ACPI文件

   将旧版EFI\OC\ACPI目录下的aml文件复制到新版相同目录即可

   主要涉及以下3个aml

   ​	SSDT-EC.aml
   ​	SSDT-PLUG.aml
   ​	SSDT-USBX.aml

 #### 7.  至此，前期准备工作完成。

### 3. 更新config.plist

~~用VSCode打开旧版config.plist和新版config.plist，使用对比功能对比新旧plist文件，根据对比结果修改变化的部分。~~

> 2021.10.24更新
>
> 复制一份旧版的config.plist文件到任意目录，然后使用vscode同时打开新版的sample.plist和刚刚复制出来的config.plist文件，打开后按住Ctrl选中两个plist文件，右键选择`将已选项进行比较`来比较新旧版本的plist文件

提示：PanicNoKextDump和PowerTimeoutKernelPanic配置项默认是false，修改为true，即引导时**不打印**引导信息到引导界面，可以在修改完plist测试U盘引导遇到问题时设为false来查看报错信息。

plist文件解析参考：

> [OpenCore 简体中文参考手册 | OpenCore 简体中文参考手册(非官方) (skk.moe)](https://oc.skk.moe/)
>
> [精解OpenCore | 黑果小兵的部落阁 (daliansky.net)](https://blog.daliansky.net/OpenCore-BootLoader.html)

### 4. U盘引导启动

修改完plist文件之后，挂载U盘旧版EFI，备份旧版U盘引导文件，删除U盘EFI\OC和EFI\BOOT目录，将修改好的OC和BOOT文件夹放到U盘的EFI目录下。

重启电脑，按F2进入BIOS，BootMenu中使用U盘引导启动尝试Mac，测试本次更新是否成功。

若成功引导需要测试项：

​	蓝牙、WI-FI、以太网、睡眠、USB口等。

### 5. 替换硬盘中的EFi

挂载硬盘EFI，备份之，删除硬盘EFI\OC和EFI\BOOT目录，将修改好的OC和BOOT文件夹放到U盘的EFI目录下，重启电脑，测试同U盘引导。



 针对OpenCore0.7.4的config.plist文件更新，更新日期20211024

ACPI 

1. Patch ->array->dict0新增

   ```xml
   <key>Base</key>
   <string></string>
   <key>BaseSkip</key>
   <integer>0</integer>
   ```

   Patch ->array->dict1新增

   ```xml
   <key>Base</key>
   <string>\_SB.PCI0.LPCB.HPET</string>
   <key>BaseSkip</key>
   <integer>0</integer>
   ```

2. Quirks dict 新增

   ```xml
   <key>SyncTableIds</key>
   <false/>
   ```

Kernel

1. Quirks dict 新增

   ```xml
   <key>ProvideCurrentCpuInfo</key>
   <false/>
   ```

2. Scheme dict 新增

   ```xml
   <key>CustomKernel</key>
   <false/>
   ```

Misc

1. Entries->dict新增

   ```xml
   <key>Flavour</key>
   <string>Auto</string>
   ```

2. Security->dict 新增

   ```xml
   <key>AllowToggleSip</key>
   <false/>
   ```

3. Tools->array->dict0新增

   ```xml
   <key>Flavour</key>
   <string>OpenShell:UEFIShell:Shell</string>
   ```

   Tools->array->dict1新增

   ```xml
   <key>Flavour</key>
   <string>MemTest</string>
   ```

   Tools->array->dict2新增

   ```xml
   <key>Flavour</key>
   <string>Auto</string>
   ```

   

NVRAM 

1. Add->dict 新增

   ```xml
   <key>4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102</key>
   <dict>
     <key>rtc-blacklist</key>
     <data></data>
   </dict>
   ```

   Add->dict->`<key>7C436110-AB2A-4BBB-A880-FE41995C9F82</key>`新增

   ```xml
   <key>ForceDisplayRotationInEFI</key>
   <integer>0</integer>
   ```

2. Delete->dict新增

   ```xml
   <key>4D1FDA02-38C7-4A6A-9CC6-4BCCA8B30102</key>
   <array>
     <string>rtc-blacklist</string>
   </array>
   ```
   Delete->dict-> `<key>7C436110-AB2A-4BBB-A880-FE41995C9F82</key>` ->array新增

   ```xml
   <key>7C436110-AB2A-4BBB-A880-FE41995C9F82</key>
   ```

   

UEFI

0. APFS->AppleInput 新增

   ```xml
   <key>GraphicsInputMirroring</key>
   <true/>
   ```

1. Drivers 修改

   ```xml
   <!--原有-->
   <key>Drivers</key>
   <array>
     <string>HfsPlus.efi</string>
     <string>OpenRuntime.efi</string>
     <string>OpenCanopy.efi</string>
   </array>
   <!--改为-->
   <key>Drivers</key>
   <array>
     <dict>
       <key>Arguments</key>
       <string></string>
       <key>Comment</key>
       <string>HFS+ Driver</string>
       <key>Enabled</key>
       <true/>
       <key>Path</key>
       <string>HfsPlus.efi</string>
     </dict>
     <dict>
       <key>Arguments</key>
       <string></string>
       <key>Comment</key>
       <string></string>
       <key>Enabled</key>
       <true/>
       <key>Path</key>
       <string>OpenRuntime.efi</string>
     </dict>
     <dict>
       <key>Arguments</key>
       <string></string>
       <key>Comment</key>
       <string></string>
       <key>Enabled</key>
       <false/>
       <key>Path</key>
       <string>OpenCanopy.efi</string>
     </dict>
   </array>
   ```

2. Output ->dict修改 

```xml
<!--原有-->
<key>GopPassThrough</key>
<false/>
<!--改为-->
<key>GopPassThrough</key>
<string>Disabled</string>
```

3. ProtocolOverrides->dict新增

```xml
<key>AppleEg2Info</key>
<false/>
```

4. Quirks->dict新增

   ```xml
   <key>ForceOcWriteFlash</key>
   <false/>
   ```

   
