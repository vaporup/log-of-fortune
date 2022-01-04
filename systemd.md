
# systemd v250

```
* systemd-nspawn's --bind=/--bind-ro= switches now optionally take
  uidmap/nouidmap options as last parameter. If "uidmap" is used the
  bind mounts are created with UID mapping taking place that ensures
  the host's file ownerships are mapped 1:1 to container file
  ownerships, even if user namespacing is used. This way
  files/directories bound into containers will no longer show up as
  owned by the nobody user as they typically did if no special care was
  taken to shift them manually.
  
* A new setting SaveIntervalSec= has been added to systemd-timesyncd,
  which may be used to automatically save the current system time to
  disk in regular intervals. This is useful to maintain a roughly
  monotonic clock even without RTC hardware and with some robustness
  against abnormal system shutdown.
  
* A new setting DefaultOOMScoreAdjust= is now supported in
  /etc/systemd/system.conf + /etc/systemd/user.conf that may be used to
  set the default process OOM score adjustment value for processes
  forked off the service manager. For per-user service managers this
  now defaults to 100, but for per-system service managers is left as
  is. This means that by default now services forked off the user
  service manager are more likely to be killed by the OOM killer than
  system services or the managers themselves.
  
* Support for encrypted and authenticated credentials has been added.
  This extends the credential logic introduced with v247 to support
  non-interactive symmetric encryption and authentication, based on a
  key that is stored on the /var/ file system or in the TPM2 chip (if
  available), or the combination of both (by default if a TPM2 chip
  exists the combination is used, otherwise the /var/ key only). The
  credentials are automatically decrypted at the moment a service is
  started, and are made accessible to the service itself in unencrypted
  form. A new tool 'systemd-creds' encrypts credentials for this
  purpose, and two new service file settings LoadCredentialEncrypted=
  and SetCredentialEncrypted= configure such credentials.

  This feature is useful to store sensitive material such as SSL
  certificates, passwords and similar securely at rest and only decrypt
  them when needed, and in a way that is tied to the local OS
  installation or hardware.
```

# systemd v249   

```
* systemd-nspawn gained a new switch --bind-user= for binding a host
  user account into the container. This does three things: the user's
  home directory is bind mounted from the host into the container,
  below the /run/userdb/home/ hierarchy. A free UID is picked in the
  container, and a user namespacing UID mapping to the host user's UID
  installed. And finally, a minimal JSON user and group record (along
  with its hashed password) is dropped into /run/host/userdb/. These
  records are picked up automatically by the userdb drop-in logic
  describe above, and allow the user to login with the same password as
  on the host. Effectively this means: if host and container run new
  enough systemd versions making a host user available to the container
  is trivially simple.
  
* systemd-nspawn's --private-user-chown switch has been replaced by a
  more generic --private-user-ownership= switch that accepts one of
  three values: "chown" is equivalent to the old --private-user-chown,
  and "off" is equivalent to the absence of the old switch. The value
  "map" uses the new UID mapping mounts of Linux 5.12 to map ownership
  of files and directories of the underlying image to the chosen UID
  range for the container. "auto" is equivalent to "map" if UID mapping
  mount are supported, otherwise it is equivalent to "chown". The short
  -U switch systemd-nspawn now implies --private-user-ownership=auto
  instead of the old --private-user-chown. Effectively this means: if
  the backing file system supports UID mapping mounts the feature is
  now used by default if -U is used. Generally, it's a good idea to use
  UID mapping mounts instead of recursive chown()ing, since it allows
  running containers off immutable images (since no modifications of
  the images need to take place), and share images between multiple
  instances. Moreover, the recursive chown()ing operation is slow and
  can be avoided. Conceptually it's also a good thing if transient UID
  range uses do not leak into persistent file ownership anymore. TLDR:
  finally, the last major drawback of user namespacing has been
  removed, and -U should always be used (unless you use btrfs, where
  UID mapped mounts do not exist; or your container actually needs
  privileges on the host).
  
* systemd-nspawn's --private-user= switch now accepts the special value
  "identity" which configures a user namespacing environment with an
  identity mapping of 65535 UIDs. This means the container UID 0 is
  mapped to the host UID 0, and the UID 1 to host UID 1. On first look
  this doesn't appear to be useful, however it does reduce the attack
  surface a bit, since the resulting container will possess process
  capabilities only within its namespace and not on the host.
  
* A new ConditionOSRelease= setting has been added to unit files to
  check os-release(5) fields. The "=", "!=", "<", "<=", ">=", ">"
  operators may be used to check if some field has some specific value
  or do an alphanumerical comparison. Equality comparisons are useful
  for fields like ID, but relative comparisons for fields like
  VERSION_ID or IMAGE_VERSION.
```
