# dmt-ffmpeg

ffmpeg for [dmt](https://github.com/imjyotiraditya/dmt), with a dolby ac-4
decoder bolted on.

one branch, `main`: upstream ffmpeg plus one commit. it sits on `release/9.0`
right now. when upstream moves i rebase, rather than keeping a branch per
version.

## the decoder isn't mine

paul b mahol wrote it in 2019 and still maintains it in librempeg
(https://github.com/librempeg/librempeg), his own fork of ffmpeg. mainline
ffmpeg has an ac-4 demuxer and a codec id but nothing to decode with, so
`ffmpeg` just says `no decoder found for: ac4`.

it went to ffmpeg-devel once, in 2020, marked `[RFC][WIP]`. review caught a
missing `kbdwin.o`, fate failed, and it was never resubmitted. it's lived in
the fork ever since.

two things needed changing to build it here:

- `FF_KBD_WINDOW_MAX` 1024 -> 2048. ac-4 uses 2048 point kbd windows and
  mainline asserts on anything bigger, so it dies on the first frame.
- a local copy of `vector_fmul_reverse_c`. librempeg exports it, mainline
  keeps it `static`. don't be tempted to drop the scalar branch and always
  call the vtable - `float_dsp.h` says `len` must be a multiple of 16, and the
  simd version will happily run off the end of the buffer if it isn't.

## licence

librempeg is gplv3, ffmpeg is lgpl-2.1+. `ac4dec.c`, `ac4dec_data.h` and
`ac4_parser.c` carry gplv3 headers, so anything built with them is gplv3.

they're gated in configure:

    ac4_decoder_deps="gpl version3"

no `--enable-gpl --enable-version3`, no decoder, and the tree stays honestly
lgpl. everything else here is untouched upstream ffmpeg.

also worth remembering ac-4 is still a live dolby patent. fine on your own
phone; think twice before handing builds to other people.

## keeping up with upstream

    git remote add upstream https://github.com/FFmpeg/FFmpeg.git
    git fetch upstream release/9.0
    git rebase upstream/release/9.0
