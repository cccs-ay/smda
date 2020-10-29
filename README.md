
# SMDA

SMDA is a minimalist recursive disassembler library that is optimized for accurate Control Flow Graph (CFG) recovery from memory dumps.
It is based on [Capstone](http://www.capstone-engine.org/) and currently supports x86/x64 Intel machine code.
As input, arbitrary memory dumps (ideally with known base address) can be processed.
The output is a collection of functions, basic blocks, and instructions with their respective edges between blocks and functions (in/out).
Optionally, references to the Windows API can be inferred by using the ApiScout method.

## Installation

With version 1.2.0, we have finally simplified things by moving to [PyPI](https://pypi.org/project/smda/)!  
So installation now is as easy as:

```
$ pip install smda
```

## Usage

A typical workflow using SMDA could like this:

```
>>> from smda.Disassembler import Disassembler
>>> disassembler = Disassembler()
>>> report = disassembler.disassembleFile("/bin/cat")
>>> print(report)
 0.777s -> (architecture: intel.64bit, base_addr: 0x00000000): 143 functions
>>> for fn in report.getFunctions():
...     print(fn)
...     for ins in fn.getInstructions():
...         print(ins)
...
0x00001720: (->   1,    1->)   3 blocks,    7 instructions.
0x00001720: (      4883ec08) - sub rsp, 8
0x00001724: (488b05bd682000) - mov rax, qword ptr [rip + 0x2068bd]
0x0000172b: (        4885c0) - test rax, rax
0x0000172e: (          7402) - je 0x1732
0x00001730: (          ffd0) - call rax
0x00001732: (      4883c408) - add rsp, 8
0x00001736: (            c3) - ret 
0x00001ad0: (->   1,    4->)   1 blocks,   12 instructions.
[...]
>>> json_report = report.toDict()
``` 

There is also a demo script:

* analyze.py -- example usage: perform disassembly on a file or memory dump and optionally store results in JSON to a given output path.

The code should be fully compatible with Python 2 and 3.
Further explanation on the innerworkings follow in separate publications but will be referenced here.

To take full advantage of SMDA's capabilities, make sure to (optionally) install:
 * lief 
 * pdbparse (currently as fork from https://github.com/VPaulV/pdbparse to support Python3)

## Version History

 * 2020-10-29: v1.4.9 - Resolves 64bit API calls of style `call qword ptr [rip + offset]`.
 * 2020-10-29: v1.4.8 - Bugfixes. Verbose mode added (THX: @jcrussell).
 * 2020-10-28: v1.4.6 - WinApiResolver now tries to resolve import by ordinal to their name if it is known - can be extended in the database of OrdinalHelper.
 * 2020-10-28: v1.4.5 - Store the (mapped) buffer that was used to do disassembly along inside a SmdaReport - goal: enable to read strings/bytes at offsets at a later time.
 * 2020-10-27: v1.4.4 - SmdaInstructions can now provide potential data references via `SmdaInstruction.getDataRefs()`.
 * 2020-10-27: v1.4.3 - SmdaInstructions can now on demand provide the detailed capstone instruction representation via `SmdaInstruction.getDetailed()`.
 * 2020-10-27: v1.4.1 - 10-20% gain in processing speed by switching to `capstone.disasm_lite()`.
 * 2020-10-26: v1.4.0 - Adding SmdaBasicBlock. Some convenience code to ease intgration with capa. (GeekWeek edition!) 
 * 2020-09-07: v1.3.11 - Summarizable DisassemblyStatistics.
 * 2020-09-02: v1.3.10 - Fixed a bug where IDA Pro would crash when failing to demangle a function name while exporting a SMDA report.
 * 2020-08-31: v1.3.9 - Adjusted Logging to avoid interference with other loggers configured outside of SMDA (THX: @BonusPlay).
 * 2020-08-25: v1.3.6 - PicHash no longer stored as list.
 * 2020-08-17: v1.3.5 - Bugfix for import parsing (ELF files).
 * 2020-08-17: v1.3.4 - Recalculate PIC hash and nesting depth for  older (v1.2.x) reports on import for compatibility.
 * 2020-08-17: v1.3.3 - Added binary variation of `push ebp;mov ebp, esp` to list of default prologues and added exception handling for DominatorTrees (THX: @fxb).
 * 2020-07-13: v1.3.2 - Use LIEF to parse Import Table for WinAPI usage data when processing unmapped files.
 * 2020-07-13: v1.3.1 - Fixed `setup.py` to properly specify dependencies (THX: @BonusPlay).
 * 2020-06-22: v1.3.0 - Added DominatorTree (Implementation by Armin Rigo) to calculate function nesting depth, shortened PIC hash to 8 byte, added some missing instructions for the InstructionEscaper, IdaInterface now demangles names.
 * 2020-05-28: v1.2.15 - Bugfixes in IntelInstructionEscaper (handling of negative RIP-relative offsets), SmdaReport (datetime handling), PeFileParser (handling of empty pefile.sections); SCC calculation changed to iterative algorithm (using @bwesterb's implementation) and activated by default again. 
 * 2020-05-14: v1.2.10 - Bug in IdaInterface fixed.
 * 2020-05-13: v1.2.9 - Bugfix in code gap identification in FunctionCandidateManager, SCC calculation is now optional.
 * 2020-05-12: v1.2.7 - Added additional default metadata field "component" to SmdaReport.
 * 2020-05-11: v1.2.6 - Export from IDA to SMDA data format is now supported (IDA 7.4).
 * 2020-05-09: v1.2.5 - Fixed off-by-one that affected wildcarding of instructions (THX to Viviane Zwanger).
 * 2020-05-04: v1.2.4 - Various minor fixes.
 * 2020-04-29: v1.2.0 - Restructured config.py into smda/SmdaConfig.py to similfy usage and now available via PyPI! The smda/Disassembler.py now emits a report object (smda.common.SmdaReport) that allows direct (pythonic) interaction with the results - a JSON can still be easily generated by using toDict() on the report.
 * 2020-04-28: v1.1.0 - Several improvements, including: x64 jump table handling, better data flow handling for calls using registers and tailcalls, extended list of common prologues based on much more groundtruth data, extended padding instruction list for gap function discovery, adjusted weights in candidate priority score, filtering code areas based on section tables, using exported symbols as candidates, new function output metadata: confidence score based on instruction mnemonic histogram, PIC hash based on escaped binary instruction sequence
 * 2020-03-10: Various minor fixes and QoL improvements.
 * 2019-08-20: IdaExporter is now handling failed instruction conversion via capstone properly.
 * 2019-08-19: Minor fix for crashes caused by PDB parser.
 * 2019-08-05: v1.0.3 - SMDA can now export reports from IDA Pro (requires capstone to be available for idapython).
 * 2019-06-13: PDB symbols for functions are now resolved if given a PDB file using parameter "-d" (THX to @VPaulV).
 * 2019-05-15: Fixed a bug in PE mapper where buffer would be shortened because of misinterpretation of section sizes.
 * 2019-02-14: v1.0.2 - ELF symbols for functions are now resolved, if present in the file. Also "-m" parameter changed to "-p" to imply parsing instead of just mapping (THX: @VPaulV).
 * 2018-12-12: all gcc jump table styles are now parsed correctly. 
 * 2018-11-26: Better handling of multibyte NOPs, ELF loader now provides base addr.
 * 2018-09-28: We now have functional PE/ELF loaders.
 * 2018-07-09: v1.0.1 - Performance improvements.
 * 2018-07-01: v1.0.0 - Initial Release.


## Credits

Thanks to Steffen Enders for his extensive contributions to this project.
Thanks to Paul Hordiienko for adding symbol parsing support (ELF PDB).
The project uses the implementation of Tarjan's Algorithm by Bas Westerbaan and the implementation of Lengauer-Tarjan's Algorithm for the DominatorTree by Armin Rigo.

Pull requests welcome! :)

