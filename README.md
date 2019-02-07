# Container wrapper for [donmelton/video_transcoding](https://github.com/donmelton/video_transcoding)

This script is based on https://hub.docker.com/r/ntodd/video-transcoding but using buildah commands and a fedora based image.

To use it you need to install [buildah](https://github.com/containers/buildah) and [podman](https://github.com/containers/libpod). :warning: On Centos or RHEL, rootless mode is not working as the newuidmap/newgidmap tools are not available there so there is no way to set up the user namespace.


## Create the image
```
$ ./Podmanfile

$ podman images
REPOSITORY                                           TAG      IMAGE ID       CREATED         SIZE
localhost/video_transcoding                          latest   3ea7517badf2   3 minutes ago   424MB
```

On a NUC with 16GB of memory and an Intel Core i5-6260U @1.80GHz (4 cores), image was created in 32 minutes.

## Runing the image
By default running the image will launch video_transcoding binary, but you can overwrite it to use detect-crop or convert-video

```
$ podman run --rm -it video_transcoding --help
Transcode video file or disc image directory into format and size similar to
popular online downloads. Works best with Blu-ray or DVD rip.

Automatically determines target video bitrate, number of audio tracks, etc.
WITHOUT ANY command line options.

Usage: /usr/local/bin/transcode-video [OPTION]... [FILE|DIRECTORY]...

Input options:
    --scan          list title(s) and tracks in video media and exit

(...)

Other options:
-h, --help          display this help and exit
    --version       output version information and exit

Requires `HandBrakeCLI`, `mp4track`, `ffmpeg` and `mkvpropedit`.

$ ls -hl 
-rw-r--r--. 1 1001 1001 5.8G Oct  7 18:54 original_movie.mkv
$ podman run --rm -it -v $PWD:/data:Z --entrypoint detect-crop video_transcoding original_movie.mkv

mpv --no-audio --vf "lavfi=[drawbox=0:2:1920:1036:invert:1]" original_movie.mkv
mpv --no-audio --vf crop=1920:1036:0:2 original_movie.mkv

transcode-video --crop 2:2:0:0 original_movie.mkv

$ podman run --rm -it -v $PWD:/data:Z video_transcoding --crop 2:2:0:0 --main-audio eng --add-audio fra --audio-width all=surround --add-subtitle eng,fra original_movie.mkv -o target_movie.mkv
Cannot load libnvidia-encode.so.1
Cannot load libnvidia-encode.so.1
[17:44:02] hb_init: starting libhb thread
[17:44:02] thread 7ffab6468700 started ("libhb")
HandBrake 20181006123132-4906e78-master (2018100701) - Linux x86_64 - https://handbrake.fr
4 CPUs detected
Opening original_movie.mkv...
[17:44:02] CPU: Intel(R) Core(TM) i5-6260U CPU @ 1.80GHz
[17:44:02]  - Intel microarchitecture Skylake
[17:44:02]  - logical processor count: 4
[17:44:02] hb_scan: path=original_movie.mkv, title_index=1

(...)

[20:37:42] mux: track 0, 140099 frames, 4351208534 bytes, 5957.16 kbps, fifo 512
[20:37:42] mux: track 1, 182603 frames, 467463680 bytes, 640.00 kbps, fifo 1024
[20:37:42] mux: track 2, 182491 frames, 280306176 bytes, 383.76 kbps, fifo 1024
[20:37:42] mux: track 3, 3 frames, 87 bytes, 0.00 kbps, fifo 8
[20:37:42] mux: track 4, 1045 frames, 53599 bytes, 0.07 kbps, fifo 16
[20:37:43] libhb: work result = 0

Encode done!
HandBrake has exited.

Elapsed time: 02:53:41

$ ls -hl
-rw-r--r--. 1 1001 1001 5.8G Oct  7 18:54 original_movie.mkv
-rw-r--r--. 1 root   root   4.8G Oct  7 22:37 target_movie.mkv
```

## Usefull links
* [Compile FFmpeg](https://trac.ffmpeg.org/wiki/CompilationGuide/Centos)
* [Installing dependencies on Fedora](https://handbrake.fr/docs/en/latest/developer/install-dependencies-fedora.html)
