# Bolt codes

A **bolt code** names a diagnostic — an error or a warning — reported by one of
Propensive's projects, in one form across all of them:

    ↯/p.1.7
    ↯/s.87.2.1
    ↯/s.118.1/t.1.4

A code, once assigned, is permanent. Each project keeps the record of what its
own codes mean, in its own repository; this repository is the index of the
projects ([`projects.tel`](projects.tel)), and holds the scheme and the schema
([`bolt.tel`](bolt.tel)) the records share.

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
either, depending on how a tool is configured or where it is raised. A project's
record says what each normally is.

### The projects

| Letter | Name | Project | Its record |
| ------ | ---- | ------- | ---------- |
| `a` | `lira` | LIRA | [`bolt.tel`](https://github.com/propensive/lira/blob/main/bolt.tel) in `propensive/lira` |
| `c` | `consequent-style` | Consequent Style | [`doc/rules/bolt.tel`](https://github.com/propensive/flare/blob/main/doc/rules/bolt.tel) in `propensive/flare` |
| `f` | `pyrocosm` | Pyrocosm, and its tools | [`bolt.tel`](https://github.com/propensive/pyrocosm/blob/main/bolt.tel) in `propensive/pyrocosm` |
| `o` | `orthant` | Orthant | [`bolt.tel`](https://github.com/propensive/orthant/blob/main/bolt.tel) in `propensive/orthant` |
| `p` | `proscala` | Proscala | [`mk/bolt-codes.tel`](https://github.com/propensive/proscala/blob/main/mk/bolt-codes.tel) in `propensive/proscala` |
| `s` | `soundness` | Soundness | `doc/work/errors/bolt.tel` in `propensive/soundness`, in the Orthant store of its documentation |
| `t` | `tel` | TEL | [`spec/bolt.tel`](https://github.com/propensive/tel/blob/main/spec/bolt.tel) in `propensive/tel` |
| `x` | `xek` | XEK | [`bolt.tel`](https://github.com/propensive/xek/blob/main/bolt.tel) in `propensive/xek` |

[`projects.tel`](projects.tel) is this table as a [TEL](https://github.com/propensive/tel)
document, whose schema is [`index.tel`](index.tel):

    tel 1.0 propensive.dev/bolt-index

    project p proscala
      repository  https://github.com/propensive/proscala
      path mk/bolt-codes.tel

A project which is to have codes takes the next letter it likes that is free,
and a name, by a row here. Nothing else about a project is recorded here: an
error which is added, renamed or retired changes its project's record, and not
this repository.

### Compound codes

A code with more than one part is a **compound**: a diagnostic of one project
carrying a diagnostic of another. The first part is the **carrier** — the
error that was actually reported — and the part after it is the **cause**, which
stands where the carrier's variant would. A carrier's group therefore has no
variant; a third number in it is a site.

The cause is a code in its own right. Dropping the carrier's part leaves it:

| Compound | Carrier | Cause |
| -------- | ------- | ----- |
| `↯/s.118.1/t.1.4` | Soundness, module 118, error 1: an invalid TEL document | `↯/t.1.4`, the TEL parsing error, as the TEL specification and its other implementations name it |
| `↯/p.2.2/s.87.2.1` | Proscala, error 2.2: an error reported by a macro | `↯/s.87.2.1`, the Soundness error the macro reported |

A cause may itself be a compound, so a Soundness macro that rejects a TEL
document at compiletime reports `↯/p.2.2/s.118.1/t.1.4`.

## Names, and the long form

Every project, subdivision, error, variant and site has a **name** as well as a
number, and a code may be written with the names in place of the letter and the
numbers, each in the place of its own:

    ↯/s.87.2.1
    ↯/soundness.legerdemain.query.missing

    ↯/f.8.1.3
    ↯/pyrocosm.remote.peer.unreachable

This is the code's **long form**. The two forms name the same diagnostic, and
either is found from the other in the project's record.

    part = <letter>.<number>[.<number>...]      short
         | <name>.<name>[.<name>...]            long

- A part is short or long throughout: `↯/s.legerdemain.2` is not a code. The
  parts of a compound may differ, as in `↯/proscala.general.macro/s.87.2.1`.
- A project's name has at least two characters, so the first element of a part
  says which form the part is in.
- The short form is the canonical one. It is what a tool prints, and what is
  compared, sorted and stored; the long form is for people to read and write.
- A code has a long form only when every level of it has a name. Not every
  level has one yet: names are being given to those which lack them, and will
  then be required.

### What a name is

A name is written in lower-case **kebab-case**: words of letters and digits,
each beginning with a letter, joined by single hyphens (`type-mismatch`,
`ipv4-byte-out-of-range`).

- **It is the same name in every language.** Where a language prefers another
  case, the name is transliterated word for word, and back: `type-mismatch` is
  `TypeMismatch` in Pascal case and `typeMismatch` in camel case. So a case of
  a Scala enum, `BadSignature`, is the variant `bad-signature`, without either
  being recorded twice.
- **An error named for a type which ends in `Error` drops it.** `Query.Error`
  is `query`, and `Certificate.Path.Error`, `certificate-path`: in a bolt code,
  that it is an error goes without saying.
- **An acronym is one word**: `oauth`, `xpath`, `webauthn`, not `o-auth`.
- **Where what the source calls it is not the name in another case**, as in
  each of the last two rules, the record gives it too, as the `symbol`.
- **A name is unique among its siblings**: the errors of one subdivision, the
  variants of one error. Subdivisions of one letter which several repositories
  share are named for their own: `fuel-core` and `fury-core`.
- **A rename keeps the number, and the old name.** The old name stays in the
  record as an `alias`, so a long-form code written before the rename still
  resolves, and is never given to another of its siblings.

## The rules

- **A code is never reused.** When an error is removed, its row stays, marked
  `retired`.
- **A rename keeps the code.** An error, a variant, a subdivision or a site that
  is renamed, or moved to another file, is the same row with a new name, and its
  old name as an `alias`.
- **Numbers are assigned contiguously**, the next free one in its group. Their
  order means nothing, except for subdivisions.
- **Errors and warnings share one numbering.**
- **A project's record is changed with its source.** The change which adds an
  error adds its row.

## A project's record

A project records its codes in a TEL document (or several, each with some of
its subdivisions) whose schema is [`bolt.tel`](bolt.tel). The document nests
the numbers of a code in the order they are written:

    tel 1.0 propensive.dev/bolt

    project p proscala

      subdivision 1 standard
        error 7 type-mismatch
        error 10 top-level-implicit-class retired

      subdivision 7 typer
        summary  naming, typing and checking
        error 40
          source dotc/typer/Checking.scala
          template  Traits cannot have secondary constructors{}

so `↯/p.1.7`, or `↯/proscala.standard.type-mismatch`, is `project p`,
`subdivision 1`, `error 7`. The four levels are:

| Keyword | Number | |
| ------- | ------ | - |
| `subdivision` | first | a part of the project |
| `error` | second | an error or a warning |
| `variant` | third | a case the error distinguishes |
| `site` | fourth, or third for an error without variants | a place it is reported from |

Each is written with its number, then its name, then optionally `retired`. (A
number which has no name yet and is retired has `retired` as a child, since an
atom after the number is its name.) Beneath any of them may be:

| Keyword | |
| ------- | - |
| `alias` | a name it had before; there may be several |
| `symbol` | what the project's source calls it, where that is not its name in another case |
| `summary` | what it is, in a line |
| `source` | where it is defined or reported from; there may be several |

and beneath an error, a variant or a site, its documentation:

| Keyword | |
| ------- | - |
| `conditions` | when it is reported: what must be so for it to occur |
| `mitigation` | what to do about it |
| `example` | a message as it is reported, with its code before it, and optionally a `note` on it; there may be several |
| `related` | the code of another worth reading about beside it; there may be several |
| `section` | anything else worth saying, under a title, with its text as its `body`; there may be several |
| `successor` | for one which is retired, a code which replaced it |
| `retirement` | for one which is retired, what became of it |
| `severity` | `error` or `warning`, as it normally is |
| `phase` | `compiletime` or `runtime`, as it normally is |
| `template`, `text` | for an error, how the project's tools recognise it where that is not by its name; there may be several of each (a message which has been reworded, say) |

`conditions`, `mitigation`, `retirement`, an example's `note` and a section's
`body` are prose in Markdown, and the rest plain text:

    error 2 query
      symbol Query.Error
      summary  A query parameter cannot be read
      source lib/legerdemain/src/query/legerdemain.Query.scala
      phase runtime
      conditions
          Legerdemain decodes named query parameters into typed values, and
          raises this for the first which it cannot.

          A parameter with a `Default` is still reported when it is missing.
      variant 1 missing
        summary  the parameter was not present in the query string
        example  [↯/s.87.2.1] the query parameter could not be read because the parameter was not present in the query string
        mitigation
            Include the parameter, or supply a `Default[T]` to fall back on.
        section  Wire code
          body
              None: the error is not sent to the client.

A value with spaces in it follows two spaces, as TEL requires. Prose of several
lines is a TEL *source atom*: the lines below its keyword, indented by four
spaces more, with blank lines between its paragraphs. TEL strips the spaces at
the end of each line of one, so prose which needs them, or a blank line at its
start or end, is written as a *literal atom* instead.

The schema makes the numbers of each level distinct. `tel validate` checks a
record against it, once `tel schema add bolt.tel`, with this repository's
`bolt.tel`, has made the schema known; and likewise `tel schema add index.tel`
for `projects.tel`. The schema cannot say that names
are distinct among siblings, nor that a record agrees with its project's source:
those are for each project to check.
