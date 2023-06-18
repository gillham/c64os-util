# c64os-util

this is a collection of utilities for parsing and generating various file types
related to [C64 OS](https://c64os.com/).

the main feature (currently) is support for C64 archive files, but there is also support for generating application menu and metadata definition files.

special thank you to gillham for sharing [`uncar.py`](https://github.com/gillham/C64/blob/main/C64OS/uncar/uncar.py) which showed how the archives are formatted.

> note: this is an early prototype of a work in progress. some features described in this readme are not functional yet. some APIs may change.

## installation

    pip install c64os-util

## usage (in code)

import the module:

    import c64os_util as c64os

open an archive:

    path = os.path.join('path', 'to', 'my', 'archive.car')

    # you can deserialize from a buffer:
    with open(path, 'rb') as f:
        archive = c64os.CbmArchive.deserialize(f)

    # or use the helper function (equivalent to above):
    archive = load_car(path)

    # or create an empty one from scratch:
    archive = c64os.CbmArchive(
        archive_type=CarArchiveType.INSTALL,
        note='hello world!',
    )

you can create and remove directories and files:

    path = os.path.join('path', 'inside', 'archive')
    archive.mkdir(path, create_missing=True)
    path = os.path.join('path', 'to', 'data.t')
    archive.touch(
        path, file_type=c64os.CarRecordType.SEQFILE,
        create_directories=True,
    )
    path = os.path.join('path', 'inside')
    archive.rm(path, recursive=True)

you can iterate through all the files:

    for file_record in archive:
        print(file_record.path())

and you can directly open any file for reading or writing:

    # invoking the archive returns a context manager
    # the API is the same as python's 'open' function
    # except when creating files you can specify record_type
    # (to distinguish between PRG and SEQ files)

    # for text files, you can specify encodings from cbmcodecs2
    path = os.path.join('path', 'to', 'data.t')
    with archive(path, 'w', encoding='petscii_c64en_lc') as f:
        print('hello world', end='\r', file=f)

    # you can copy files into the archive like this
    path = os.path.join('path', 'to', 'program.prg') # path in archive
    h_path = os.path.join('out', 'program.prg') # path on host filesystem
    t = CarRecordType.PRGFILE
    with archive(path, 'wb', record_type=t) as f1, open(h_path, 'rb') as f2:
        data = f2.read()
        f1.write(data)

when you're finished, you can save the full archive to a file:

    path = os.path.join('path', 'to', 'my', 'archive.car')

    # by serializing to a buffer:
    with open(path, 'wb') as f:
        archive.serialize(f)

    # or using the helper function (equivalent to above):
    save_car(archive, path)

## usage (from cli)

the package will provide a script called `car`. this script's cli flags and behavior will be designed to mimic `tar`, to facilitate familiarity.

one notable change we'll have to make is for dealing with cbm file types. with `tar`, it's easy to say "compress this whole folder" -- but with `car`, we need to know whether each file is a SEQ or PRG file.

this makes the command line a bit messy. we might do what VICE does, which is append the source path with "!S" or "!P". but that still means listing each file individually instead of a whole directory (unless you're fine with every file in that directory being all the same type).

## contributing

i would love help. i'm in the C64 OS discord.

## todo (short-term)

* [ ] get archive support working
* [ ] menus: support separators
* [ ] get cli scripts implemented
* [ ] publish on test-pypi
* [ ] publish on pypi
* [ ] incorporate into example-app

## todo (longer-term)

* [ ] document APIs
* [ ] more rigorous error handling
* [ ] write higher-quality tests
* [ ] set up formatters and linters
* [ ] publish documentation
* [ ] set up contributing guidelines
* [ ] pydantic?
