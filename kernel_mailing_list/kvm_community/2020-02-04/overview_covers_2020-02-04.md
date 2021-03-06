

#### [PATCH v3 0/3] selftests: KVM: AMD Nested SVM test infrastructure
##### From: Eric Auger <eric.auger@redhat.com>

```c
From patchwork Tue Feb  4 15:00:37 2020
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
X-Patchwork-Submitter: Eric Auger <eric.auger@redhat.com>
X-Patchwork-Id: 11364781
Return-Path: <SRS0=QQOC=3Y=vger.kernel.org=kvm-owner@kernel.org>
Received: from mail.kernel.org (pdx-korg-mail-1.web.codeaurora.org
 [172.30.200.123])
	by pdx-korg-patchwork-2.web.codeaurora.org (Postfix) with ESMTP id 528BC1395
	for <patchwork-kvm@patchwork.kernel.org>;
 Tue,  4 Feb 2020 15:01:00 +0000 (UTC)
Received: from vger.kernel.org (vger.kernel.org [209.132.180.67])
	by mail.kernel.org (Postfix) with ESMTP id 2FE512087E
	for <patchwork-kvm@patchwork.kernel.org>;
 Tue,  4 Feb 2020 15:01:00 +0000 (UTC)
Authentication-Results: mail.kernel.org;
	dkim=pass (1024-bit key) header.d=redhat.com header.i=@redhat.com
 header.b="ced4XvfP"
Received: (majordomo@vger.kernel.org) by vger.kernel.org via listexpand
        id S1727419AbgBDPA6 (ORCPT
        <rfc822;patchwork-kvm@patchwork.kernel.org>);
        Tue, 4 Feb 2020 10:00:58 -0500
Received: from us-smtp-delivery-1.mimecast.com ([207.211.31.120]:36823 "EHLO
        us-smtp-1.mimecast.com" rhost-flags-OK-OK-OK-FAIL) by vger.kernel.org
        with ESMTP id S1727305AbgBDPA6 (ORCPT <rfc822;kvm@vger.kernel.org>);
        Tue, 4 Feb 2020 10:00:58 -0500
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=redhat.com;
        s=mimecast20190719; t=1580828457;
        h=from:from:reply-to:subject:subject:date:date:message-id:message-id:
         to:to:cc:cc:mime-version:mime-version:
         content-transfer-encoding:content-transfer-encoding;
        bh=+Ut4XoP/Hrpe2MidinxPjS+a6EQ5ccTH+FceX9o7das=;
        b=ced4XvfPq0diGmWlhlqhu51wGohfbVCtprEqm7d/INkhq46L4JfeAclgLbb6j4ETJkvhaG
        Sk3VudHOT8IlecHoWpI7bdOW1Bh9T1ZNLdqb8VYbrESsYSZUjz57IT0FksS40dveKjIIWq
        q5H562LzC9oS/xNGNznJNeux4AzzO2s=
Received: from mimecast-mx01.redhat.com (mimecast-mx01.redhat.com
 [209.132.183.4]) (Using TLS) by relay.mimecast.com with ESMTP id
 us-mta-393-89EjOiL0NGml79LQGV0gQg-1; Tue, 04 Feb 2020 10:00:52 -0500
X-MC-Unique: 89EjOiL0NGml79LQGV0gQg-1
Received: from smtp.corp.redhat.com (int-mx01.intmail.prod.int.phx2.redhat.com
 [10.5.11.11])
        (using TLSv1.2 with cipher AECDH-AES256-SHA (256/256 bits))
        (No client certificate requested)
        by mimecast-mx01.redhat.com (Postfix) with ESMTPS id 658E518FE866;
        Tue,  4 Feb 2020 15:00:51 +0000 (UTC)
Received: from laptop.redhat.com (ovpn-116-37.ams2.redhat.com [10.36.116.37])
        by smtp.corp.redhat.com (Postfix) with ESMTP id D8F2584D90;
        Tue,  4 Feb 2020 15:00:44 +0000 (UTC)
From: Eric Auger <eric.auger@redhat.com>
To: eric.auger.pro@gmail.com, eric.auger@redhat.com,
        linux-kernel@vger.kernel.org, kvm@vger.kernel.org,
        pbonzini@redhat.com, vkuznets@redhat.com
Cc: thuth@redhat.com, drjones@redhat.com, wei.huang2@amd.com
Subject: [PATCH v3 0/3] selftests: KVM: AMD Nested SVM test infrastructure
Date: Tue,  4 Feb 2020 16:00:37 +0100
Message-Id: <20200204150040.2465-1-eric.auger@redhat.com>
MIME-Version: 1.0
X-Scanned-By: MIMEDefang 2.79 on 10.5.11.11
Sender: kvm-owner@vger.kernel.org
Precedence: bulk
List-ID: <kvm.vger.kernel.org>
X-Mailing-List: kvm@vger.kernel.org

Add the basic infrastructure needed to test AMD nested SVM.
Also add a first basic vmcall test.

Best regards

Eric

This series can be found at:
https://github.com/eauger/linux/tree/v5.5-amd-svm-v3

History:
v2 -> v3:
- Took into account Vitaly's comment:
  - added "selftests: KVM: Replace get_gdt/idt_base() by
    get_gdt/idt()"
  - svm.h now is a copy of arch/x86/include/asm/svm.h
  - avoid duplicates

v1 -> v2:
- split into 2 patches
- remove the infrastructure to run low-level sub-tests and only
  keep vmmcall's one.
- move struct regs into processor.h
- force vmcb_gpa into rax in run_guest()


Eric Auger (3):
  selftests: KVM: Replace get_gdt/idt_base() by get_gdt/idt()
  selftests: KVM: AMD Nested test infrastructure
  selftests: KVM: SVM: Add vmcall test

 tools/testing/selftests/kvm/Makefile          |   3 +-
 .../selftests/kvm/include/x86_64/processor.h  |  28 +-
 .../selftests/kvm/include/x86_64/svm.h        | 297 ++++++++++++++++++
 .../selftests/kvm/include/x86_64/svm_util.h   |  36 +++
 tools/testing/selftests/kvm/lib/x86_64/svm.c  | 159 ++++++++++
 tools/testing/selftests/kvm/lib/x86_64/vmx.c  |   6 +-
 .../selftests/kvm/x86_64/svm_vmcall_test.c    |  85 +++++
 7 files changed, 606 insertions(+), 8 deletions(-)
 create mode 100644 tools/testing/selftests/kvm/include/x86_64/svm.h
 create mode 100644 tools/testing/selftests/kvm/include/x86_64/svm_util.h
 create mode 100644 tools/testing/selftests/kvm/lib/x86_64/svm.c
 create mode 100644 tools/testing/selftests/kvm/x86_64/svm_vmcall_test.c
```
#### [RFC PATCH 0/7] vfio/pci: SR-IOV support
##### From: Alex Williamson <alex.williamson@redhat.com>

```c
From patchwork Tue Feb  4 23:05:34 2020
Content-Type: text/plain; charset="utf-8"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
X-Patchwork-Submitter: Alex Williamson <alex.williamson@redhat.com>
X-Patchwork-Id: 11365327
Return-Path: <SRS0=QQOC=3Y=vger.kernel.org=kvm-owner@kernel.org>
Received: from mail.kernel.org (pdx-korg-mail-1.web.codeaurora.org
 [172.30.200.123])
	by pdx-korg-patchwork-2.web.codeaurora.org (Postfix) with ESMTP id 750E8112B
	for <patchwork-kvm@patchwork.kernel.org>;
 Tue,  4 Feb 2020 23:05:47 +0000 (UTC)
Received: from vger.kernel.org (vger.kernel.org [209.132.180.67])
	by mail.kernel.org (Postfix) with ESMTP id 49CCC217F4
	for <patchwork-kvm@patchwork.kernel.org>;
 Tue,  4 Feb 2020 23:05:47 +0000 (UTC)
Authentication-Results: mail.kernel.org;
	dkim=pass (1024-bit key) header.d=redhat.com header.i=@redhat.com
 header.b="TZxM/3ha"
Received: (majordomo@vger.kernel.org) by vger.kernel.org via listexpand
        id S1727801AbgBDXFq (ORCPT
        <rfc822;patchwork-kvm@patchwork.kernel.org>);
        Tue, 4 Feb 2020 18:05:46 -0500
Received: from us-smtp-2.mimecast.com ([207.211.31.81]:42875 "EHLO
        us-smtp-delivery-1.mimecast.com" rhost-flags-OK-OK-OK-FAIL)
        by vger.kernel.org with ESMTP id S1727483AbgBDXFp (ORCPT
        <rfc822;kvm@vger.kernel.org>); Tue, 4 Feb 2020 18:05:45 -0500
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed; d=redhat.com;
        s=mimecast20190719; t=1580857543;
        h=from:from:reply-to:subject:subject:date:date:message-id:message-id:
         to:to:cc:cc:mime-version:mime-version:content-type:content-type:
         content-transfer-encoding:content-transfer-encoding;
        bh=50yJ/0uSQd8YRp4Nzpt7nFwJU44bA7UT36hqDKQZmJc=;
        b=TZxM/3haiTtCkhp6j1IFQXCq254ZD8+17Vsj0txbfa0Rse90I1MUHaVokzKc8qWbqX6tjJ
        2LSR9gYDHh7zRFCSP+1dnXykAvIzT+KJpGMy3UXRCKIsjjrByuIEl9dQ6SL3mL96Vt9Chq
        MhpByx/UZ4yQ+EoIzD9jk2ZZH+gSKa8=
Received: from mimecast-mx01.redhat.com (mimecast-mx01.redhat.com
 [209.132.183.4]) (Using TLS) by relay.mimecast.com with ESMTP id
 us-mta-63-_PGm0uoKPEO2dpkXo_Cobw-1; Tue, 04 Feb 2020 18:05:39 -0500
X-MC-Unique: _PGm0uoKPEO2dpkXo_Cobw-1
Received: from smtp.corp.redhat.com (int-mx07.intmail.prod.int.phx2.redhat.com
 [10.5.11.22])
        (using TLSv1.2 with cipher AECDH-AES256-SHA (256/256 bits))
        (No client certificate requested)
        by mimecast-mx01.redhat.com (Postfix) with ESMTPS id E1C5F1085926;
        Tue,  4 Feb 2020 23:05:37 +0000 (UTC)
Received: from gimli.home (ovpn-116-28.phx2.redhat.com [10.3.116.28])
        by smtp.corp.redhat.com (Postfix) with ESMTP id 88DEA1084194;
        Tue,  4 Feb 2020 23:05:34 +0000 (UTC)
Subject: [RFC PATCH 0/7] vfio/pci: SR-IOV support
From: Alex Williamson <alex.williamson@redhat.com>
To: kvm@vger.kernel.org
Cc: linux-pci@vger.kernel.org, linux-kernel@vger.kernel.org,
        dev@dpdk.org, mtosatti@redhat.com, thomas@monjalon.net,
        bluca@debian.org, jerinjacobk@gmail.com,
        bruce.richardson@intel.com, cohuck@redhat.com
Date: Tue, 04 Feb 2020 16:05:34 -0700
Message-ID: <158085337582.9445.17682266437583505502.stgit@gimli.home>
User-Agent: StGit/0.19-dirty
MIME-Version: 1.0
X-Scanned-By: MIMEDefang 2.84 on 10.5.11.22
Sender: kvm-owner@vger.kernel.org
Precedence: bulk
List-ID: <kvm.vger.kernel.org>
X-Mailing-List: kvm@vger.kernel.org

There seems to be an ongoing desire to use userspace, vfio-based
drivers for both SR-IOV PF and VF devices.  The fundamental issue
with this concept is that the VF is not fully independent of the PF
driver.  Minimally the PF driver might be able to deny service to the
VF, VF data paths might be dependent on the state of the PF device,
or the PF my have some degree of ability to inspect or manipulate the
VF data.  It therefore would seem irresponsible to unleash VFs onto
the system, managed by a user owned PF.

We address this in a few ways in this series.  First, we can use a bus
notifier and the driver_override facility to make sure VFs are bound
to the vfio-pci driver by default.  This should eliminate the chance
that a VF is accidentally bound and used by host drivers.  We don't
however remove the ability for a host admin to change this override.

The next issue we need to address is how we let userspace drivers
opt-in to this participation with the PF driver.  We do not want an
admin to be able to unwittingly assign one of these VFs to a tenant
that isn't working in collaboration with the PF driver.  We could use
IOMMU grouping, but this seems to push too far towards tightly coupled
PF and VF drivers.  This series introduces a "VF token", implemented
as a UUID, as a shared secret between PF and VF drivers.  The token
needs to be set by the PF driver and used as part of the device
matching by the VF driver.  Provisions in the code also account for
restarting the PF driver with active VF drivers, requiring the PF to
use the current token to re-gain access to the PF.

The above solutions introduce a bit of a modification to the VFIO ABI
and an additional ABI extension.  The modification is that the
VFIO_GROUP_GET_DEVICE_FD ioctl is specified to require a char string
from the user providing the device name.  For this solution, we extend
the syntax to allow the device name followed by key/value pairs.  In
this case we add "vf_token=3e7e882e-1daf-417f-ad8d-882eea5ee337", for
example.  These options are expected to be space separated.  Matching
these key/value pairs is entirely left to the vfio bus driver (ex.
vfio-pci) and the internal ops structure is extended to allow this
optional support.  This extension should be fully backwards compatible
to existing userspace, such code will simply fail to open these newly
exposed devices, as intended.

I've been debating whether instead of the above we should allow the
user to get the device fd as normal, but restrict the interfaces until
the user authenticates, but I'm afraid this would be a less backwards
compatible solution.  It would be just as unclear to the user why a
device read/write/mmap/ioctl failed as it might be to why getting the
device fd could fail.  However in the latter case, I believe we do a
better job of restricting how far userspace code might go before they
ultimately fail.  I'd welcome discussion in the space, and or course
the extension of the GET_DEVICE_FD string.

Finally, the user needs to be able to set a VF token.  I add a
VFIO_DEVICE_FEATURE ioctl for this that's meant to be reusable for
getting, setting, and probing arbitrary features of a device.

I'll reply to this cover letter with a very basic example of a QEMU
update to support this interface, though I haven't found a device yet
that behaves well with the PF running in one VM with the VF in
another, or really even just a PF running in a VM with SR-IOV enabled.
I know these devices exist though, and I suspect QEMU will not be the
primary user of this support for now, but this behavior reaffirms my
concerns to prevent mis-use.

Please comment.  In particular, does this approach meet the DPDK needs
for userspace PF and VF drivers, with the hopefully minor hurdle of
sharing a token between drivers.  The token is of course left to
userspace how to manage, and might be static (and not very secret) for
a given set of drivers.  Thanks,

Alex
---

Alex Williamson (7):
      vfio: Include optional device match in vfio_device_ops callbacks
      vfio/pci: Implement match ops
      vfio/pci: Introduce VF token
      vfio: Introduce VFIO_DEVICE_FEATURE ioctl and first user
      vfio/pci: Add sriov_configure support
      vfio/pci: Remove dev_fmt definition
      vfio/pci: Cleanup .probe() exit paths


 drivers/vfio/pci/vfio_pci.c         |  315 ++++++++++++++++++++++++++++++++---
 drivers/vfio/pci/vfio_pci_private.h |   10 +
 drivers/vfio/vfio.c                 |   19 ++
 include/linux/vfio.h                |    3 
 include/uapi/linux/vfio.h           |   37 ++++
 5 files changed, 356 insertions(+), 28 deletions(-)
```
