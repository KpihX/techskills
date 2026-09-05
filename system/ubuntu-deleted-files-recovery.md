# 🖼️ Ubuntu — Recover deleted images (ext4)

> **Machine:** KpihX-Ubuntu · **FS:** ext4 `/dev/nvme0n1p4` · **DE:** GNOME  
> **Canonical skill ref:** `$HOME/.agents/skills/k-optim/references/ubuntu-deleted-files-recovery.md`

Short guide for recovering images after Trash was emptied. Full procedure, scripts, and pitfalls live in the **k-optim** reference above.

---

## TL;DR strategy

| Tier | Tool | Sudo? | What you get |
|------|------|-------|----------------|
| **A** | `recently-used.xbel` + `~/.cache/thumbnails/` | No | Original **filenames**, **preview** quality (~512 px) |
| **B** | PhotoRec `freespace` | Yes (tmux) | **Full resolution**, anonymous `f1234567.jpg` names |
| **C** | extundelete | Yes + **unmounted** FS | Paths + files if inodes intact |
| **D** | Telegram re-export | No | Cloud copy if still in chat |

**2026-05-26 session:** Tier A → **103 previews** (all RAS VeniceAI + most Telegram JPG). Tier B → **~1500** carved images in `~/Recovery_PhotoRec_2026-05-26/photorec.*`.

---

## Tier A (try first)

```bash
grep -oP 'file:///home/kpihx/Downloads/[^<"]+' \
  ~/.local/share/recently-used.xbel | sort -u
ls ~/.cache/thumbnails/x-large/ | head
```

Copy matching hashes back to `~/Recovery_candidates_*/from_thumbnails/` with original basenames (see canonical ref for Python hash snippet).

---

## Tier B (PhotoRec)

**Sudo only in tmux** `default:0.0` — no split panes.

```bash
sudo bash ~/Recovery_PhotoRec_2026-05-26/run_photorec_only.sh
```

**Do not** use `jpeg`, `webp`, or `heic` in `/cmd` — PhotoRec 7.2 rejects them. Use `jpg`, `png`, `gif`, `tif`, `bmp` only.

---

## See also

- [🧹 Cleanup Suite](system/clean.md) — prevention (empty `Downloads/` after use)
- [k-optim Ubuntu scope](file://$HOME/.agents/skills/k-optim/SKILL.md) — home media policy
