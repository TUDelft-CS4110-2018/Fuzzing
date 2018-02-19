# Software Reversing Lab 1 - Fuzzing and Analysis

You are asked to fuzz two existing applications using AFL. This can be a piece of code one of you developed, any downloaded binary (check CVE's https://cve.mitre.org/ for common vulnerabilities in (older) binaries), an open source project, or a CTF (check https://writeups.easyctf.com/). In addition to trying to make it crash, you should investigate and explain why it crashes, e.g., using a debugger such as GDB, OllyDBG, or LLDB.

In the case of available source code:
   *	Provide the source code that causes the crash and determine how to avoid it.

In the case of a binary:
   *	Provide the stack content at time the crash, and discuss why it happens.

You may use any tools available such as IDA, Radare, Valgrind, decompilers, etc. Use whatever software you are familiar with.

If you cannot find any crash, please find two other software tools that do crash (see the AFL website or the CVEs for possibilities). One popular tool to crash is ImageMagick, check earlier versions if necessary but find a different bug than the one shown in last year's video. Do not waste too much time by fuzzing applications that are very unlikely to contain simple bugs. Nowadays, many developers test their software using fuzzing tools such as AFL.

You have to create a small report of up to 4 pages. Clearly demonstrate how to implement the required steps, what results are obtained, and their meaning. Optionally, you may include a short description of the required tools that need to be downloaded and how to set them up, preferably in an .md file format (not part of the report). Please indicate whether you allow us to share your experiment and results with your fellow students.

