# AFLCombined: An Integration of AFLChurn (Regression Greybox Fuzzing) into American Fuzzy Lop plus plus (AFL++)

<img align="right" src="https://raw.githubusercontent.com/AFLplusplus/Website/master/static/aflpp_bg.svg" alt="AFL++ logo" width="250" heigh="250">

Release version: [4.04c](https://github.com/AFLplusplus/AFLplusplus/releases)

GitHub version: 4.05a

Original Repository:
[https://github.com/AFLplusplus/AFLplusplus](https://github.com/AFLplusplus/AFLplusplus)

AFLCombined was developed by:

* Pete Fyffe <pfyffe@iu.edu>

AFLChurn was developed by:

* Xiaogang Zhu <xiaogangzhu@swin.edu.au>
* Marcel Böhme <marcel.boehme@acm.org>

AFL++ is maintained by:

* Marc "van Hauser" Heuse <mh@mh-sec.de>
* Heiko "hexcoder-" Eißfeldt <heiko.eissfeldt@hexco.de>
* Andrea Fioraldi <andreafioraldi@gmail.com>
* Dominik Maier <mail@dmnk.co>
* Documentation: Jana Aydinbas <jana.aydinbas@gmail.com>

Originally developed by Michał "lcamtuf" Zalewski.

AFL++ is a superior fork to Google's AFL - more speed, more and better
mutations, more and better instrumentation, custom module support, etc.

You are free to copy, modify, and distribute AFL++ with attribution under the
terms of the Apache-2.0 License. See the [LICENSE](LICENSE) for details.

## Getting started

Here is some information to get you started:

* For an overview of the AFL++ documentation and a very helpful graphical guide,
  please visit [docs/README.md](docs/README.md).
* To get you started with tutorials, go to
  [docs/tutorials.md](docs/tutorials.md).
* For releases, see the
  [Releases tab](https://github.com/AFLplusplus/AFLplusplus/releases) and
  [branches](#branches). The best branches to use are, however, `stable` or
  `dev` - depending on your risk appetite. Also take a look at the list of
  [important changes in AFL++](docs/important_changes.md) and the list of
  [features](docs/features.md).
* If you want to use AFL++ for your academic work, check the
  [papers page](https://aflplus.plus/papers/) on the website.
* To cite our work, look at the [Cite](#cite) section.
* For comparisons, use the fuzzbench `aflplusplus` setup, or use
  `afl-clang-fast` with `AFL_LLVM_CMPLOG=1`. You can find the `aflplusplus`
  default configuration on Google's
  [fuzzbench](https://github.com/google/fuzzbench/tree/master/fuzzers/aflplusplus).

## Building and installing AFLCombined

To have AFLCombined easily available with everything compiled, pull the AFL++ image directly
from the Docker Hub (available for both x86_64 and arm64):

```shell
docker pull aflplusplus/aflplusplus
docker run -ti -v /location/of/your/target:/src aflplusplus/aflplusplus
```

This image is automatically published when a push to the stable branch happens
(see [branches](#branches)). If you use the command above, you will find your
target source code in `/src` in the container.

Enter the container with command line:
```shell
docker exec -it aflplusplus /bin/bash
```

Then, pull the AFLCombined repository onto the image.
```
cd /
git clone https://github.com/fyffep/AFLplusplus-Contiguity.git
```

Compile the program with `make`. More details at [docs/INSTALL.md](docs/INSTALL.md).
```
make distrib
```

## Quick start: Fuzzing with AFLCombined

*NOTE: Before you start, please read about the
[common sense risks of fuzzing](docs/fuzzing_in_depth.md#0-common-sense-risks).*

This is a quick start for fuzzing targets with the source code available. To
read about the process in detail, see
[docs/fuzzing_in_depth.md](docs/fuzzing_in_depth.md).

To learn about fuzzing other targets, see:
* Binary-only targets:
  [docs/fuzzing_binary-only_targets.md](docs/fuzzing_binary-only_targets.md)
* Network services:
  [docs/best_practices.md#fuzzing-a-network-service](docs/best_practices.md#fuzzing-a-network-service)
* GUI programs:
  [docs/best_practices.md#fuzzing-a-gui-program](docs/best_practices.md#fuzzing-a-gui-program)

Step-by-step quick start:

1. Compile the program or library to be fuzzed using `afl-cc`. A common way to
   do this would be:

   ```
   CC=/path/to/afl-cc CXX=/path/to/afl-c++ ./configure --disable-shared
   make clean all
   ```

2. Get a small but valid input file that makes sense to the program. These files may be provided by the program's
   authors or, if no files are available, using garbage data with `cp /bin/ps* /path/to/fuzz/input` 
   is sufficent. When fuzzing verbose syntax (SQL, HTTP, etc.), create a dictionary as described in
   [dictionaries/README.md](dictionaries/README.md), too.

3. If the program reads from stdin, run `afl-fuzz` like so:

   ```
   ./afl-fuzz -i seeds_dir -o output_dir -- \
   /path/to/tested/program [...program's cmdline...]
   ```

   To add a dictionary, add `-x /path/to/dictionary.txt` to afl-fuzz.

   If the program takes input from a file, you can put `@@` in the program's
   command line; AFL++ will put an auto-generated file name in there for you.

4. Investigate anything shown in red in the fuzzer UI by promptly consulting
   [docs/afl-fuzz_approach.md#understanding-the-status-screen](docs/afl-fuzz_approach.md#understanding-the-status-screen).

5. You will find found crashes and hangs in the subdirectories `crashes/` and
   `hangs/` in the `-o output_dir` directory. You can replay the crashes by
   feeding them to the target, e.g. if your target is using stdin:

   ```
   cat output_dir/crashes/id:000000,* | /path/to/tested/program [...program's cmdline...]
   ```

   You can generate cores or use gdb directly to follow up the crashes.

6. We cannot stress this enough - if you want to fuzz effectively, read the
   [docs/fuzzing_in_depth.md](docs/fuzzing_in_depth.md) document!

## Contact

Questions? Concerns? Bug reports?

* The AFL++ contributors can be reached via (e.g., by creating an issue):
  [https://github.com/AFLplusplus/AFLplusplus](https://github.com/AFLplusplus/AFLplusplus).
* Take a look at [FAQ](docs/FAQ.md). If you find an interesting or important
  question missing, submit it via
  [https://github.com/AFLplusplus/AFLplusplus/discussions](https://github.com/AFLplusplus/AFLplusplus/discussions).
* Best: join the [Awesome Fuzzing](https://discord.gg/gCraWct) Discord server.
* There is a (not really used) mailing list for the AFL/AFL++ project
  ([browse archive](https://groups.google.com/group/afl-users)). To compare
  notes with other users or to get notified about major new features, send an
  email to <afl-users+subscribe@googlegroups.com>, but note that this is not
  managed by us.

<details>

  <summary>List of contributors</summary>

  ```
    Jann Horn                             Hanno Boeck
    Felix Groebert                        Jakub Wilk
    Richard W. M. Jones                   Alexander Cherepanov
    Tom Ritter                            Hovik Manucharyan
    Sebastian Roschke                     Eberhard Mattes
    Padraig Brady                         Ben Laurie
    @dronesec                             Luca Barbato
    Tobias Ospelt                         Thomas Jarosch
    Martin Carpenter                      Mudge Zatko
    Joe Zbiciak                           Ryan Govostes
    Michael Rash                          William Robinet
    Jonathan Gray                         Filipe Cabecinhas
    Nico Weber                            Jodie Cunningham
    Andrew Griffiths                      Parker Thompson
    Jonathan Neuschaefer                  Tyler Nighswander
    Ben Nagy                              Samir Aguiar
    Aidan Thornton                        Aleksandar Nikolich
    Sam Hakim                             Laszlo Szekeres
    David A. Wheeler                      Turo Lamminen
    Andreas Stieger                       Richard Godbee
    Louis Dassy                           teor2345
    Alex Moneger                          Dmitry Vyukov
    Keegan McAllister                     Kostya Serebryany
    Richo Healey                          Martijn Bogaard
    rc0r                                  Jonathan Foote
    Christian Holler                      Dominique Pelle
    Jacek Wielemborek                     Leo Barnes
    Jeremy Barnes                         Jeff Trull
    Guillaume Endignoux                   ilovezfs
    Daniel Godas-Lopez                    Franjo Ivancic
    Austin Seipp                          Daniel Komaromy
    Daniel Binderman                      Jonathan Metzman
    Vegard Nossum                         Jan Kneschke
    Kurt Roeckx                           Marcel Boehme
    Van-Thuan Pham                        Abhik Roychoudhury
    Joshua J. Drake                       Toby Hutton
    Rene Freingruber                      Sergey Davidoff
    Sami Liedes                           Craig Young
    Andrzej Jackowski                     Daniel Hodson
    Nathan Voss                           Dominik Maier
    Andrea Biondo                         Vincent Le Garrec
    Khaled Yakdan                         Kuang-che Wu
    Josephine Calliotte                   Konrad Welc
    Thomas Rooijakkers                    David Carlier
    Ruben ten Hove                        Joey Jiao
    fuzzah                                @intrigus-lgtm
  ```

</details>