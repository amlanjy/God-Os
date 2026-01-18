IOMOS — Intent-Oriented Multisemantic Operating System
====================================================

STATUS
------
Design and specification phase.
No implementation committed yet.


ABSTRACT
--------
IOMOS proposes an operating system architecture that separates semantic intent
from OS-specific execution mechanisms. A single neutral kernel hosts multiple
concurrent OS semantic personalities (Linux/POSIX, Windows NT, Darwin/BSD),
all operating over shared hardware, boot chain, and filesystem.

Interoperability is achieved via semantic translation, not CPU emulation,
ABI multiplexing, virtualization, or binary rewriting.


DESIGN GOALS
------------
- Single kernel, single EFI, single root filesystem
- Concurrent execution of multiple OS semantic models
- Intent-first execution model
- No CPU emulation or virtual machines
- No proprietary code or DRM interaction
- Incremental, user-space-first implementation
- Explicit handling of semantic divergence


NON-GOALS
---------
- Perfect behavioral equivalence
- Syscall-number compatibility
- ABI-level compatibility
- Binary patching or rewriting
- Container-based isolation
- Replacement of existing operating systems


HIGH-LEVEL ARCHITECTURE
-----------------------

Intent Expression
        |
        v
Intent Resolution Engine
(context + scope + persona)
        |
        v
Semantic Personality Layer
(Linux / NT / Darwin)
        |
        v
Neutral Core Kernel
        |
        v
Hardware


CORE KERNEL MODEL
-----------------
The kernel implements mechanisms only.

Kernel responsibilities:
- Scheduling
- Virtual memory primitives
- IPC and message routing
- Filesystem object abstraction
- Capability enforcement
- Persona isolation and dispatch

Kernel does NOT implement:
- POSIX semantics
- NT object semantics
- BSD/Mach semantics
- Path syntax
- Command syntax or flags

The kernel exposes a structured request interface, not a traditional syscall ABI.


SEMANTIC PERSONALITIES
---------------------
A semantic personality defines how intent is interpreted and mapped to kernel
primitives.

Linux / POSIX personality:
- fork/exec lifecycle
- POSIX filesystem semantics
- signals
- Unix permission model
- case-sensitive paths

Windows NT personality:
- NT object handles
- CreateProcess semantics
- mandatory file locking
- drive-letter namespace
- case-insensitive paths

Darwin / BSD personality:
- BSD syscall semantics
- extended attributes
- resource fork mapping
- Mach-style IPC abstraction

Each process is bound to exactly one personality at creation and cannot switch
personas.


WORKSPACE MODEL
---------------
- Multiple personas execute concurrently
- Workspaces are semantic contexts, not virtual machines
- CPU scheduling and memory are globally shared
- Cross-persona interaction is explicit and mediated

Workspace switching affects user interaction context only.


FILESYSTEM MODEL
----------------

Physical layer:
- Single EFI
- Single kernel image
- Single root filesystem
- Single set of physical file objects

Semantic projection:
- Windows maps drive letters to directories
- Linux uses FHS-style hierarchical root
- Darwin uses BSD layout with extended metadata

Internal path representation:
Paths are stored internally as component vectors:

["mnt", "abc", "d", "dfy"]

Neutral path syntax (optional user interface):

/\mnt/\abc/\d/\dfy

Syntax translation occurs only at persona boundaries.


INTENT LANGUAGE
---------------
IOMOS defines a canonical, OS-neutral intent representation.

Example user input:
delete dfy

Resolved intent:
operation: delete
target: path(/\mnt/\abc/\d/\dfy)
recursive: true
force: true

Persona execution mapping:
- Linux: unlinkat() recursion
- Windows: NT delete disposition
- BSD: unlink() with metadata handling


INTENT GRAMMAR (BNF)
-------------------

<intent>        ::= <operation> <target> <modifier_list>

<operation>     ::= delete | create | open | read | write
                  | execute | move | copy | list | stat

<target>        ::= <path> | <object_ref>

<path>          ::= <neutral_path>
<neutral_path>  ::= "/\" <component> ("/\" <component>)*

<component>     ::= <identifier>

<modifier_list> ::= (<modifier>)*
<modifier>      ::= <attribute> | <constraint>

<attribute>     ::= recursive | force | exclusive | shared | append

<constraint>    ::= <key> ":" <value>

<identifier>    ::= [a-zA-Z_][a-zA-Z0-9_]*
<value>         ::= identifier | number | boolean


CANONICAL INTENT IR (JSON)
-------------------------

{
  "operation": "delete",
  "target": {
    "type": "path",
    "value": ["mnt", "abc", "d", "dfy"]
  },
  "attributes": {
    "recursive": true,
    "force": true
  },
  "constraints": {}
}


KERNEL–PERSONA IPC
------------------
The kernel does not interpret semantics.
Personas do not access hardware directly.

IPC message structure (conceptual):

struct intent_msg {
    version
    persona_id
    operation_id
    target
    attributes
    constraints
    execution_context
}

Kernel response structure:

struct intent_result {
    status
    error_class
    message
    result_data
}

Kernel validates structure and capabilities only.
Semantic decisions belong exclusively to personas.


INTENT RESOLUTION RULES
-----------------------
1. Context resolution (workspace, cwd, persona)
2. Deterministic semantic expansion
3. Ambiguity detection
4. Explicit failure on irreconcilable semantics
5. No silent fallback behavior

Example divergence:
Linux permits deletion of open files.
Windows forbids deletion of open files.

Resolution must be explicit: fail, defer, or degrade with warning.


INCREMENTAL IMPLEMENTATION STRATEGY
----------------------------------
1. User-space intent interpreter (Linux host)
2. POSIX semantic personality
3. Structured intent IPC
4. Filesystem semantic projection
5. Optional kernel hooks
6. Additional semantic personalities

Early prototypes require no kernel modification.


LICENSING
---------
This project is licensed under the GNU General Public License v3 (GPLv3).

All implementations and derivatives must remain GPLv3.
No closed or proprietary semantic forks permitted.


LEGAL POSITION
--------------
- Clean-room implementation only
- No proprietary source usage
- Behavior reimplemented via documentation and observation
- No DRM or license circumvention
- Comparable precedent: Wine, Samba, ReactOS


SUMMARY
-------
IOMOS reframes operating system interoperability as a semantic translation
problem rather than a binary or hardware one. By separating intent from
execution and enforcing strict semantic personas, it enables concurrent,
honest interoperability without emulation or virtualization.
