---
title: 'The NAS That Came Back From the Dead'
description: 'How I recovered a Synology DS218j after nuking the OS without a full reinstall'
date: 2026-05-08
category: 'general'
toc: true
---

Yesterday, I did something that apparently several decades of experience and reading endless horror stories can still
not completely protect against: I typed `rm -rf /` with admin privileges on my primary data storage NAS. Fat fingered
the enter key while typing out the full path to the thing I _actually_ wanted to delete. Mental note for the future: put
`-rf` at the _end_ of the command, not the beginning...

I cancelled it in a split second, but the damage had been done. I could no longer log in via the web UI, SMB access was
broken, and a lot of other things were clearly suspect.

I have offsite backups of all the actual _data_ on there, so I wasn't especially worried about losing anything
irreplaceable, but I was filled with dread about spending the rest of my evening rebuilding the nas, it's configuration,
and having to go through a multi-day process to backup and restore terabytes of data. Since I could still access the
data and that wasn't affected, it would have made more sense to copy everything off it, versus restoring from my
offsite. That would probably take days to download; either option would mean work and this thing being out of service
for quite some time.

A year ago, I would absolutely have torn this thing down and done a fresh install. Today, this was a problem for Claude
Code. And Claude was able to solve it in about an hour in a half. Below is Claude's recollection of the incident, and
the timeline, in their words.

---

## Timeline

| Time      | Event                                                                                                                           |
| --------- | ------------------------------------------------------------------------------------------------------------------------------- |
| 17:10     | Accidental deletion sweeps through system directories. `/etc/ssl/certs` gutted to 13 entries.                                   |
| 17:11     | DSM package manager crashes — `smpkg_status_record` updated.                                                                    |
| 17:11     | smbd crashes (core dump). ASLR=2 incompatible with Samba 4.15.13 on ARM.                                                        |
| **17:15** | **ASLR fix applied** — `kernel.randomize_va_space=1` written to `/etc/sysctl.conf`. SMB restored.                               |
| **17:27** | **`syno-boot-workaround.service` created** — manually starts synocgid, synoscgi, and sets STEP=4 on boot. DSM web UI reachable. |
| 17:41     | DSM auto-backs up `/etc/pam.d/webui` during recovery work.                                                                      |
| **18:02** | **Version spoof** — `/etc/VERSION` and `/etc.defaults/VERSION` set to base 86009 to trick synoupgrade.                          |
| **18:08** | **`7zz` copied to `/usr/bin/`** for firmware extraction attempts (ultimately not needed).                                       |
| **18:13** | **Firmware update applied** — incremental Update 3 patch accepted. 282+ binaries restored.                                      |
| 18:16     | `/etc.defaults` updated by the patch. Version correctly restored to Update 3.                                                   |
| **18:26** | **Boot workaround disabled** — proper systemd units can now take over.                                                          |
| **18:33** | **Reboot.** DSM 7.3.2-86009 Update 3 comes back clean. Login works.                                                             |
| **18:40** | **SSL certs restored** from media NAS — 299 certs copied via tar pipe.                                                          |
| **18:45** | **Final audit and cleanup** — `webui.bak` deleted, smpkg counters zeroed.                                                       |

---

### SMB Rises From the Ashes

_~17:11_

The first problem to tackle was Samba. When smbd tried to start, it was panicking immediately with a cryptic
`reinit_after_fork()` error. A bit of digging revealed the culprit: Linux's Address Space Layout Randomisation (ASLR)
was set to level 2 — full randomisation — and Samba 4.15.13 on ARM simply can't cope with that. It needs level 1.

A one-liner fixed it:

```bash
echo 1 > /proc/sys/kernel/randomize_va_space
```

SMB came back to life. We wrote the fix into `/etc/sysctl.conf` so it would survive a reboot, then manually restarted
smbd through its service scripts. File sharing: restored.

---

### The Web UI: A Three-Headed Problem

_~17:27_

Getting the DSM web interface back was trickier. It turned out three separate things had gone wrong simultaneously.

**Problem one:** The web UI kept saying "system getting ready, try again later" (error 499). This turned out to be DSM
checking a tiny file — `/run/boot_seq.cfg` — for a line reading `STEP=4`, meaning "boot is complete." The file didn't
exist. We created it. The 499 went away.

**Problem two:** Without the file, synoscgi (DSM's web backend) had nowhere to put its socket. We started it manually
with the right environment variables:

```bash
SOCKET=/run/synoscgi.sock LD_PRELOAD=/usr/lib/openhook.so \
  /usr/syno/sbin/synoscgi --mode=scgi --task-per-worker=32
```

The login page appeared.

**Problem three:** Logging in still hung with a spinner forever. Tracing through the system, we found that
`/var/run/synocgid/` — the directory holding synocgid's 15 authentication sockets — had been deleted along with
everything else. synocgid was running, but deaf. We killed it and restarted it clean, and it recreated all 15 sockets.
The spinner vanished.

We wrapped all three fixes into a systemd service (`syno-boot-workaround.service`) so the NAS could survive a reboot in
its broken state while we worked on the real problem.

---

## The Real Problem: Missing Binaries

_~17:30_

The DSM web UI was now reachable, but login still hung. Digging deeper, we found lists in `/tmp/bin_broken.txt` and
`/tmp/sbin_actual.txt` — DSM's own record of what was missing. The damage was extensive: **289 binaries missing from
`/usr/syno/bin/`** and **164 from `/usr/syno/sbin/`**. Core tools, authentication helpers, disk utilities — all gone.

The obvious fix was a firmware reinstall via `synoupgrade --patch`. We had a second NAS (DS423+, same DSM version)
available to borrow files from — except it was x86_64 and the DS218j was armv7l. Every binary on one was useless on the
other.

So we turned to the firmware itself.

---

## The Upgrade Gauntlet

_18:02–18:13_

We downloaded the full 305MB firmware package — `DSM_DS218j_86009.pat` — directly to the NAS. Then `synoupgrade --patch`
spat it back:

```
"Can not downgrade."
```

The system was running DSM 86009 **Update 3**, but the base `.pat` was just 86009. From synoupgrade's perspective, this
was a downgrade.

We modified both `/etc/VERSION` and `/etc.defaults/VERSION`, setting `smallfixnumber` from `3` to `0` — making the
system believe it was running the base release. synoupgrade accepted the version change... and then returned a new
error:

```
error_verify_patch (5222)
```

The firmware's cryptographic signature check was failing. The full firmware `.pat` uses Synology's proprietary encrypted
format — not a tar, not a 7z, nothing standard. We couldn't extract it manually, and we couldn't make synoupgrade trust
it.

We were stuck. Until we checked the Synology archive and found something we'd overlooked:

```
https://global.synologydownload.com/download/DSM/criticalupdate/
  update_pack/86009-3/synology_armada38x_ds218j.pat
```

**3.15 megabytes.** The Update 3 _incremental_ patch — not the full firmware, just the delta from base 86009 to
Update 3. And because our spoofed version was now _base_ 86009, this was a legitimate _upgrade_, not a downgrade. No
version games. No signature fights.

```
synoupgrade --patch /tmp/sm_update.pat
→ check_environment: success
→ unpack: success
→ update_files: success
→ version: success
```

282 out of 289 missing binaries: restored. The remaining 7 were directories, not files — the check had been using `-f`
instead of `-e`. Everything was back.

---

## The Final Miles

_18:26–18:33_

With the binaries restored, we reset the admin password (the original hash was unrecoverable), disabled the boot
workaround service — the proper systemd units could now handle everything themselves — and rebooted.

The NAS didn't come back immediately, and for a tense few minutes it seemed like it might not. But it did. `STEP=4`. 15
synocgid sockets. synoscgi listening. System info: **DS218j, DSM 7.3.2-86009 Update 3, 40°C, uptime 2 minutes.**

The login worked.

---

## The Last Ghost: Missing CA Certificates

_~20:00_

After the reboot, almost everything worked — except the internet. `wget https://www.google.com` connected fine but threw
an SSL certificate verification error. DNS resolved, TCP connected, but every HTTPS request failed. DSM couldn't check
for updates.

The firmware update had restored 282 system binaries, but it doesn't touch the CA certificate store — that's managed
separately. A check revealed the damage: `/etc/ssl/certs/` had been gutted from a full store of ~299 certificates down
to just 13. The `ca-certificates.crt` bundle was gone entirely.

The fix was a two-step copy from the second NAS (both running the same DSM version). CA certificates are plain text PEM
files — architecture-independent — so they could just be copied.

A final audit of the root filesystem — comparing modification timestamps between the incident window and the firmware
update — found no other missing config. The only other traces of the incident were a leftover `webui.bak` file (DSM had
auto-backed up its PAM config during recovery) and a package manager status record showing non-zero crash counts from
when we'd been killing daemons. Both cleaned up in seconds.

---

## What We Learned

A Synology NAS, even with most of its system binaries deleted, can be saved. The data volume is separate from the
system. The boot sequence is just a file. The authentication layer is just a daemon with some sockets. The firmware
update system — if you know how to talk to it — can put everything back, one layer at a time. And the CA certificate
store, the one piece the firmware _doesn't_ restore, can be borrowed from a sibling NAS in a single pipe.

The NAS lived.
