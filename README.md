# Bolt codes

A **bolt code** names a diagnostic — an error or a warning — reported by one of
Propensive's projects, in one form across all of them:

    ↯/p.1.7
    ↯/s.87.38.2
    ↯/s.87.1/t.1.4

This repository is the record of which code means what. A code, once assigned,
is permanent.

## The form

    ↯/<part>[/<part>...]

    part  = <letter>.<group>
    group = <subdivision>.<error>[.<variant>][.<site>]

- `↯` marks a bolt code.
- Each **part** begins with `/`, and is the **letter** of a project followed by a
  **group** of numbers in that project.
- A group's numbers are separated by `.`, and each counts from 1:
  - the **subdivision** of the project: a module of Soundness, a tool of
    Pyrocosm, a part of the compiler. A project without subdivisions uses `1`.
    Subdivisions are numbered in an order in which a subdivision comes after
    those it depends on, as the project stood when each was numbered;
  - the **error** within that subdivision;
  - optionally the **variant** of that error, where it distinguishes several
    (`Busy`, among the reasons an I/O operation failed);
  - optionally the **site**, where the same error (and variant) is reported
    from more than one place and it helps to tell them apart. An error
    without variants has its site as its third number.

A code does not say whether it is an error or a warning, nor whether it is
reported when a program is compiled or when it runs: the same diagnostic can be
either, depending on how a tool is configured or where it is raised. The
registry records what each normally is.

### The projects

| Letter | Project | Registry |
| ------ | ------- | -------- |
| `a` | LIRA | [`a.tel`](a.tel) |
| `c` | Consequent Style | [`c.tel`](c.tel) |
| `f` | Pyrocosm, and its tools | [`f.tel`](f.tel) |
| `p` | Proscala | [`p.tel`](p.tel) |
| `s` | Soundness | [`s.tel`](s.tel) |
| `t` | TEL | [`t.tel`](t.tel) |
| `x` | XEK | [`x.tel`](x.tel) |

Every project's codes have been assigned.

### Compound codes

A code with more than one part is a **compound**: a diagnostic of one project
carrying a diagnostic of another. The first part is the **carrier** — the
error that was actually reported — and the part after it is the **cause**, which
stands where the carrier's variant would. A carrier's group therefore has no
variant; a third number in it is a site.

The cause is a code in its own right. Dropping the carrier's part leaves it:

| Compound | Carrier | Cause |
| -------- | ------- | ----- |
| `↯/s.87.1/t.1.4` | Soundness, module 87, error 1: an invalid TEL document | `↯/t.1.4`, the TEL parsing error, as the TEL specification and its other implementations name it |
| `↯/p.2.2/s.87.38.2` | Proscala, error 2.2: an error reported by a macro | `↯/s.87.38.2`, the Soundness error the macro reported |

A cause may itself be a compound, so a Soundness macro that rejects a TEL
document at compiletime reports `↯/p.2.2/s.87.1/t.1.4`.

(The Soundness module numbers in these examples are illustrative: its codes are
not assigned yet.)

## The rules

- **A code is never reused.** When an error is removed, its row stays, marked
  `retired`.
- **A rename keeps the code.** An error, a variant, a subdivision or a site that
  is renamed, or moved to another file, is the same row with a new name.
- **Numbers are assigned contiguously**, the next free one in its group. Their
  order means nothing, except for subdivisions.
- **Errors and warnings share one numbering.**

## The registries

Each project's codes are recorded in a [TEL](https://github.com/propensive/tel)
document named for its letter, whose schema is [`bolt.tel`](bolt.tel). The
document nests the numbers of a code in the order they are written:

    tel 1.0 propensive.dev/bolt

    project p proscala

      subdivision 1 standard
        error 7 TypeMismatch
        error 10 TopLevelImplicitClass retired

      subdivision 7 typer
        description  naming, typing and checking
        error 40
          source dotc/typer/Checking.scala
          template  Traits cannot have secondary constructors{}

so `↯/p.1.7` is `project p`, `subdivision 1`, `error 7`. The four levels are:

| Keyword | Number | |
| ------- | ------ | - |
| `subdivision` | first | a part of the project |
| `error` | second | an error or a warning |
| `variant` | third | a case the error distinguishes |
| `site` | fourth, or third for an error without variants | a place it is reported from |

Each is written with its number, then optionally the name the project knows it
by, then optionally `retired`. (An unnamed number which is retired has `retired`
as a child, since an atom after the number is its name.) Beneath it may be:

| Keyword | |
| ------- | - |
| `description` | what it is, for the reader only |
| `source` | where it is defined, for the reader only |
| `template`, `text` | for an error, how the project's tools recognise it where that is not by its name; there may be several of each (a message which has been reworded, say) |

A value with spaces in it follows two spaces, as TEL requires. The schema makes
the numbers of each level distinct; `tel validate p.tel` checks a registry
against it, once `tel schema add bolt.tel` has made the schema known.

`p.tel` is maintained by Proscala's `bin/proscala-bolt`, which describes its
subdivisions and how the compiler's messages are recognised.

`t.tel` records the errors the TEL specifications define, which every
implementation of TEL reports under the same codes, and those of the `tel`
tool. `x.tel` records XEK's: those of the launcher at the start of every XEK
executable, and those of the `xek` command, whose every call site is recorded
as a `site`. Both are edited by hand, with the next free number.

`s.tel` records Soundness's: a subdivision for each library, numbered so that
each follows those it depends on, an error for each numbered error of the
library, and a variant for each reason the error distinguishes. A library
added later takes the next free number.

`a.tel` records LIRA's: subdivision 1 is the validity rules of the LIRA
Specification and its companions, which every implementation reports under the
same codes (`↯/a.1.8` was L108), and the subdivisions after it are the modules
of the `lira` implementation, in the order of their dependencies.

`c.tel` records Consequent Style's: a subdivision for each of its principles, in
the order the standard sets them out, an error for each rule, numbered within
its principle, and a variant for each distinct failure of a rule (`↯/c.2.2.1`
was `A2.1`). A checker reports a violation under a code of its own carrying the
rule's: flare's is `↯/f.21.1/c.2.2.1`.

`f.tel` records the codes of Pyrocosm and its tools, which share a letter: a
subdivision for each module of Pyrocosm, numbered so that each follows those it
depends on (`harness` 1 to `cli` 14), then the modules of fever (15–16), flame
(17–20), flare (21–24), fuel (25–28), fume (29–32) and fury (33–34), each
tool's in the order of its own modules' dependencies. A subdivision added
later takes the next number: fury's lint findings, first numbered `33.2` to
`33.10` among its core module's errors, moved to a subdivision of their own,
`35`, and their old numbers are retired. Errors and the
diagnostics a tool reports, a linter's findings and a document's faults alike,
have codes.
