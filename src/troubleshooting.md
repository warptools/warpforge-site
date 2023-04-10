---
title: "Troubleshooting"
layout: base.njk
eleventyNavigation: 
    key: "Troubleshooting"
    order: 90
---

Troubleshooting
===============

Here we have some collected info on difficulties folks have encountered while trying to run Warpforge.  Hopefully, you're too lucky to hit any of these yourself — but if you do, maybe these notes can help you fix them.

## General Debugging Tips

- Keep a log!!!!
- Look at all the Known Situations below.
- If you still have to keep looking for clues…
    - `dmesg` can be a source of information.
    - `lsmod` and `modprobe` can be sources of information.
    - `mount` can be a source of information.
    - Some files under `/proc` and `/sys` can be sources of information (but you'll have to figure out which ones are relevant, somehow — there's a lot of them).
- Come to one of the chats listed in the [Community](/community) page and ask us, and share your log!

## Meta: Adding to this page

Please include snippets of the log that the user would see if they're experiencing a problem.  Something to pattern-match on to figure out *which* problem is going on is more than half the battle!

## Overlayfs refuses to work on ZFS

Warpforge uses OverlayFS for `overlay` mounts. OverlayFS does not support ZFS.  This can result in some tricky situations.  Generally, the fix is to ask Warpforge to use some other directories on a non-ZFS filesystem for certain internal operations.

### How to spot a problem that's related to ZFS

TODO: show the logs a user sees immediately in the terminal.

The problem can be confirmed by inspecting `dmesg` output, which will include errors like this:

```
[130225.957891] overlayfs: upper fs does not support RENAME_WHITEOUT.
[130225.957895] overlayfs: upper fs does not support xattr, falling back to xino=off.
[130225.957896] overlayfs: upper fs missing required features.
```

### Fixing (or working around) the ZFS problem

Find someplace on your system that isn't ZFS, and set the `WARPFORGE_RUNPATH` variable to point to a directory there.

```
WARPFORGE_RUNPATH=/somewhere/not/on/zfs/ warpforge run
```

After this, we can run modules from anywhere on our system (even within ZFS). It's only the "upperdir" — an internal detail of overlayfs — that can't be on ZFS, and with this environment variable, we relocate where warpforge makes those.

If you have Root on ZFS (which Ubuntu now supports during install) then you have some additional hurdles to clear.  But, you can create an ext4 volume within ZFS, mount that, then run from there:

```
sudo zfs create -s -V 10G rpool/ext4playspace
sudo mkfs.ext4 -b 4096 -L warpforge /dev/zvol/rpool/ext4playspace
sudo mkdir /mnt/ext4playspace
sudo mount /dev/zvol/rpool/warpforge /mnt/ext4playspace/
sudo chown $USER:$USER /mnt/ext4playspace
export WARPFORGE_RUNPATH=/mnt/ext4playspace
```

This problem isn't unique to Warpforge. Similar issues are tracked in several other systems:

[](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1873917)

[Topicbox](https://zfsonlinux.topicbox.com/groups/zfs-discuss/Tf85a18bdab2d891a)

## Some kernels don't permit Overlayfs rootlessly

Warpforge tries to run "rootless" (i.e., no sudo needed) containers.  This is easier on some hosts than others.  Unfortunately, kernel configuraiton flags sometimes vary, and in some setups, things needed to have efficient rootless containers just don't work.

### How to spot a problem caused by lack of support for rootless overlayfs

You'll get an error in the logs saying something about…  "`error during container init`" (this just means *something* went wrong)…

… and then "`error mounting \"none\" to rootfs at \"/\"`" (this means either overlayfs or bind mounting has failed)…

… and then if you see "`lowerdir`" or "`upperdir`" anywhere in the message, it means we're hearing about overlayfs for sure…

… and if the error ends in "`no such device`": then almost certainly, you're here.

… otherwise, if the error ends in "`permission denied`": actually, we have no idea, and there might be more than one issue at play, but there's still a good chance you might still be here.

((TODO: ^ this is actually only a hypothesis!  More observations needed!!))

Here are some additional debugging steps you can use to try to inspect the situation:

- Check to see if your host reports that you have overlayfs enabled.  (Note: commands will likely need `sudo`.)
    - Use `lsmod | grep overlay` to see if overlayfs is enabled.
    - Use `modprobe overlay` to enable overlayfs if it is availabe.
- You might be able to guess that this is the problem if you have an old kernel: [overlayfs has questionable support](https://rootlesscontaine.rs/how-it-works/overlayfs/)  on non-Ubuntu systems prior to linux kernel 5.11.
    - (Why "non-Ubuntu"?  Ubuntu specified custom kernel flags in earlier systems, and has thereby supported this behavior for a longer time.)
- Please note that we have not tested overlayfs-fuse.  If you learn more about the behavior, and any errors of that system, please get in touch in [Community](/community) spaces and let us know what you find!

### Fixing (or working around) a lack of support for rootless overlayfs

- **Option 0:** if you're missing overlayfs features entirely… get them enabled!
    - Check the modprobe stuff, above.
    - Usually this isn't the problem on any even remotely recent system, though.
- **Option 1:** upgrade your kernel to 5.11 or newer!
    - The newest linux kernels support rootless overlayfs by default, so getting an updated kernel should address this issue!
- **Option 2:** enable this behavior with a kernel module option!
    - In some host environments, you *may* be able to enable rootless overlayfs mounting with a kernel module option. If present, the magic system control file named `/sys/module/overlay/parameters/permit_mounts_in_userns` may control this.  Reading that file will tell you whether or not this is enabled — "1" means enabled.  As root, you should also be able to `echo 1 > permit_mounts_in_userns` into that file to change the setting.
    - In some systems (Debian?) there may be a file which configures this at boot.  Below is a typical place to encounter such a file (but your system may vary) — and you'd want it to contain a "1" for the value shown:
        
        ```
        ~ $ cat /etc/modprobe.d/10-overlay.conf
        options overlay permit_mounts_in_userns=1
        ```
        
    - This is NOT CONFIRMED to work!  Forum discussions on the internet describe this as a potential solution, however we have not confirmed this.
- **Option 3:** …?
    - (We suspect there are other ways to configure this!  If you find some, let us know in [Community](/community) and we'll make sure to get that info up here!)

## Some kernels don't permit rootless userNS at all

Warpforge tries to run "rootless" (i.e., no sudo needed) containers.  An absolutely essential prerequisite for rootless containers is that the host permits something called "userns_clone" without elevated privileges.  Almost all systems support this, so if you're here… well, wow.

### How to spot a problem caused by lack of support for rootless overlayfs

We don't have an example of what the log you'll see from warpforge will look like if you encounter this, because we've never actually seen it.

Nonetheless, as a health check, you can inspect this magical system file, and it should say "1":

```jsx
cat /proc/sys/kernel/unprivileged_userns_clone
1
```

### Fixing (or working around) a lack of support for rootless overlayfs

Probably `sudo` and then `echo 1 >  /proc/sys/kernel/unprivileged_userns_clone` , if we had to guess.

## Terminal doesn't echo inputs after running test suite

Currently, running the warpforge test suite can result in the terminal being left in a state where it's been configured to not echo inputs.  This is a bug we haven't figured out the origins of yet.

(Possibly some kind of race condition in runc itself?  Unclear.  Investigation welcomed.)

### Fixing a broken/weird terminal

If you encounter this (or any issues with a tty being "weird"), `reset` is a command available in most environments that should re-normalize the tty's configuration and get you back to a happy place. Alternatively, `stty sane` should also work.

## Warpforge runpath must be writable

If `WARPFORGE_RUNPATH` is set, that path must be writable. If it is not set, then the default temporary files path must be writable (I.E. `/tmp`).

### How to spot this problem

This shows as `permission denied` errors.

```
# JSON Output
$ warpforge -v -json run -f 2>&1 >/dev/null | jq .
{
  "CodeString": "warpforge-error-plot-step-failed",
  "Message": "plot step \"hello-world\" failed: formula execution failed: io error: failed to create temp run directory: mkdir /mnt/warpforge/warpforge-run-3795592395: permission denied",
  "Details": [
    [
      "stepName",
      "hello-world"
    ]
  ],
  "Cause": {
    "CodeString": "warpforge-error-formula-execution-failed",
    "Message": "formula execution failed: io error: failed to create temp run directory: mkdir /mnt/warpforge/warpforge-run-3795592395: permission denied",
    "Details": null,
    "Cause": {
      "CodeString": "warpforge-error-io",
      "Message": "io error: failed to create temp run directory: mkdir /mnt/warpforge/warpforge-run-3795592395: permission denied",
      "Details": [
        [
          "context",
          "failed to create temp run directory"
        ],
        [
          "path",
          "none"
        ]
      ],
      "Cause": {
        "CodeString": "",
        "Message": "mkdir /mnt/warpforge/warpforge-run-3795592395: permission denied",
        "Details": null,
        "Cause": null
      }
    }
  }
}
```

```
# Normal Output
$ warpforge run -f 2>&1 >/dev/null
┌─ plot  
│  plot  inputs:
│  plot  	type = catalog
│  plot  		ref = catalog:min.warpforge.io/alpinelinux/rootfs:v3.15.4:amd64
│  plot  		wareId = tar:5tYLAQmLw9K5Q1puyxrkyKF4FAVNTGgZqWTPSZXC3oxrzqsKRKtDi3q17E7XwbmnkP
│  plot  		wareAddr = https://dl-cdn.alpinelinux.org/alpine/v3.15/releases/x86_64/alpine-minirootfs-3.15.4-x86_64.tar.gz
├─ plot  (hello-world) evaluating protoformula
│ ┌─ formula  
error: plot step "hello-world" failed: formula execution failed: io error: failed to create temp run directory: mkdir /mnt/warpforge/warpforge-run-3227176136: permission denied
```

You may also see complaints about cache problems.

```
executor engine failed:
    the runc engine reported error:
        cannot initialize cache dirs:
            mkdir /.warpforge.container/workspace/cache:
                permission denied
```

```json
{
  "result": {
    "error": {
      "category": "rio-local-cache-problem",
      "message": "cannot initialize cache dirs: mkdir /.warpforge.container/workspace/cache: permission denied"
    }
  }
}
```

### Fixing this problem

Make sure that `/tmp` or wherever temporary files are located is writable by the active user.

If using `WARPFORGE_RUNPATH` then executing `chown -r $USER:$USER $WARPFORGE_RUNPATH` should do the trick. If not, then you may need to look into how these directories are mounted and the filesystems in use.

## Wanna force yourself to have these issues?

Probably not; you're not a silly person.

But sometimes we do actually want to, for testing purposes.

### Intentionally breaking overlayfs

Try this:

```jsx
sudo
echo 'blacklist overlay' > '/etc/modprobe.d/disable-overlay.conf'
modprobe -r overlay
```

### Intentionally breaking cache dirs

Set the warpforge runpath to a directory where you do not have write permissions.

Assuming the current user does not have write access to directories owned by root.

```
export WARPFORGE_RUNPATH=$(mktemp -d)
chown root:root "$WARPFORGE_RUNPATH"
```
