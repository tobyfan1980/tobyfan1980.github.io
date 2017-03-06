---
layout: post
title:  "Understanding Libvirt CPU model"
date:   2017-03-06 11:00:00 -0800
categories:    Virtualization
tags:    Libvirt, Linux
---

In a linux system, you can find a file at /usr/share/libvirt/cpu_map.xml.  In the first section, we can find 

```
<cpus>
  <arch name='x86'>
    <!-- vendor definitions -->
    <vendor name='Intel' string='GenuineIntel'/>
    <vendor name='AMD' string='AuthenticAMD'/>

    <!-- standard features, EDX -->
    <feature name='fpu'> <!-- CPUID_FP87 -->
      <cpuid function='0x00000001' edx='0x00000001'/>
    </feature>
    <feature name='vme'> <!-- CPUID_VME -->
      <cpuid function='0x00000001' edx='0x00000002'/>
    </feature>
    <feature name='de'> <!-- CPUID_DE -->
      <cpuid function='0x00000001' edx='0x00000004'/>
    </feature>
    <feature name='pse'> <!-- CPUID_PSE -->
      <cpuid function='0x00000001' edx='0x00000008'/>
    </feature>
    <feature name='tsc'> <!-- CPUID_TSC -->
      <cpuid function='0x00000001' edx='0x00000010'/>
    </feature>
    <feature name='msr'> <!-- CPUID_MSR -->
      <cpuid function='0x00000001' edx='0x00000020'/>
    </feature>
    <feature name='pae'> <!-- CPUID_PAE -->
      <cpuid function='0x00000001' edx='0x00000040'/>
    </feature>
    ..........
```

This list shows the output of linux kernel function cpuid(), which get all features of the host's CPU and corresponding bits in register 'edx' will be set. 
Then it reads the register, get all features, and put them into the file at /proc/cpuinfo, in the flags field.

```
rocessor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model           : 37
model name      : Intel(R) Xeon(R) CPU           X5650  @ 2.67GHz
stepping        : 1
microcode       : 0x710
cpu MHz         : 2600.000
cache size      : 20480 KB
physical id     : 0
siblings        : 1
core id         : 0
cpu cores       : 1
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 11
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc aperfmperf pni pclmulqdq ds_cpl vmx ssse3 cx16 sse4_1 sse4_2 popcnt aes lahf_lm ida arat epb pln pts dtherm tpr_shadow vnmi ept vpid
bogomips        : 5200.00
clflush size    : 64
cache_alignment : 64
address sizes   : 40 bits physical, 48 bits virtual
power management:
```

The next a few sections in the cpu_map.xml describe CPU models of different vendors and the corresponding features they have.

```
    <!-- Intel CPU models -->
    <model name='Conroe'>
      <model name='pentiumpro'/>
      <vendor name='Intel'/>
      <feature name='mtrr'/>
      <feature name='mca'/>
      <feature name='pse36'/>
      <feature name='clflush'/>
      <feature name='pni'/>
      <feature name='ssse3'/>
      <feature name='syscall'/>
      <feature name='nx'/>
      <feature name='lm'/>
      <feature name='lahf_lm'/>
    </model>

    <model name='Penryn'>
      <model name='Conroe'/>
      <feature name='cx16'/>
      <feature name='sse4.1'/>
    </model>

    <model name='Nehalem'>
      <model name='Penryn'/>
      <feature name='sse4.2'/>
      <feature name='popcnt'/>
    </model>

    <model name='Westmere'>
      <model name='Nehalem'/>
      <feature name='aes'/>
    </model>
    ...
```

Most of CPU models inherit from another one, and it has all features of its parent model. For example, Westmere inherit from Nehalem, so it has feature 'popcnt'.

Now let take a look at the CPU model of the local host. Run "virsh capabilities", and we can see the output below (only display the first part).

```
<capabilities>

  <host>
    <uuid>7a1e4d56-1b50-2bc0-d063-4cfc3f7ca992</uuid>
    <cpu>
      <arch>x86_64</arch>
      <model>Westmere</model>
      <vendor>Intel</vendor>
      <topology sockets='4' cores='1' threads='1'/>
      <feature name='rdtscp'/>
      <feature name='vmx'/>
      <feature name='ds_cpl'/>
      <feature name='pclmuldq'/>
      <feature name='ss'/>
      <feature name='acpi'/>
      <feature name='ds'/>
      <feature name='vme'/>
    </cpu>
```

This tells us the host cpu is Westmere and it contains all the features of Westmere plus the features in the list.  

When we start a virtual machine, we can specify its cpu model in its libvirt domain. For example,

```
<cpu match='exact'>
  <model fallback='forbid'>Westmere</model>
  <vendor>Intel</vendor>
  <topology sockets='1' cores='2' threads='1'/>
  <feature policy='require' name='acpi'/>
</cpu>
```

This means, the VM CPU should have features defined in Westmere plus 'acpi'. If the host CPU is not a superset of it, it will fail to start this VM.
This is very useful in VM migration, because we want to make sure the VM can work as needed in the target host it migrates to.
