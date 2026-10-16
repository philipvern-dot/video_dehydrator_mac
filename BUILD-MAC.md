# Build Video Dehydrator for Mac

This file is the whole brief. Build a new Mac program from this document. Do not look for a Windows or Linux source tree. Do not publish, push, tag, or create a GitHub release. Do not convert or delete files on a network share unless the user asks.

This is the Mac edition of the shipping Video Dehydrator (Windows 1.0.6.1). Match that program's jobs, wording, list, and rules. Change only the pieces that cannot exist on macOS: the process launcher, sleep, shutdown, mount paths, and the update file. The Mac app's own version is `1.0.0`.

The program finds videos that are too big for their picture size, encodes a smaller copy, and puts that copy where the original was. The original waits in a folder named `.vd-originals` until the user deletes it.

## What to build

A folder the user can run on this Mac:

```text
video-dehydrator-mac/
  version.txt
  requirements.txt
  README.md
  LICENSE
  vd-icon.png
  tools/
    ffmpeg
    ffprobe
    ffplay
    HandBrakeCLI
    SOURCES.txt
    FFmpeg-LICENSE.txt
  app/
    video_dehydrator/
      __init__.py
      __main__.py
      engine.py
      store.py
      recycle.py
      compare.py
      helptext.py
      selftest.py
      ui.py
```

`version.txt` is `1.0.0`. `requirements.txt` is `PySide6>=6.6`. The license is MIT. The README says HandBrake is GPL-2.0 and FFmpeg is GPL-3.0, and points at `tools/SOURCES.txt`.

Create `vd-icon.png` before the program is finished: a square picture with transparent corners and a mark that reads as video being shrunk. Set it as the window icon and the Dock icon. A later install uses the same picture.

`__main__.py` starts the window. `python3 -m video_dehydrator --self-test` runs the tests and does not open a window.

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python -m video_dehydrator --self-test
python -m video_dehydrator
```

Run from the project root with `PYTHONPATH` pointing at `app`. A `video-dehydrator` script in the project can set that path and exec the venv Python.

A `.app`, a `.dmg`, code signing, and notarization wait until this window matches the behavior below. The running program does not wait on those.

## Tools

The program uses only the copies in `tools/`. It does not call `ffmpeg` or `HandBrakeCLI` from `PATH` at runtime.

`ffmpeg`, `ffprobe`, `ffplay`, and `HandBrakeCLI` are all required. If any is missing, name the paths in the activity log and in an error dialog, and keep Scan disabled.

Find the chip with `uname -m`. `arm64` and `x86_64` need matching binaries. Check each one with `file`.

Likely locations:

- Apple silicon Homebrew: `/opt/homebrew/bin/ffmpeg`, `ffprobe`, `ffplay`, `HandBrakeCLI`
- Intel Homebrew: `/usr/local/bin/` for the same names
- HandBrake app: `/Applications/HandBrake.app/Contents/MacOS/HandBrakeCLI`

Symlink or copy those into `tools/`. If they are missing, ask the user before installing the Homebrew packages `ffmpeg` and `handbrake`. The `ffmpeg` package must include `ffplay`. Do not download the Linux static FFmpeg archive. Do not use Flatpak.

Confirm `HandBrakeCLI --help` accepts `--multi-pass`, `--vb`, `--audio-copy-mask`, and `--audio-fallback`. If one is missing, stop and tell the user. Do not switch the encoder to VideoToolbox, NVENC, or any hardware encoder.

`tools/SOURCES.txt` names the HandBrake version, the FFmpeg version, and where each binary came from.

## Window

One main window, title `Video Dehydrator`. Aim for a client area of 1120 by 920. Shrink it to fit the screen, and keep a minimum near 960 by 760. Page background `#eef1f4` (`238,241,244`). Ink `#111827`. Muted text `#4b5563`. Body text `#374151`. Hairline `#d6dae1`.

Use the system UI font at the sizes below. The activity box uses a monospace font.

From top to bottom:

1. Dark header, about 112px, background `#1c2430`. Title `Video Dehydrator` in white, about 22px bold. Under it, in `#d1d5db`: `Shrink videos that are too big for their picture size. The smaller file takes the original's place.` On the right, two rows. Top row: Check for updates, Update. Bottom row: Donate, Report a bug. Donate and Report a bug stay enabled during a job. Check for updates and Update are disabled while a scan or convert is running.

2. Folder card. Label `Folder`, a path box, Browse. Browse is a folder dialog. Its description is `Choose the folder to scan`. It starts at the saved folder and does not offer to create a folder. Dropping a folder on the window or the path box sets the folder. Dropping a file sets the folder that contains that file. Ignore drops while a job is running.

3. Hint row. Left: `Include subfolders looks inside every folder within this one. Checked rows are the ones Convert will shrink.` Right: the estimated-savings label. It always shows, including `Estimated savings: 0 B`.

4. List header. Label `Videos`. Then a folder dropdown, Select folder, an overage dropdown, a select-overage button, Select all, Deselect all, Clear. The folder dropdown lists folders that contain Bloated or Failed files, each as `relative/path  (3 files)`. A parent folder is listed as well as the season folder inside it, so a show and its seasons both appear. Skip the scan root itself when it has no bloated file of its own and every bloated file is already counted in a child. Select folder checks only Bloated and Failed rows in that folder and its children, clears the other checks, and logs `Checked N files in <relative folder>.` The overage dropdown items are `Slight  (N files)`, `Heavy  (N files)`, and `Extreme  (N files)`. The button text follows the choice: `Select slight`, `Select heavy`, or `Select extreme`. It checks only that band and clears the other checks. It does not start Convert. Log `Checked N files slightly over budget.` Use `heavily over budget` and `extremely over budget` for the other bands. The button is disabled when that band's count is 0. Clear is disabled when the list has no scanned rows.

5. The file table. One selection at a time. Checkboxes. Grid lines off. White background. Show a row tooltip.

6. Options card, white. Left column, stacked: Include subfolders, Delete originals automatically, Re-encode audio that cannot be copied, Show detailed activity, Shutdown computer upon completion, then Clear skips. Right column, titled Budgets, split near x=360: a 2 by 2 grid of 2160p, 1080p, 720p, and 480p. Each has a 14px color swatch, a colored label, and a dropdown. Clear skips is disabled when the skip list is empty. All of these controls are disabled while a job is running.

7. Action row. Left: Scan, Convert, Cancel. Right: Compare, Undo, Delete originals, How this works. Scan and Convert are the primary blue buttons (`#1d4ed8`). Cancel is enabled only while a job is running. Escape cancels the running job.

8. Activity log, about 180px, background `#1c2430`, monospace. See Activity log.

9. Status bar. Status text on the left, version from `version.txt` on the right. Under that, a thin progress track about 8px tall. The percent is not drawn inside the bar. During an encode the status line carries the percent.

Startup lines, in order:

```text
Ready. Choose a folder, then click Scan.
Scan lists videos that are too big for their picture size. The Budgets row starts on Recommended.
Convert puts the smaller file where the original was and keeps the original until you delete it.
```

Status starts as `Ready`.

## Tooltips

Copy these. Where a Windows sentence is named, the Mac wording is already substituted.

- Path box: `The folder Scan will look through. Drop a folder here, or click Browse. The choice is remembered.`
- Savings: `Current size minus the New size column, for the checked files Convert will shrink. A New size that starts with ~ had no picture rate, so that guess can run high. It changes as you check and uncheck rows.`
- Browse: `Choose the folder to scan.`
- Include subfolders: `Look inside folders within the chosen folder. Turn this off to scan only the files sitting directly in that folder.`
- Delete originals automatically: `After a new file checks out, delete the original. The first time, the program explains what you give up. On a shared drive, the matching #recycle copy is removed when the path and the size match. The .plexignore from that convert is removed when it only hides the holding folder, and empty folders from the delete are removed when nothing else is in them. Leave this off to compare first and delete with the Delete originals button.`
- Re-encode audio: `Some audio, such as Windows Media Audio, cannot be copied into the new file. With this on, that audio is re-encoded to AAC and the convert can finish. With this off, the convert stops and the original stays. Audio that can be copied is still copied either way.`
- Show detailed activity: `List each encode step in the activity box. The running percent stays on the bar, not as a new line.`
- Shutdown: `After Convert finishes, turn the computer off. A one-minute countdown lets you cancel. The computer stays on if you stop the convert. A scan does not turn it off.`
- Clear skips: `Forget files that were left off later scans because the new copy was larger. Scan again after this to list them. A smaller budget lists them again without clearing.`
- Budgets title: `The size line for each picture. Recommended is where the program starts. Scan again after you change one.`
- Each budget dropdown: `<name> budget, <current choice>. Files with this color in Dimensions follow this choice. The new file is encoded at <kbps> kbps. HEVC and AV1 are left alone under <hevc GiB/h>. A nearby picture size follows this choice. The choice is remembered.`
- Scan: `Read the videos in the folder and list the ones that are too big for their picture size.`
- Convert: `Shrink every checked video that is bloated, failed, or skipped earlier. The new file takes the original place. A file marked Skipped was left alone because the new copy was not smaller.`
- Cancel: `Stop the scan or the file that is converting. Esc does this too.`
- Compare: `Play the original and the new file side by side. The sound will not stay in sync with the picture. Go to the file on disk to confirm the audio.`
- Undo: `Put the originals from the last conversion back, and delete the new files. The program asks first.`
- Delete originals: `Delete the checked originals that are waiting in .vd-originals. The new files stay. On a shared drive, the matching copy in #recycle is removed when the path and the size match. The .plexignore from that convert is removed when it only hides the holding folder, and empty folders from the delete are removed when nothing else is in them.`
- How this works: `Open a short guide to the size rule, comparing, and what each part of this folder is for. The guide includes the steps to clear a drive recycle bin by hand.`
- Check for updates: `Look on GitHub for a newer version of Video Dehydrator. A window opens the first time a newer version is found. The status line keeps saying that version is available.`
- Donate: `Free forever. Tips of $5 or more can include a feature suggestion.`
- Report a bug: `Open a GitHub bug report with this version and the Mac system name filled in. Include latest activity is optional. Leave it unchecked and nothing else is added.`
- Update: `Download the newer Mac version and open it. This window closes while that installer opens.`
- Folder dropdown: `Folders that have bloated files. A show is listed with the folders inside it. Pick one to make a smaller convert job.`
- Select folder: `Check the bloated files in the folder shown here, and clear the other checks.`
- Overage dropdown: `How far each file is over its budget. Slight is under 1.5 times, Heavy is under 3 times, and Extreme is 3 times or more.`
- Select overage: `Check the files in the group shown here, and clear the other checks. Convert then shrinks that group.`
- Select all: `Check every row.`
- Deselect all: `Clear every check.`
- Clear: `Remove the scanned files from the list. Files still waiting in .vd-originals stay.`
- Table: `Bloated videos are checked. Over shows how many times the budget each file uses. Space Saved is the original size minus the converted file, filled in after a convert finishes. Click a heading to sort. Drag a heading sideways to move that column. The order is remembered. Right-click a row to select that folder or that overage. Double-click a file that is ready to delete to compare it.`
- Activity: `Saved space, skipped videos, and problems are listed here.`
- Progress: `How far the current file has been encoded.`
- Status: `What the program is doing right now.`
- Version: `This copy of Video Dehydrator.`

Donate opens exactly `https://ko-fi.com/philipvern`.

## List

Columns, in this order, with these starting widths: File 280, Dimensions 120, Length 80, Size 170, GiB/h 70, Budget 70, Over 78, Target 100, New size 110, Status 150, Space Saved 120. File stretches to fill the remaining width and stays at least 160.

| Column | Text |
| --- | --- |
| File | Path relative to the scanned folder. A file outside that folder shows its full path. |
| Dimensions | `1920x1080`, or blank. |
| Length | `h:mm:ss` when an hour or more, otherwise `m:ss`. |
| Size | Byte size. After a shrink, `400.0 MiB from 1.30 GiB` when the old and new sizes differ. |
| GiB/h | Two decimals. |
| Budget | Two decimals. |
| Over | `2.0x`. |
| Target | `3000 kbps`. |
| New size | Byte size, or `~` plus the size when the picture rate was missing. Blank when there is no estimate. |
| Status | The status word. |
| Space Saved | Old minus new after a successful shrink. Blank before that, blank when the new file is not smaller, cleared on undo, kept after the original is deleted. |

Byte sizes: GiB with two decimals at 1 GiB and up, MiB with one decimal at 1 MiB and up, KiB with no decimals at 1 KiB and up, otherwise `N B`. GiB is `1024^3`.

A heading click sorts. Do not sort during a job. The first click on Over (column 6) and Space Saved (column 10) sorts largest first. Other columns sort ascending first. A second click flips the direction. The sorted heading shows `^` or `v`. Empty values sort after values that exist. Ties break by full path, ascending. File sorts by full path. Dimensions sorts by pixel count. Length, Size, GiB/h, Budget, Over, Target, New size, and Space Saved sort as numbers. Status sorts as text.

Drag a heading sideways to reorder columns. A move of at least 6 pixels starts the drag. Dragging the divider resizes the column and does not reorder. After a drop, save the order immediately. A drop also produces a click; swallow that one click so the sort does not change. Restoring an order repaints the rows under the new headings. `columnOrder` is used only when it is a complete permutation of the current titles. A short list, a duplicate, or an unknown name keeps the default order. An update that changes the column set may put the headings back, and that is acceptable.

Right-click a row and select it. The menu is Select all, Deselect all, Select files in this folder, Select this overage, Clear. When the row has a band, the overage item reads `Select slight files`, `Select heavy files`, or `Select extreme files`. Disable items that cannot run: busy rows, no selected row, no band, or nothing scanned to clear.

Double-click the selected row when Compare is enabled and open Compare.

Row tooltip, one fact per line: full path, reason, then when the row is Bloated or Failed and has a band `2.0x the budget (Heavy).`, then `Follows the 1080p budget.` when the picture has an anchor, then `New size starts with ~ because the picture rate could not be read. That guess can run high.` when the estimate is rough, then `Space saved is the original size minus the converted file.` when that number exists.

Color the Dimensions cell from the anchor the file follows. Color the Over cell only for Bloated and Failed.

| Anchor | Swatch | Cell background | Cell text |
| --- | --- | --- | --- |
| 2160p | `#1d4ed8` | `#dbeafe` | `#1e3a8a` |
| 1080p | `#0f766e` | `#ccfbf1` | `#115e59` |
| 720p | `#6d28d9` | `#ede9fe` | `#5b21b6` |
| 480p | `#c2410c` | `#ffedd5` | `#9a3412` |

| Band | Range | Background | Text |
| --- | --- | --- | --- |
| Slight | under 1.5x, and at least 1x | `#dcfce7` | `#166534` |
| Heavy | from 1.5x up to under 3x | `#fef9c3` | `#854d0e` |
| Extreme | 3x or more | `#fee2e2` | `#991b1b` |

Ratios under 1 have no band. Skipped, Ready to delete, Replaced, and the other finished statuses can show the ratio and stay uncolored.

Checked when added: Bloated, Failed, and Ready to delete. Skipped is unchecked. Select all checks every row. Deselect all only clears checks.

Estimated savings adds current size minus New size for checked Bloated, Failed, and Skipped rows. Show `Estimated savings: 0 B` when none of those rows are checked. When a checked row has no usable New size, leave it out and append `, 1 file not included` or `, N files not included`. The savings label does not get a tilde. The tilde stays on that row's New size. Finished rows are not part of the total. Update the label as checks change.

Clear removes scanned rows and keeps rows that still have a pending id. Log `Cleared the scanned files. Files still waiting in .vd-originals stay.` A new Scan also drops previous scan rows and puts the pending rows back.

Do not list a scanned file whose path is already the new path of a row on screen.

## Size rule

Reference picture is 1920x1080. Reference video rate is 3000 kbps. Reference file budget is 2 GiB per hour. HEVC and AV1 use 3 GiB per hour at that same picture. The floor is 200 kbps. Encode is fixed-rate average bitrate. There is no quality slider and no resolution change.

Scale is `(width * height) / (1920 * 1080)`.

Target kbps is `round(3000 * scale * budget factor)`, then raised to 200 if it is lower.

File budget in GiB/h is `2 * scale * factor` for other codecs, and `3 * scale * factor` when the video codec is `hevc`, `h265`, or `av1`.

Budget factor comes from the nearest of the four anchors by pixel count, then the dropdown for that anchor:

| Choice | Factor |
| --- | --- |
| Half | 0.5 |
| Recommended | 1.0 |
| 1.5 times | 1.5 |
| Double | 2.0 |

Anchors: 3840x2160, 1920x1080, 1280x720, 720x480. A 720p file uses the 720p dropdown even when the 1080p dropdown changes.

At Recommended the self-test must hit:

| Picture | kbps | File budget | HEVC or AV1 budget |
| --- | --- | --- | --- |
| 1920x1080 | 3000 | 2 GiB/h | 3 GiB/h |
| 3840x2160 | 12000 | 8 GiB/h | 12 GiB/h |
| 1280x720 | 1333 | 0.888... GiB/h | 1.333... GiB/h |
| 720x480 | 500 | 0.333... GiB/h | 0.5 GiB/h |

A packed budget string looks like `2160=recommended;1080=plus;720=recommended;480=recommended`. With 1080 set to 1.5 times, the 1080p HEVC line is 4.5 GiB/h and the target is 4500 kbps. Dropdown item text is `Recommended, 2 GiB/h`, with trailing zeros trimmed from the number.

GiB/h of a file is `(size_bytes / 1024^3) / (duration_seconds / 3600)`.

A file is listed only when the decision is bloated. Rules, in order:

1. Dolby Vision is left alone (`dv`). Detect a video codec tag of `dvhe`, `dvh1`, `dva1`, or `dvav`, side data whose text contains `DOVI`, `Dolby Vision`, or `dovi`, or a `dovi_configuration` record.
2. Missing duration or picture size is left alone.
3. File GiB/h under the budget is left alone (`lean`).
4. Otherwise, if the video bitrate is known, is at least 40 percent of the whole-file bitrate, and is at most 105 percent of the target video bitrate, the file is left alone (`within`). Extra size is audio or other tracks.
5. Everything else that is at or over the budget is bloated.

Video bitrate is the stream `bit_rate` when it is a positive integer. Otherwise a tag named `BPS`, or a tag whose name starts with `bps-`. Otherwise `NUMBER_OF_BYTES` times 8, divided by duration.

Changing a budget recalculates Bloated, Failed, and Skipped rows from the stored width, height, size, duration, video bitrate, and codec. Rows that fall under the new line leave the list. Log which anchors changed, for example `1080p is 1.5 times, 4.5 GiB/h, encoding at 4500 kbps.` When one file leaves, add `1 file fell under the new budget and left the list.` When a factor went down, add `Scan again to include files that were under the old budget.` Join those notes into one activity line.

Over is file GiB/h divided by budget GiB/h.

New size keeps the non-video bytes when the video bitrate is known: other bits per second is file bits per second minus video bitrate, floored at 0, then `((target_kbps * 1000) + other) * duration / 8`. When the video bitrate is missing, New size is the target video rate times the length and `EstimateRough` is true. Files with no duration have no estimate.

## Scan

Video extensions: `.avi`, `.mpg`, `.mpeg`, `.mp4`, `.mkv`, `.m4v`, `.wmv`, `.mov`, `.ts`, `.vob`, `.m2ts`, `.webm`, `.flv`, `.divx`.

Skip names ending in `.vd.tmp.mkv` or `.hb.tmp.mkv`, and names containing `.hb.src.bak`. Do not follow symlinks. Do not enter a directory named `.vd-originals`. Include subfolders walks every other directory under the chosen folder. With it off, only the files in that folder are read.

If the folder is missing, say `Choose a folder that is on this computer.` Scan requires the tools and a non-empty path.

Probe with `ffprobe -v error -show_format -show_streams -print_format json -- <file>`. Use the first video stream that has a width and is not an attached picture. Duration and size come from the format. If size is missing, use the file size.

On start, log `Scanning <folder>` and set status `Listing videos...`. As files are read, set status `Scanned <seen>, <hits> bloated - <file name>`.

A remembered size skip hides the file when the path matches, the size matches, and the current target kbps is the same or higher than the saved target. A smaller budget lists it again.

When the scan finishes, status and an ok activity line:

```text
Scanned 12 videos. 3 bloated (4.20 GiB).
```

Omit the size parentheses when the bloated bytes are 0. Then one dim line of the counts that are greater than zero, joined with `. `:

- `N already under the budget`
- `N already at the picture rate`
- `N Dolby Vision left alone`
- `N could not be measured`
- `N could not be read`
- `1 file left off the list because an earlier convert was not smaller. A smaller budget lists it again. Clear skips forgets that mark`
- `N files left off the list because an earlier convert was not smaller. A smaller budget lists them again. Clear skips forgets those marks`

If nothing is bloated, something was seen, and nothing was ignored as a size skip, add `Nothing in this folder is over the budget for its picture size.` A cancelled scan sets status `Scan stopped` and logs `Scan stopped. Files already listed stay in the box.`

Clear skips asks `Forget N file(s) that were skipped because the new copy was larger?` and says the next scan can list them again. Yes forgets every mark and logs `Forgot N skipped files. Scan again to list them.`

## Encode

One file at a time. Convert runs the checked rows whose status is Bloated, Failed, or Skipped. Log `Converting N files.` If re-encode is on, log `Audio that cannot be copied will be re-encoded to AAC. Other audio is still copied.` If shutdown is on, log `The computer will shut down when this convert finishes.`

Before each file, set status `File N of M - <name>`. If the file is gone, status Failed, reason `The file is no longer there.`, log `Missing  <path>`. Probe again. If the decision is no longer bloated, set status `Already small enough`, uncheck the row, and log `Left alone  <name>  <reason>`. If another file already has the `.mkv` name, fail before encoding.

Software x265 only. 8-bit progressive SDR uses encoder `x265`. 10-bit, HDR10, HDR10+, or HLG uses `x265_10bit`. 10-bit means the pixel format contains `p10`, `10le`, or `10be`. HDR10 means color transfer `smpte2084` or side data mentioning mastering display or content light level. HDR10+ means side data mentioning `HDR10+`, `SMPTE2094-40`, or `SMPTE 2094`. HLG means color transfer `arib-std-b67`. Interlaced means field order is set and is not `progressive` or `unknown`.

Write the new file to `.vd-originals/<stem>.vd.tmp.mkv` beside the original. The finished file is `<stem>.mkv` in the original's folder. Container is MKV. The picture is not cropped or scaled. Frame rate stays variable as the source had it. Create the holding folder and the `.plexignore` before the encode. Remove a leftover temp file from an earlier run.

Arguments for an 8-bit file with subtitles, re-encode off, Recommended 1080p:

```text
HandBrakeCLI -i SOURCE -o TEMP --format av_mkv --encoder x265
  --encoder-preset medium --vb 3000 --multi-pass --turbo --vfr
  --crop-mode none --crop 0:0:0:0 --markers
  --all-audio --aencoder copy
  --audio-copy-mask aac,ac3,eac3,truehd,dts,dtshd,mp2,mp3,flac,opus
  --audio-fallback none
  --all-subtitles --subtitle-burned=none --subtitle-default=none --keep-subname
  --no-comb-detect --no-deinterlace --no-decomb --no-bwdif
```

Changes from that base:

- No subtitle streams: `--subtitle none` instead of the `--all-subtitles` group.
- Re-encode audio on: `--audio-fallback av_aac`. The checkbox is off by default.
- HDR10+: add `--hdr-dynamic-metadata hdr10plus`.
- Interlaced: `--comb-detect default --decomb` instead of the four `--no-*` filters.
- 10-bit or HDR: `--encoder x265_10bit`.

Audio copy mask does not include `wmav2`. With the checkbox off, a file whose audio cannot be copied fails and the original stays. With it on, an uncopyable codec may become AAC. A copyable codec that changes, or a track that disappears, still fails.

Log `Converting  <name>  at 3000 kbps`. With detailed activity on, also log the full argument line. Fold HandBrake's `Encoding: task N of M, P %` lines into one percentage: `((task - 1) + P/100) / tasks * 100`. Status becomes `File N of M - <name> - 40%`. Treat `Encoding:` and `Muxing:` percent lines as progress noise. Keep the last 500 HandBrake lines for the failure reason. With detailed activity on, log the other lines in the dim color. An ERROR string inside HandBrake output is not, by itself, an app failure.

While the encode runs, show a Working mark. See Activity log.

Cancel stops the running file, kills the HandBrake, ffmpeg, ffprobe, and ffplay processes this program started, deletes the temp file, sets that row to `Stopped` with reason `Cancelled before the new file was checked.`, and leaves finished files in place. Status `Convert stopped`. Log `Convert stopped. Finished files stay in place.` If shutdown was checked, log `Convert was stopped, so the computer will stay on.`

When the batch finishes, status and log `Finished. Saved 1.20 GiB.` Saved is the sum of old minus new for successful replacements. If Auto is off, also log `Compare a file, then click Delete originals when it looks right.` Then show any recycle problems, then offer shutdown.

## Check the new file

Set the row to `Checking` and the status line to `Checking the new file` before the check.

Reject the encode when any of these are true:

- The temp file is missing or smaller than 64 KiB.
- HandBrake's exit code is not 0. Reason starts `HandBrake stopped (<code>).` plus the last six log lines joined with ` | `.
- The HandBrake log says passthrough was impossible or a fallback encoder was used, and re-encode is off.
- The log says a subtitle was burned in (`Burned-in` or `Render/Burn-in`).
- Duration differs by more than 2 seconds.
- Width or height changed.
- The first 15 seconds or the last 15 seconds fail to decode. The command is `ffmpeg -hide_banner -nostdin -v error -xerror`, plus `-ss 0` for the start or `-sseof -15` for the end when the file is longer than 15 seconds, then `-t 15 -i FILE -f null -`. A shorter file uses its whole length, at least 1 second. Exit 0 whose only message is FFmpeg's equal-DTS note is success. The note looks like `Application provided invalid, non monotonically increasing dts to muxer in stream 0: 10 >= 10` and the two numbers must be equal. A backwards timestamp, any other message, or a non-zero exit fails.
- Audio track count changes, or a track's codec changes. The exception is re-encode on, and the old codec is outside the copy mask, and the new codec is `aac`.
- Subtitle count drops.
- The new file is not smaller in bytes, or not smaller in GiB/h.

When the only problems are `the new file is not smaller` and `the new file is not smaller per hour`, build the larger-file note, then append ` Later scans leave this file off the list. A smaller budget lists it again. Clear skips forgets that mark.` The row becomes Skipped, unchecked. Log `Skipped  <name>  <note>` as a warning. Convert log status is `SKIP`. Remember the path, size, and target kbps. Mixed size-plus-other problems stay Failed and checked. Log `Failed  <name>  <reason>` as an error, and log the larger-file note too when one exists. Delete the temp file on every failure. The original stays.

The larger-file note says the original was left in place, states the target rate, says whether audio was copied or an uncopyable track was re-encoded to AAC, gives both sizes, and ends with `That happens when the target rate plus the audio is more than this file already used.`

## Replace, undo, and delete

On success, move the original into `.vd-originals` and move the temp file onto `<stem>.mkv`. If a held file of that name already exists, insert a timestamp before the extension. If the source itself is already that `.mkv`, move it aside first, then move the temp into its place. If the second move fails, move the original back. If the file is under 64 KiB after the move, put the original back and fail.

Create or append `.plexignore` in the video's folder with exactly these two lines when they are missing:

```text
.vd-originals
.vd-originals/*
```

A `.plexignore` that also names other things keeps those other lines. Remove the two lines, and the file when nothing else remains, only after `.vd-originals` itself is gone.

`app/pending.json` is a JSON array of objects. One object per held original. Never wrap that array in a second array. Keys:

```json
{
  "id": "hex",
  "source": "/path/original.avi",
  "kept": "/path/.vd-originals/original.avi",
  "new": "/path/original.mkv",
  "when": "2026-10-15T10:44:19",
  "oldBytes": 1400000000,
  "newBytes": 429000000,
  "width": 1920,
  "height": 1080,
  "targetKbps": 3000,
  "batch": "hex"
}
```

`when` is local `yyyy-MM-ddTHH:mm:ss`. One Convert click shares one batch id.

After a kept success: status `Ready to delete`, checked, reason `Original is in .vd-originals. Compare, then delete it when the new file looks right.` Log `Ready  <name>  saved 900.0 MiB. Original kept for review.` Convert log status `KEPT`.

After Auto deletes the original: status `Replaced`, unchecked, reason `Original deleted after the check.` Log `Replaced  <name>  saved 900.0 MiB.` Convert log status `REPLACED`. Then run the recycle purge.

A successful convert removes that path from the size-skip list. Size becomes the new size, OldBytes becomes the original size, and GiB/h becomes the new rate. New size estimate is cleared by leaving the estimate empty so the cell goes blank. Space Saved fills in.

On the next launch, a pending row whose kept file still exists shows as Ready to delete, checked, with the same reason. If the new file is missing, status is `New file missing` and the reason is `The new file is missing, so the original stays.`

Undo is enabled when any pending kept file is still on disk. It restores the newest batch. Newest means the greatest `when`. When that row has a batch id, restore every still-present row with that id. Older rows with no batch id restore the ones whose `when` starts with the same calendar day. Ask first:

```text
Undo the last conversion?

N files will be put back, and the new files will be deleted.

name one
name two
and 3 more.
```

Show at most eight names. The held file must be a regular file inside `.vd-originals`. The new file is removed. The original returns to its old path. A missing held file cannot be undone. The row becomes `Restored`, unchecked, path returns to the source, OldBytes clears, SizeBytes returns to the old size, reason `The original was put back and the new file was deleted.` Log `Restored  <name>`. Convert log status `UNDONE`. Status line `Restored N files.` Close Compare if it is open.

Delete originals removes checked Ready to delete rows. It refuses any path that is not inside `.vd-originals`. If the new file is missing, the original stays. Ask first:

```text
Delete N originals?

The new smaller files stay where the originals were. On a shared drive, the matching copy in #recycle is removed when the path and the size match. The .plexignore from that convert is removed when it only hides the holding folder.
```

Checkbox: `Don't show this in the future`. Hint under it: `This question comes back when a new version of the program is installed.` Buttons: Delete and Cancel. Remember the checkbox only when Delete is confirmed with the box checked, by saving `deleteConfirmVersion` as the current app version. A different version asks again. Cancel does not remember. The row becomes `Original deleted`, unchecked, kept path cleared, reason `The held original was deleted.` Log `Deleted original of <new file name>`. Status line `Deleted N originals.`

When Auto is turned on the first time, explain:

```text
Delete originals automatically removes each original as soon as its new file checks out.

The check looks at size, length, picture size, audio, subtitles, and whether the start and end play. It does not judge how the picture looks.

After the original is deleted, Compare and Undo cannot bring it back.
```

Buttons: Turn auto on, Leave it off. Checkbox: `Don't show this again`. That checkbox is saved only with Turn auto on, in `autoWarnAccepted`. Leave it off clears the Auto checkbox. Turning Auto on must not flip itself off because a dialog opened. Apply the choice once.

Before every convert while Auto is on, ask OK or Cancel:

```text
Auto is on.

After each file checks out, its original is deleted. The new file stays in the original's place. There will be nothing left to compare. On a shared drive, the matching copy in #recycle is removed when the path and the size match. The .plexignore from that convert is removed when it only hides the holding folder.
```

Cancel leaves the queue unstarted.

## Compare

Compare is enabled when one selected row is Ready to delete and both the kept original and the new file exist. If either is missing, say `The original and the new file both need to be on disk to compare them.` If duration is missing, probe the new file and fill it in.

The window title is `Compare`. Picture area is black. Minimum about 760 by 480, opening near 1100 by 680. Header bar is 86px tall, background `#1c2430`, so the note can wrap. The title is the new file's name in white. Under it, in `#d1d5db`: `Original on the left. New file on the right. Sound is the new file, and it will not stay in sync with the picture. Go to the file on disk to confirm the audio.`

Each side is 960x540 at 12 fps, scaled down with black bars, stacked horizontally. Pump raw `bgr24` frames from ffmpeg. A bottom bar holds Play or Pause, a slider, and a clock `m:ss / m:ss`. Play starts immediately. Pause freezes the picture and stops the sound. The slider moves both pictures together. Dragging the slider seeks. At the end, stop and the button returns to Play. Space toggles play and pause while this window is focused. Tooltip on the button: `Play or pause the side by side preview. Space does this too.` Tooltip on the slider: `Move through both copies together.`

Sound is the new file through `ffplay -hide_banner -loglevel error -nodisp -vn -autoexit -ss <start> -i <new file>`. Start it when the first picture frame arrives, so the clock and the sound begin together. They are still separate players and will drift. Do not build a single locked player. Stop ffplay when playback stops, when Compare closes, when a convert or delete or undo starts, and when the window closes.

## Drive recycle bin

A NAS share can intercept a delete and keep a copy in `#recycle` at the top of that share, with the same relative path. The first delete does not free the space. A second delete of that copy removes it. This is the share's bin, not the Mac Trash. Never touch `~/.Trash`. Never empty `#recycle`. Never delete a video whose path does not contain `.vd-originals`.

Run `mount` and parse lines like:

```text
//user@PhilipSNAS/UnifiedMedia on /Volumes/UnifiedMedia (smbfs, nodev, nosuid, mounted by philip)
```

The share root is the mount point. The file system type is the first token inside the parentheses. Treat `smbfs`, `nfs`, `cifs`, `webdav`, and `afpfs` as shares. `apfs` and `hfs` are local disks. Pick the longest mount point that contains the file. A local disk produces no recycle plan. The usual share on this Mac is `/Volumes/UnifiedMedia`.

The copy to remove is `share_root + /#recycle/ + path relative to the share`. For a file, the parent folder in that relative path must be `.vd-originals`. Refuse `.` or `..` segments. Refuse a relative path that already contains `#recycle` or `@recycle`. The file must be a regular file, not a symlink, and the byte size must match the file just deleted. Then delete that one file. A temp file removed from `.vd-originals`, and the new file discarded by undo, use the same rule.

Remove empty directories upward and stop at `#recycle`. Do not remove `#recycle`. If the only listed child is the directory just removed, recheck a few times about 150 milliseconds apart, because SMB listings go stale. A folder that contains anything else stays, so two computers can delete at once.

The `.plexignore` copy under `#recycle` is removed only when the path and size match and the file's non-empty lines are exactly the two holding-folder lines. Any other contents stay.

Log each recycle result once. `purged` is ok. `denied`, `failed`, and `remained` are errors. `missing` and `mismatch` are warnings. `absent` is a normal line, logged once per message, and does not open the window. `skipped` is not shown.

When a real problem remains, after that delete, undo, or convert, open one window. Intro:

```text
The original is already deleted from the library. The drive recycle bin may still have a copy, and the program could not remove it.

The link below opens the manual steps. Those steps remove only .vd-originals folders inside #recycle. A share that does not keep a recycle bin can show this same window, and then there is nothing to remove.
```

List up to eight problems: name, message, and recycle path. If more remain, add `and N more.` Link: `Show the manual deletion steps`. Button: Close. Opening the link sets a hold flag and opens How this works at Drive recycle bin. If that happened and shutdown was checked, do not shut down. Log `The computer will stay on so the drive recycle bin can be cleared.` No `#recycle` directory means the share kept nothing, and no window is shown.

## Sleep and shutdown

During scan and convert, start `caffeinate -i -s` and keep it running. Refresh is unnecessary while that process lives. Stop it when the work returns to idle, on error, or when the window closes. That blocks idle sleep. The display may turn off. Do not pass `-d`. Lid close and a manual Sleep still work.

Shutdown runs only after a convert batch finishes, and only when the checkbox is on. It does not run after a scan, after a cancel, or after the recycle help was opened from the problem window.

The dialog is topmost. It starts at 60 seconds and ticks every second:

```text
Convert finished. This computer will shut down in 60 seconds.

Cancel if you want to keep working. Open programs will close.
```

Log `Convert finished. Shutting down in 60 seconds unless you cancel.` Buttons: Shut down now (red), Cancel. Escape cancels. When the count hits zero, treat it as Shut down now.

Cancel logs `Shutdown cancelled. The computer stays on.` and sets status `Finished. Shutdown cancelled.`

Shut down now, or the timer ending, runs:

```bash
osascript -e 'tell application "System Events" to shut down'
```

Log `Shutting down the computer.` and set status `Shutting down...`. If AppleScript fails, log that the computer will stay on. Do not call `sudo`. Do not use the Linux `shutdown -h` syntax. The self-test must not open this dialog or run AppleScript.

## Activity log

The log is colored text, not a plain single-color box.

| Kind | Color |
| --- | --- |
| text | `#e5e7eb` |
| ok | `#86efac` |
| warn | `#fcd34d` |
| err | `#fca5a5` |
| dim | `#9ca3af` |

Append a line only when the view is already near the bottom. If the user has scrolled up, leave their place. When the text grows past 180000 characters, delete about the first 60000. Do not beep when the log updates. Avoid a caret move that rings the system beep.

During an encode, the last line is `Working  |`, then `/`, `-`, and `\`, rotating about every 320 milliseconds. Replace that line in place. Remove it before writing a real log line, then put it back if the encode is still running. A bug report drops a trailing `Working  |` line.

## Settings and logs

Save `app/settings.json` when the folder, checks, budgets, column order, or a remembered prompt changes. Keys:

```json
{
  "folder": "",
  "includeSubfolders": true,
  "autoDelete": false,
  "autoWarnAccepted": false,
  "reencodeAudio": false,
  "detailed": false,
  "budgets": "",
  "shutdown": false,
  "updatePrompt": "",
  "deleteConfirmVersion": "",
  "columnOrder": ["File", "Dimensions", "Length", "Size", "GiB/h", "Budget", "Over", "Target", "New size", "Status", "Space Saved"]
}
```

Defaults: Include subfolders on, the four option checks off, budgets Recommended, column order as listed. `app/skipped.json` is a flat array of `{ "path", "size", "targetKbps" }`. Accept a UTF-8 BOM. If a file is accidentally nested as `[[...]]`, flatten it on read and write it flat.

`app/convert-log.tsv` header:

```text
Timestamp	Status	Reason	Source	New	Kept	OldGiB	NewGiB	OldGiBph	NewGiBph	Width	Height	Kbps
```

Statuses are `KEPT`, `REPLACED`, `SKIP`, `FAIL`, and `UNDONE`. Sizes are GiB with three decimals. Rates are GiB/h with three decimals. The timestamp is local `YYYY-MM-DDTHH:MM:SS`. Reason is one line. Do not attach this log to a public bug report.

## Report a bug

The dialog says the GitHub page is filled with this version and the Mac system name, that Include latest activity adds the latest activity lines and those lines can include folder paths, and that the program does not send the report. The checkbox starts unchecked and is disabled when the activity box is empty. It is not remembered. Buttons: Open report, Cancel.

The only allowed URLs are the Ko-fi page and this form:

`https://github.com/philipvern-dot/video-dehydrator/issues/new?template=bug_report.yml`

Add `version` and `windows`. The form field is named `windows`. Put the Mac system name there anyway, from `sw_vers`, for example `macOS 15.6 (build 24G84)`. When the box is checked, add `activity`. Build that text from the latest lines, dropping a trailing Working line, and keep the URL-encoded length at or under 4500. If one long line does not fit, keep its trailing slice that does fit.

## Updates

On first show, and from Check for updates, GET `https://api.github.com/repos/philipvern-dot/video-dehydrator/releases/latest` with a User-Agent and `Accept: application/vnd.github+json`. Ignore drafts. A tag counts when stripping one leading `v` leaves three or four numeric parts. Compare component by component. `1.0.5` is older than `1.0.5.1`, which is older than `1.0.6`.

If a job is running, say `Finish or cancel the current job before checking for updates.` and do not start the request.

The public files today are Windows installers named `video_dehydrator.exe`. Do not download that file. A Mac update is an asset whose name contains `mac` and ends in `.dmg`, `.zip`, or `.tar.gz`, whose download URL starts with `https://github.com/philipvern-dot/`, and whose published size is greater than zero.

When the tag is newer and a Mac asset exists, status and an ok log line are `Version <version> is available.` Enable Update. The first time that version is seen, a dialog says `Version <version> is available. Click Update to install it.` Closing it saves `updatePrompt` so the same version does not open the dialog again, including from Check for updates. The status line keeps that sentence whenever the app is idle. A later tag shows the dialog again.

When the tag is newer and the only installer is the Windows exe, leave Update disabled, keep `Version <version> is available.` on the status line, and log that the published file is the Windows installer and this Mac copy does not download it. Still show the one-time dialog, with that extra sentence.

When this copy is current and a Mac asset exists, say `You have the latest version (1.0.0).` A missing version number says `The latest release has no version number.` HTTP 404 says `No published update was found.` Any other failure says `Could not reach GitHub to check for updates.` A failed recheck keeps the previous offer and puts its sentence back on the status line.

Update asks before it downloads. `Version <version> is ready to install. This window will close and the installer will open.` Cancel does nothing. The download shows a dialog, `Downloading the update...`, a percent bar, and Cancel. Save the file under the temp directory. Cancel deletes the partial file and logs `Update cancelled.` A failed download logs `The download did not finish.` If the byte size differs from the published size, delete the file and log both sizes: `The download was N bytes. GitHub published M bytes.` On success, open the file with the system opener and close this window.

Do not invent a Mac release. Until a real Mac asset is published, the button stays disabled.

## How this works

A dialog titled `How this works`, with a section list and a page. Opening a named section scrolls to it. Use these sections and this wording.

Opening the program. After setup, open Video Dehydrator from the project script, or run `video-dehydrator` in Terminal. The window is the whole program. FFmpeg and HandBrake live in the `tools` folder. Removing the program folder does not remove videos.

What it does. Point it at a folder. Scan finds videos that are too big for their picture size. Convert makes a smaller copy with the same picture size, the same audio, and the same subtitles. The new file takes the original's place. The original is kept in a folder named `.vd-originals` next to it, until you delete it. Turn on Delete originals automatically to remove each original as soon as its new file checks out.

The size rule. A 1920x1080 video is too big when the whole file is 2 GiB per hour or more. HEVC and AV1 are left alone under 3 GiB per hour. The target for that 1080p picture is 3000 kbps of video. Other picture sizes multiply those numbers by how many pixels they have compared with 1920x1080. 3840x2160 has four times the pixels, so 4K is allowed 8 GiB per hour and is encoded at 12000 kbps. 1280x720 is encoded at 1333 kbps. 720x480 is encoded at 500 kbps. A very small picture stays at 200 kbps. The Budgets area has a choice for 2160p, 1080p, 720p, and 480p. Each choice has its own color, and Dimensions uses that same color. 2160p is blue, 1080p is teal, 720p is purple, and 480p is orange. Each one starts on Recommended. Half, 1.5 times, and Double lower or raise that line, and the encode rate moves with it. A picture of some other size uses the closest of those four, then scales by its number of pixels. HEVC and AV1 stay at one and a half times the budget you picked. The choices are remembered. A higher budget takes files that are no longer over the line off the list. A lower budget finds new files on the next scan. The encode is software x265, two passes, medium speed. The frame rate stays as it was. The picture is not cropped or scaled. Audio that can be copied is copied. Re-encode audio that cannot be copied turns the other kinds, such as Windows Media Audio, into AAC. With that box off, a file whose audio cannot be copied is left as it was. Subtitles are copied and are not burned into the picture. A file whose picture is already at that target rate is left alone, even when the file is large because of the audio. Dolby Vision is left alone.

Scanning. Browse, or drop a folder onto the path, then click Scan. Include subfolders looks inside every folder within the one you chose. The list shows the bloated files, checked and ready to convert. Select all checks every row. Deselect all clears every check. Clear removes the scanned files from the list. Files still waiting in `.vd-originals` stay. The folder list names each folder that has bloated files, and a show is listed along with the season folders inside it. Select folder checks only the bloated files in the folder you picked and clears the other checks. Right-click a row for the same choices. GiB/h is the size of the whole file per hour of runtime. Budget is the line for that picture size. Over is how many times that budget the file uses. 2.0x is twice the line. Slight, shown in green, is under 1.5 times and barely over. Heavy, shown in yellow, is from 1.5 times up to 3 times. Extreme, shown in red, is 3 times the budget or more. Click a heading to sort. The first click on Over puts the worst files first. Drag a heading sideways to move that column. The order is remembered, and an update can put the columns back. The list beside Select folder offers Slight, Heavy, and Extreme. Choosing one checks only that group. Target is the video rate the new file will use. New size is a rough guess. When the current picture rate can be read, the guess keeps the audio and other tracks. When that rate is missing, the guess uses the target picture rate for the length of the video and starts with a tilde. Estimated savings is the current size minus New size for the checked files. A file with no New size figure is left out of the total. Space Saved is the original file size minus the converted file. It fills in when a convert finishes and stays blank before that.

Converting. Convert shrinks every checked file whose status is Bloated, Failed, or Skipped. One file is encoded at a time. The bar is that file. Cancel stops the file that is running. Files that already finished stay finished. The new file is written inside `.vd-originals` first, then checked: it has to be smaller, the same length, the same picture size, with the same audio when that audio can be copied, and at least as many subtitles, and the start and end have to play. When the check passes, the original moves into `.vd-originals` and the new `.mkv` takes its place. When the check fails, the original stays where it was. When the new file is not smaller, the row is marked Skipped and the activity box says why. That file is left unchecked and remembered. Later scans leave it off the list. A smaller budget lists it again. Clear skips forgets those marks. The folder needs free space for the new file while the original is still there. Show detailed activity lists each encode step. The activity box does not add a line for each percent. A moving mark on the last line shows that the file is still being worked on. With that box off, the activity box lists when a file starts and when it finishes. Shutdown computer upon completion turns the computer off after the convert finishes. A one-minute countdown gives you a chance to cancel. If you stop the convert, the computer stays on. A scan does not turn the computer off.

Checking the picture. Select a file marked Ready to delete and click Compare. The window plays both copies side by side: original on the left, new file on the right. Sound is the new file. It will not stay in sync with the picture. Go to the file on disk to confirm the audio. The slider moves through both pictures together. This preview is a smaller picture than the files themselves.

Originals and deleting. Delete originals removes the checked files that are still in `.vd-originals`. The new files stay. The button refuses to delete anything outside that holding folder. If the new file is missing, the original is kept. The program asks before it deletes. Don't show this in the future hides that question until the next version of the program. Undo puts back the last conversion. The originals return to where they were, and the new files are deleted. The program lists the files and asks before it does this. One use covers every file from that convert. It can only undo a conversion whose original is still in `.vd-originals`. A `.plexignore` file tells Plex to skip `.vd-originals` while originals are held there. When that holding folder is removed, those two lines are removed too. If the file also names other things, those lines stay. Held originals are remembered in `app/pending.json`. Each result is also written to `app/convert-log.tsv`.

Auto. Delete originals automatically does the deletion as soon as a file checks out. The check does not judge how the picture looks. The first time Auto is turned on, the program explains that Compare and Undo cannot bring the original back. You can hide that explanation. The program still asks before every convert with Auto turned on. With Auto off, every original stays until you press Delete originals. On a shared drive, the matching copy in `#recycle` is removed when the path and the size match. The `.plexignore` from that convert is removed when it only hides the holding folder. When that removal fails, a window opens with a link to the manual steps.

Drive recycle bin. A shared drive can keep its own copy after a delete. The copy goes in `#recycle` at the top of the share, with the same folders underneath, including `.vd-originals`. That folder is on the drive. It is separate from the Trash on this Mac. After Delete originals, Auto, undo's discarded new file, or a removed temp file, the program looks for that one copy. The file has to sit directly inside `.vd-originals`, and the size has to match. When both match, that copy is deleted. The recycled `.plexignore` is removed only when it contains the two holding lines and nothing else. Empty folders are removed up to, and not including, `#recycle`. A folder that still contains something else is left. Anything else in the bin stays.

To remove a leftover yourself: open the share in Finder. When the share is mounted at `/Volumes/UnifiedMedia`, the bin is `/Volumes/UnifiedMedia/#recycle`. Open `#recycle`, then the same show and season folders, then `.vd-originals`. Delete the original there. A file deleted from inside `#recycle` is removed for good. Leave every file that is outside a `.vd-originals` folder. When `#recycle` is not visible, this Mac account is not allowed to open it. Use the drive's admin site, or SSH.

List the holding folders, changing the mount to this share:

```bash
find '/Volumes/UnifiedMedia/#recycle' -type d -name '.vd-originals' -print
```

Remove only those folders:

```bash
find '/Volumes/UnifiedMedia/#recycle' -type d -name '.vd-originals' -exec rm -rf '{}' +
```

To remove one file:

```bash
rm -f '/Volumes/UnifiedMedia/#recycle/TV/Show/Season 1/.vd-originals/Episode.avi'
```

When the Mac cannot open the bin, sign in to the drive with SSH. Change `/volume1/share` to the volume path:

```bash
find '/volume1/share/#recycle' -type d -name '.vd-originals' -print
find '/volume1/share/#recycle' -type d -name '.vd-originals' -exec rm -rf '{}' +
```

Keep an SSH path in single quotes. A delete run on the drive itself leaves the file out of `#recycle`.

Updates. The bottom right corner shows this copy's version. The program checks for a newer published release when it opens. Check for updates looks again. When a newer version is found, a window tells you once. Closing that window keeps it from opening again until a later version is published. The status line keeps saying that version is available. The published file today is the Windows installer. This Mac copy does not download that file. Update downloads a Mac installer when one is published, then this window closes. Finish or cancel the current job before you update. Donate opens a tip page. The program stays free. A tip of $5 or more can include a feature suggestion on that page. Report a bug opens a GitHub page with this version and the Mac system name filled in. Include latest activity is a choice on that step. Leave it unchecked and nothing else is added. This program does not send the report.

What is in this folder. `video-dehydrator` starts the program. `app` holds the program, settings, the list of originals still waiting, and the convert log. `tools` holds HandBrake, which encodes, and FFmpeg, which reads the files and plays the comparison. FFmpeg is free software under the GPL. Its license is `tools/FFmpeg-LICENSE.txt`. HandBrake is free software under the GPL.

## Self-test

`python -m video_dehydrator --self-test` prints `SELF-TEST OK` and exits 0, or prints the failed check and exits 1. It uses fake probe data. It does not run HandBrake, does not read the user's movies, does not write settings, and does not shut the computer down.

Assert all of these:

- App version reads `1.0.0` from `version.txt`.
- Recommended targets are 3000, 12000, 1333, and 500 kbps for 1080p, 2160p, 720p, and 480p.
- A 320x240 picture floors at 200 kbps.
- 1080p budgets are 2 GiB/h and 3 GiB/h for HEVC. 2160p file budget is 8 GiB/h.
- Packed `1080=plus` makes the 1080p HEVC budget 4.5 GiB/h and the target 4500 kbps.
- Dolby Vision is `dv` even when the file is huge.
- A file under the budget is `lean`.
- A file over the budget whose video bitrate is within 5 percent of the target and at least 40 percent of the file bitrate is `within`.
- A file over the budget otherwise is `bloated`.
- A reason that is only the two "not smaller" sentences is a size skip. Adding any other reason is a real failure.
- The size-skip note tells the user that later scans leave the file off the list.
- A remembered skip matches on path and size when the new target is equal or higher, and does not match when the target is lower.
- Pending written then read is a flat list of objects, including `when` and `batch`.
- A nested `[[{...}]]` pending file is read as one object and written back flat.
- Version comparison orders `1.0.5`, `1.0.5.1`, and `1.0.6`.
- Space saved is old minus new when the original is larger, and blank when either size is missing or the new file is not smaller.
- A column order is kept only when it names each current heading once.
- A recycle plan for an `apfs` path is absent.
- A recycle plan for an `smbfs` mount at `/Volumes/UnifiedMedia` and a file `.../Show/.vd-originals/Episode.avi` points at `/Volumes/UnifiedMedia/#recycle/.../Show/.vd-originals/Episode.avi`.
- That plan is absent when the parent folder is not `.vd-originals`, when the path contains `..`, or when the relative path already contains `#recycle`. A size mismatch does not delete.
- Owned `.plexignore` text is only the two holding lines. An extra line is not owned.
- The HandBrake argument list for the 1080p example contains `x265`, `medium`, `--vb`, `3000`, `--multi-pass`, `--audio-fallback`, `none`, and the copy mask. It does not contain `videotoolbox` or `nvenc`.
- A display path under the scan folder drops the scan-folder prefix.

## Before you stop

1. `python -m video_dehydrator --self-test` prints `SELF-TEST OK`.
2. `file tools/ffmpeg tools/ffprobe tools/ffplay tools/HandBrakeCLI` shows Mach-O for this Mac's chip.
3. The window opens with the icon, the dark header, the eleven columns, the options card, and the three welcome lines.
4. How this works opens on Drive recycle bin from a direct call, and the Finder steps are the Mac paths.
5. Dropping a folder fills the path box. A heading click sorts. A heading drag saves `columnOrder` and does not also flip the sort.
6. If `tools` is incomplete, an error dialog names the missing binaries and Scan stays disabled.
7. Do not encode a library file as part of this first build.

## Leave for later

A `.app` bundle, a `.dmg`, code signing, notarization, and uploading a Mac file to GitHub. The updater already refuses `video_dehydrator.exe` and waits for a real Mac asset.
