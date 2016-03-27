curlbash: locally save and checksum/review before curl|bash-ing

# curlbash

Installing things by curling them to a shell probably isn't a great
practice security-wise, but several things can make it a little better.
This script caches the curl'd installer, can optionally verify a
checksum, and give the user a chance to review what was downloaded
before executing it. Consider it harm reduction for the command line.

Despite the name, curlbash is targeted at `/bin/sh` for portability
reasons. A reasonable effort is made to test it with bash, mksh, pdksh,
dash, and other common shells that attempt to be POSIX-compliant /
compatible with the Bourne shell. (In other words, bash-isms are bugs.)

curlbash is distributed under the ISC license.


# Usage

    Usage: curlbash [-c] [-e SHA256] [-f | -r] [-h] [-v] URL
      -c           ignore cache
      -e SHA256    expect this SHA256, or exit
      -f           fetch and exit
      -h           print this help and exit
      -r           run, without confirmation
      -v           increase verbosity
