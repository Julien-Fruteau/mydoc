---
title: protocols
tags:
  - network
  - sftp
---

## sftp

### chroot jail

- restrict user/group to specific directory

#### how to

- update `sshd_config`

```text
Match group sftp
  ChrootDirectory /home/sftp
  X11Forwarding no
  AllowTcpForwarding no
  ForceCommand internal-sftp
```

- filesystem ownership :
  - /home/sftp root:root, chmod 755
  - /home/sftp/\* root:sftp, chmod 755 for readonly access
