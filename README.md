TVBOX 2.03 — “Live TV” Scheduler + MPV Tuner 
Now with optional Guide (EPG) and RSS-based News/Weather channels

TVBOX is a “live TV” style player built on MPV plus a pre-generated schedule.
Drop shows into a predictable folder layout, build an index (durations + labels),
generate 7 days of schedules, then run tvbox.py to “tune” channels like a TV.

In addition to standard channels, TVBOX can now generate synthetic broadcast channels
(e.g. Program Guide, News & Weather) from metadata and external feeds.

This repo is structured as a portable bundle: Python, MPV, ffprobe, and
AutoHotkey, wkhtml are expected under /runtime/ (placeholders included)

If you have a remote control, there is an available hotkey file and launcher .BAT (search Amazon for Remote Control with USB Infrared Receiver from SUNGOOYUE for a matching remote)

*** It's built under Windows 11 but should be possible to port to other OS ***

--------------------------------------------------
WHAT’S INCLUDED
--------------------------------------------------

Core scripts
- media_index.py
  Builds/refreshes state/media_index.json (durations, labels, signatures)

- scheduler.py
  Generates schedule JSONs and readable CSVs for today + next 6 days

- tvbox.py
  MPV tuner that plays today’s lineup and supports channel switching

- tvbox_watchdog.py
  Monitors MPV playback and forces a safe reload if playback appears stuck

- scan_videos.py
  Scans videos for integrity and outputs bad files to the quarantine folder

- empty_check.py
  Finds empty timeslot folders (helpful if you have multiple channels and need to keep track)

- guidegen.py
  Builds a Program Guide (EPG) channel from schedule data

- guide_html_gen.py / guide_video_gen.py
  Render guide pages (HTML → image → video)

- rssgen.py
  Fetches and caches RSS feeds (news, weather, etc.)

- rss_video_gen.py
  Renders RSS pages into broadcast-safe video segments

- mpg_to_mpv_cleanup.py
  Seeks out mpeg video files and converts them to MKV (mpeg is not supported due to playback and timing offset limitations)

- fix_mkv_missing_duration.py
  Seeks out mkv files with missing duration (a byproduct of some file conversions) and fixes them. (necessary to actually "schedule" a file)

BAT wrappers
- index.bat               -> media_index.py
- scheduler.bat           -> scheduler.py (schedules only specific or missing <7 days)
- schedule_refresh.bat    -> scheduler.py --refresh-all (rewrites the next 7 days)
- tvbox.bat               -> tvbox_watchdog.py 
- tvbox_with_remote.bat   -> watchdog + AutoHotkey remote + keep-alive
- scan_videos.bat         -> scan_videos.py
- empty_check.bat         -> empty_check.py
- remote_listen.bat       -> remote_listen.ahk
- guidegen.bat            -> generate Guide (EPG) channel
- rssgen.bat              -> generate RSS / News channel (feeds edited here)
- mpg_to_mpv_cleanup.bat  -> mpg_to_mpv_cleanup.py
- fix_mkv_missing_duration.bat -> fix_mkv_missing_duration.py


Optional AutoHotkey
- remote.ahk
- remote_listen.ahk (to identify remote key-presses to map to TVBox functions)
- keep_alive.ahk (current band-aid to avoid time-outs/no-files)

--------------------------------------------------
ADDITIONAL FILES / BINARIES NEEDED
--------------------------------------------------

root/runtime/
  python/ (download and extract python-3.5.0b1-embed-amd64.zip to this folder)
  ffmpeg/bin/ (download and extract ffprobe.exe here)
  ffmpeg/bin/ (download and extract ffmpeg.exe here)
  mpv/ (download and extract mpv-x86_64-20251214-git-f7be2ee.7z here)
  mpv/portable_config 
	input.conf (retain the version provided in this repo)
	mpv.conf (retain the version provided in this repo)
  ahk/ (install AutoHotkey_2.0.19_setup.exe, then copy AutoHotkey64.exe to this folder)
  wkhtml/ (install wkhtmltoimage files here) 

--------------------------------------------------
QUICK START
--------------------------------------------------

1) Drop required files into runtime/
2) Add channels and videos (example files are in the Channel and Video folders)
3) Run index.bat (to build the indexed video library)
4) Run scheduler.bat (to build the programming schedule)
5) Run tvbox.bat or tvbox_with_remote.bat (to start watching)

--------------------------------------------------
CHANNEL SETUP
--------------------------------------------------

Channels live under the channels/ directory.
Each channel must be in its own folder, prefixed with a two-digit number
to define channel order.

Example:

channels/
  01 Comedy/
  02 Drama/
  10 Talk/

Channel folder contents:

channels/01 Comedy/
  title.txt        (optional to label the channel on the guide and on-screen display)
  slots.json       (required if you have videos in structured time-based slot folders)
  video/

title.txt
- Plain text file
- Overrides the channel name shown in the on-screen display (OSD)
- If missing, the folder name is used

--------------------------------------------------
VIDEO FOLDER STRUCTURE
--------------------------------------------------

Videos must live under:

channels/<Channel Name>/video/<Time-based Folder>/<Program Name>

Inside video/, you may define any number of time-based folders.
Folder names are freeform but must match what is referenced in slots.json. 

Example:

channels/01 Comedy/video/
  morning/
  daytime/
  evening/
  latenight/

Each time folder may contain show subfolders, episodes directly,
or deeper nesting. Discovery is recursive.

Example:

channels/01 Comedy/video/evening/
  Sesame Street/
    S01E01.mp4
    S01E02.mp4
  Felix the Cat/
    S03E10.mkv

Time folder subfolders show up as the 'on-screen' display as the current program

--------------------------------------------------
SLOTS.JSON CONFIGURATION
--------------------------------------------------

slots.json defines which video folder is active at which time of day.
It is used by both the scheduler and tvbox fallback playback.

Example slots.json:

{
  "default_folder": "evening",
  "slots": [
    {"name": "overnight", "start": "00:00", "end": "06:00", "folder": "overnight"},
    {"name": "morning",   "start": "06:00", "end": "09:00", "folder": "morning"},
    {"name": "daytime",   "start": "09:00", "end": "15:00", "folder": "daytime"},
    {"name": "afternoon", "start": "15:00", "end": "18:00", "folder": "afternoon"},
    {"name": "evening",   "start": "18:00", "end": "21:00", "folder": "evening"},
    {"name": "latenight", "start": "21:00", "end": "24:00", "folder": "latenight"}
  ]
}

Rules:
- Times use local machine time
- 24:00 is allowed as an end boundary
- Cross-midnight ranges are supported
- Folder names must exactly match folders under video/
- Must be valid JSON (no trailing commas)

If slots.json is missing or invalid:
- Scheduler may skip the channel
- tvbox.py will fall back to default_folder if possible

--------------------------------------------------
SETTING UP THE FEED FOR CHANNEL 99 TBN
--------------------------------------------------
- Find your favorite RSS or ATOM based feeds and make note of the URLs
- In your text-editor, open rssgen.bat, and locate the "call runtime\python\python.exe rssgen.py" commands. Some example feeds have been added for reference
- Replace the RSS or XML links provided with your desired feeds
- Add the output label name (this is key for the video generator to pick-up the right output file, and must start with 'rss-')
- Note that it's sometimes particular about HTTPS urls. If you have an issue, try just HTTP instead.
- You can add or remove feeds. The channel loops every 60 seconds, so any feeds will provided even fractions of time. I.e. 4 feeds = 15 seconds each feed.
Syntax below:
call runtime\python\python.exe rssgen.py "{XML or RSS url}" "rss-{output label name}" "{Feed title}"

--------------------------------------------------
CONTROLS
--------------------------------------------------
Esc		-> Reload
Q		-> Exit
Up Arrow / + 	-> Next Channel
Down Arrow / -	-> Prev Channel
Number Keys	-> Specific Channel (ex: Press 1 for Channel 1, 12 for Channel 12)
Vol Up		-> Increase Volume by 5%
Vol Down	-> Decrease Volume by 5%
Mute		-> Mute

--------------------------------------------------
KNOWN ISSUES
--------------------------------------------------

- The tvbox.py occasionally does not send the next video, resulting in 'No File' via MPV. It points to a pipe or write issue, that cannot yet be resolves. Two band-aids are added to address: scheduled 'channel change' every 30 minutes, triggered by the keep_alive.ahk AutoHotkey script, and a parent 'watchdog' that monitors MPV for continuous playback, then reloads if it fails. If you can find of a better solution, let me know!

--------------------------------------------------
VERSION CHANGES
--------------------------------------------------
2026-02-01
- Added an electronic program guide channel a la 1995
- Added a customizable RSS / ATOM feed channel 
- Improved video cropping to match a 4:3 orientation
- Locked in programs to the nearest 30-minute block with ~7 minute interval commercial break injections, to give it a more true to 'broadcast tv' feel
- Added more tools to find and correct unplayable videos
- Added an assets folder and reconsolidated existing asset files 
