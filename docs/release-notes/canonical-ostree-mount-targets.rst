:Type: Storage
:Summary: Resolve OSTree and bootc mount targets inside the deployment (#2520626)

:Description:
    New OSTree and bootc installations resolve deployment symlinks when
    preparing mount targets and writing fstab. For example, a requested
    /home mount becomes /var/home when the deployment defines that link.
    Other payloads retain their existing fstab behavior. Symlink traversal
    above the deployment root and chains longer than 40 links are refused.

    This does not migrate existing installations. Administrators need a
    documentation/advisory note to check fstab against the deployment's
    canonical mount paths; this installer change cannot repair an already
    installed system.

    Partition types are unchanged. Anaconda enables Blivet's
    gpt_discoverable_partitions flag by default. Blivet's
    PartitionDevice.part_type_uuid_req uses gpt_part_uuid_for_mountpoint:
    /home maps to the DPS home GUID, whereas /var/home has no DPS mapping.
    This selection happens before deployment, so canonicalizing fstab
    does not retroactively change a partition's type.

    A canonical fstab entry alone is not a demonstrated fix for the GPT
    auto-generator failure. In systemd v259, the home partition path calls
    add_partition_mount for /home without checking fstab. path_is_busy
    returns false for an already mounted path, allowing generation during
    reloads. The fstab generator canonicalizes its target but does not
    create a home.mount alias for var-home.mount. A separate partition-type
    policy change (generic Linux data for redirected DPS mountpoints), or
    a generator fix, should be evaluated before claiming the rescue-mode
    regression is fully resolved. Neither is implemented here.

:Links:
    - https://bugzilla.redhat.com/show_bug.cgi?id=2520626
    - https://github.com/storaged-project/blivet/blob/master/blivet/devices/partition.py
    - https://github.com/storaged-project/blivet/blob/master/blivet/devicelibs/gpt.py
    - https://github.com/systemd/systemd/blob/v259/src/gpt-auto-generator/gpt-auto-generator.c
    - https://github.com/systemd/systemd/blob/v259/src/fstab-generator/fstab-generator.c
