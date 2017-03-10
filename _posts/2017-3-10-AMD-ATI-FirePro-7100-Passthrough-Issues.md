---
layout: post
title: AMD ATI FirePro 7100 Passthrough Issues
---
I have been working on pci-passthrough for over two months. I was testing it with a NVidia Quadro 2000 graphic card on a centos 7.3 system. I could do render jobs in the vm perfectly. The whole system works perfectly until last week. One customer need to passthrough a ATI FirePro 7100 graphic card into a windows 7 guest vm. Considering my previous work, I thought it could be pretty easy to do that. So I blacklist the amdgpu driver, add vfio option and turn on the iommu_intel in the grub commandline and then setup a windows 7 virtual machine. All things went well, I installed the graphic card's driver and happy to see it appeared in the device list. But when I restart from the windows 7 vm, problem occured. The screen was black and apparently the system couldn't boot up. I destroyed the vm and start it again but still got the same problem. After dozens of tests, I found out a rule that every first time I start the vm after the **host** has been reboot, the vm boot up successfully with graphic card passthroughed. After you restart the vm or turn  it off, it'll never work again. So I googled this issue and found it is a well known issue called " PCI bus reset issue". Here's lots of discussions on this issue and the best explaination comes from the code comments of [pci-quirks.c](https://github.com/qemu/qemu/blob/master/hw/vfio/pci-quirks.c#L1704):

```C++
/*
 * Reset quirks
 */

/*
 * AMD Radeon PCI config reset, based on Linux:
 *   drivers/gpu/drm/radeon/ci_smc.c:ci_is_smc_running()
 *   drivers/gpu/drm/radeon/radeon_device.c:radeon_pci_config_reset
 *   drivers/gpu/drm/radeon/ci_smc.c:ci_reset_smc()
 *   drivers/gpu/drm/radeon/ci_smc.c:ci_stop_smc_clock()
 * IDs: include/drm/drm_pciids.h
 * Registers: http://cgit.freedesktop.org/~agd5f/linux/commit/?id=4e2aa447f6f0
 *
 * Bonaire and Hawaii GPUs do not respond to a bus reset.  This is a bug in the
 * hardware that should be fixed on future ASICs.  The symptom of this is that
 * once the accerlated driver loads, Windows guests will bsod on subsequent
 * attmpts to load the driver, such as after VM reset or shutdown/restart.  To
 * work around this, we do an AMD specific PCI config reset, followed by an SMC
 * reset.  The PCI config reset only works if SMC firmware is running, so we
 * have a dependency on the state of the device as to whether this reset will
 * be effective.  There are still cases where we won't be able to kick the
 * device into working, but this greatly improves the usability overall.  The
 * config reset magic is relatively common on AMD GPUs, but the setup and SMC
 * poking is largely ASIC specific.
 */
```

It's a hw bug and apparently we could do little about it. Qemu gives this quirk as a workaround, but it's based on the device-id of your device and that means only limited number of devices get support currently.

```C++
void vfio_setup_resetfn_quirk(VFIOPCIDevice *vdev)
{
    switch (vdev->vendor_id) {
    case 0x1002:
        switch (vdev->device_id) {
        /* Bonaire */
        case 0x6649: /* Bonaire [FirePro W5100] */
        case 0x6650:
        case 0x6651:
        case 0x6658: /* Bonaire XTX [Radeon R7 260X] */
        case 0x665c: /* Bonaire XT [Radeon HD 7790/8770 / R9 260 OEM] */
        case 0x665d: /* Bonaire [Radeon R7 200 Series] */
        /* Hawaii */
        case 0x67A0: /* Hawaii XT GL [FirePro W9100] */
        case 0x67A1: /* Hawaii PRO GL [FirePro W8100] */
        case 0x67A2:
        case 0x67A8:
        case 0x67A9:
        case 0x67AA:
        case 0x67B0: /* Hawaii XT [Radeon R9 290X] */
        case 0x67B1: /* Hawaii PRO [Radeon R9 290] */
        case 0x67B8:
        case 0x67B9:
        case 0x67BA:
        case 0x67BE:
            vdev->resetfn = vfio_radeon_reset;
            trace_vfio_quirk_ati_bonaire_reset(vdev->vbasedev.name);
            break;
        }
        break;
    }
}
Contact GitHub API Training Shop Blog About
© 2017 GitHub, Inc. Terms Privacy Security Status Help
```

You could add your device's product id in the list and recompile it to get support from qemu and that's what I did. After I replace qemu-kvm with my new package, things did change, but only a very little better than before:

1. I can restart vm twice before I got the same problem.
2. After destroyed and recreated, the vm returned to the initial state which means you can restart it twice again :).

In a word, I could get the vm run bumpy without restart the host. That's not perfect but enough for my current senario. 