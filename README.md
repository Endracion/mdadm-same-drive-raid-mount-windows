# mdadm-raid-mount-windows

Preface: Why even write this guide?

Simple really, I wanted to extract more performance out of the dual actuator hard drive I got, but unlike its SAS counterpart, the Exos Mach 2 SATA does not expose 2 LUNs to the system, it appears as a single drive; bummer.
In reality, you can split the drive in half and get the same performance out of each side, in parellel, which means raiding them should increase performance. But raid as it normally stands requires disks, not partitions, so I had to look into alternative solutions, and finally stumbled upon and successfully tested mdadm coupled with WinMD: https://github.com/maharmstone/winmd

Here's are the before and after Cystal Disk Mark tests:

![DiskMark64_2023-11-18_13-03-11](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/e4adbec5-0d9f-4b97-ad23-52cf8f011446)

![DiskMark32_2023-11-22_00-04-41](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/1506a2ca-18f6-4481-8d35-a41aea967f05)

Setup:

Get yourself a linux machine or a live USB distrubution; I used Kali KDE through Ventoy with the persistence plugin, so I could keep what I install on it.

Once launched, 
