---
title: 'A Better Packed File-System'
author: marco.lizza
layout: post
permalink: "/a-better-packed-file-system"
comments: true
categories:
  - tofu-engine
  - devlog
tags:
  - i/o
  - internals
---
I already [talked about it in the past](/tofu-engine-5) about Tofu Engine's custom *packed format*.

After some time, I evolved that initial implementation to a directory-based one. At the end of the archive, there is a *directory header* structure telling the total amount of entries in the archive and the position where the directory resides. There, by reading a sequence of *directory entries* the directory could be processed, or loaded into memory with a single I/O burst.

The problem is the implementation was... dumb. :) I aimed to exploit it to perform seeks straight from the on-disk directory (rather than from memory), or to load it into memory efficiently. However, since every *directory entry* was variable in length (as the name of the entry was stored as a variable-length string) a straight-from-the-file binary search could not be performed. Also, even loading the directory into memory wasn't performed in a single burst.

Back to square one, then, and discuss each issue one by one.

**Q1.** Do we need to store the actual file name (with relative path)?

**A1.** No, not really. We are already calculating the filename digest to (eventually) use it as an encryption key. We could store it straight into the directory index and, at the expense of a constant *directory entry* size, load it into memory in one go!

> Note that need to derive *another* encryption key from the digest. Using as-it-is would be way too naive, as we would be storing the file both the key and the data, side by side! :)

**Q2.** Can we afford to keep in memory the whole directory?

**A2.** Let's do some math.

Each directory entry will be defined as follows

```c
typedef struct Directory_Entry_s {
    uint8_t digest[16]; // Oh! A 128-bit digest! :)
    uint32_t offset;
    uint32_t size;
} Directory_Entry_t;
```

> Please note that we will be using [MD5](https://it.wikipedia.org/wiki/MD5) to compute the file digest. Some engines use CRC32 to represent the distinct entries, which is not safe enough for collisions even with a small set of files. A more complex digest, such as SHA-256, will be suggested for very big games. In our case, we picked MD5 as the sweet spot.

How many single distinct entries are we going to handle? Since I haven't the craziest idea, so I randomly picked two open-source games ([Frogatto](https://github.com/frogatto/frogatto) and [Endless Sky](https://github.com/endless-sky/endless-sky)) that fall into the *mid-sized non-AAA 2D game* category (which the game engine aims to support). Excluding the source code, we have a total of approximately *5000* distinct files. Let's increase it a bit, and say we have 8K files at most.

This results in `8096 * 24 = 194304`, a total of about 190KiB of RAM will be used by the directory. This is not that much and should fit in any current system (especially since the engine is not aimed to be a "fake limited resources platform").

**Q3.** Is there a benefit in storing the *directory* explicitly in the archive file?

**A3.** When a *directory* entry is loaded from the file, being a single one or multiple entries, it can't be used as it is. The serialized data-types conform to some arbitrary format (e.g. the file size is stored using a 32-bit unsigned value), but we could be running on architecture with a different word size[^1] (e.g. 64 bit). This requires an additional post-load marshalling step.

Also, creating the directory into the archive file requires a (minor) additional effort as it needs to be computed and written somewhere with the file (typically, either just after the archive header or at the very end of it)

Alternatively, if we write a directory entry just before a single entry data we need to scan the whole archive during the opening phase to build the directory. This might be a problem with very slow media types, but modern ones are fast enough to make this operation almost instantaneous.

**Q4.** Are searches from a serialized-to-archive directory unmotivated?

**A4.** I wouldn't say that. With a very huge and populated archive (let's say with a million entries) the heap memory occupation for the directory would be significant but not excessive (~24MiB); however, the time to build it during the archive opening would be surely noticeable as the whole `10^6` entries would be scanned one after the other.

Searching from within the on-file directory, on the other hand, requires several seeks but that is something that happens with an ordinary file-system structure. Moreover, the directory seek-time is only a (very) minor fraction of the total time during the loading process; (prematurely?) optimizing that aspect at the expense of memory occupation/thrashing is not the best of the decisions.

**Q5.** Could be worth considering an [IFF-like](https://en.wikipedia.org/wiki/Interchange_File_Format) format for the archive?

**A5.** as we are modelling something like a virtual file-system on file, and not a generic extendable file format; the file is readable, we know what it will contain and we can optimize it. if we need more data/format in the future we will just update the version of it.

---

With all this being said we redesigned the archive format once again, making it more similar to the initial implementation. We can recap the design choices as follows:

* the archive structure is straight a simple: header, directory, payload;
* each file is represented in the archive with the MD5 digest of its name[^2];
* filenames are *case insensitive* (I'm adamant that using case sensitiveness in a file-system is just a recipe for disaster), they are treated as lowercase to avoid confusion/mistakes;
* the directory of the archive is stored at the beginning of the archive, just after the archive header: this makes directory access trivial, as the archive header is fixed in size and we don't need to store this offset somewhere in the archive metadata;
* we are not implementing compression and each file is stored with its native format (which is in turn quite likely an optimally chosen one, e.g. PNG, with the sole exception of the script files which could potentially benefit from a custom compression algorithm);
* we store only the bare-minimum information in the archive meta-data (either for the archive or the directory entries): offsets in the archive and size (no CRCs, date and time marker, flags, permissions, and so on...).

Storing the file offset in each directory entry enables [binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm) as once the correct entry is fetched all the required details are ready to be used. We make this feature optional, however, with a flag into the archive meta-data telling whether the directory is sorted. If not a linear search is used.

This results in a very minimalistic file format[^3], which stores only what is vital for the archive to work.

---

[^1]: There might also be a difference in endian-ness, which is something that is becoming rarer and rarer nowadays, as almost all architectures are *little-endian*. Still, since we are designing the game as multiplatform we should address this. By the way, we stick to *little-endian*.

[^2]: MD5 is known for being cracked/unsafe. While it's true that one could mischievously find collisions for a given digest, under (fair) usage the change for the first clash would happen with `2*10^19` entries. We don't plan to have archives that big, for the moment, so we are good with that. :D

[^3]: Incidentally our format ended in being similar to [Build engine's GRP format](https://moddingwiki.shikadi.net/wiki/GRP_Format), in which file offsets aren't explicitly stored and need to be calculated. However, in `.grp` archives and entry offset (in the archive) is calculated during the scanning process (by accumulating the size of the previous entries).
