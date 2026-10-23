---
package: fermitools-fhelp
summary: Plain-text fhelp help files (one per tool) installed to share/fermitools/help; no code.
layer: support
depends_on: []
used_by: []
last_reviewed_commit: f798915
---

# fermitools-fhelp Architecture

## Purpose and scope

**Does:** holds the user-facing help text for the Fermitools command-line
tools, one file per tool, in the HEASoft `fhelp` plain-text style. The
package README says that "any changes to the fhelp files should be
committed to this repository". Help for a tool therefore lives here, not
in the tool's own package.

**Does not:** contain any code, tests, par files or parameter defaults.
Par files (`pfiles/<tool>.par`) stay in each tool's package and are the
source of truth for parameter names, types and defaults. This package
also does not ship an `fhelp` viewer.

## Position in the system

| Direction | Package | Why |
|-----------|---------|-----|
| Depends on | — | Nothing. `CMakeLists.txt` only installs files. |
| Used by | — | No code in `src/` or `recipe/` reads the installed help directory. There are no `#include`s and no runtime lookup of `FERMI_INSTALL_HELPDIR`. The files are for people, read with an external `fhelp` or by hand. |
| Documents | Tools in `Likelihood`, `evtbin`, `dataSubselector`, `irfs`, `timeSystem`, `orbitSim`, `observationSim`, `rspgen`, `burstFit`, `sourceIdentify`, `SolarSystemTools`, `pyExposure`, `pyBurstAnalysisGUI`, `modelEditor` | Content only. Nothing enforces that a help file matches its tool's par file. |

## Key abstractions

Not applicable (no code). The only "abstraction" is the file convention:

- **Layout:** a flat directory, `fhelp_files/`, holding 41 `*.txt` files.
- **Naming:** `<tool>.txt`, where `<tool>` is the executable name (e.g.
  `gtbin.txt`, `gtexpcube2.txt`, `modeleditor.txt`). The name must match
  the command, because that is what a user types to `fhelp`.
- **Structure:** the HEASoft help layout, with upper-case section headings
  at column 0: `NAME`, `USAGE`, `DESCRIPTION`, `PARAMETERS`, `EXAMPLES`,
  `SEE ALSO`. Some files also have `KNOWN BUGS`, `NOTES`, and so on. All
  files except `pypsf.txt` (a free-form 2008 document) use this layout.
  `PARAMETERS` usually lists each par-file parameter in the form
  `name [type]`, and `USAGE` lists the positional parameters in par-file
  order.

## Data flow and control flow

The build only copies files:

```
fhelp_files/*.txt --install(DIRECTORY fhelp_files/ ...)--> <prefix>/share/fermitools/help/
```

`FERMI_INSTALL_HELPDIR` is set in the top-level `CMakeLists.txt` to
`share/${PROJECT_NAME}/help`, i.e. `share/fermitools/help`. The conda
activation script (`recipe/activate.sh`) sets
`FERMI_INST_DIR=$CONDA_PREFIX/share/fermitools`, so the help directory is
`$FERMI_INST_DIR/help`. It does not set any variable (such as HEASoft's
`LHEA_HELP`) that points a help viewer at this directory. How users
actually reach these files outside the source tree has not been verified.
They are probably read with HEASoft `fhelp` or through the FSSC web pages
(inferred).

## Public interfaces

- **Installed files:** `share/fermitools/help/<tool>.txt`.
- **C++ API, applications, Python, par files:** none.

### Environment variables

None: the package has no code. Variables set by conda activation are
described under "Runtime environment" in ScienceTools'
`docs/architecture/README.md`.

### Coverage against built tools

Built tools are taken from `add_executable(gt*)` in `src/*/CMakeLists.txt`
and the console entry points in `recipe/meta.yaml`, as of ScienceTools
`9f7097e4`.

- **Built tools with no help file (5):** `gtdispcube` (map_tools),
  `gtebl`, `gthealcube` and `gttscube` (Likelihood), and `gtexpcubesun`
  (SolarSystemTools). The many GtBurst console scripts (`gtdolike`,
  `gtllebin`, …) have no help files either. Only `gtburst` and
  `modeleditor` among the Python entry points are covered.
- **Help files for tools that are not built (5):** `gtephem`, `gtpphase`,
  `gtophase` and `gtpulsardb` belong to `pulsarDb`/`pulsePhase`, which are
  commented out of `FERMI_COMPONENTS` and are not submodules. `pypsf` is
  an old GLAST-era PyPSF GUI that no current package provides. All five
  are still installed.
- **Par-file parameters not mentioned in the help text** (a whole-word
  search of each `.txt` for every par-file parameter name):
  - `gtdrm`: `edisp_bins`, `wmap`, `convol`, `resample`, `rfactor`,
    `minbinsz`, `ptsrc`, `psfcorr`.
  - `gttsmap`: `wmap`, `edisp_bins`.
  - `gtrspgen`: `phinumbins`.
  - `gtorbsim`: `TimelnCmd`.
  - `gtsrcid`: `outCatQty03`–`09` and `select03`–`08`. These are probably
    covered by a range description.

  Every other help file names all of its tool's parameters.

## Build and test

- **CMake:** no targets. There is a single
  `install(DIRECTORY fhelp_files/ DESTINATION ${FERMI_INSTALL_HELPDIR})`.
  `FERMI_INSTALL_HELPDIR` is defined only by the top-level ScienceTools
  `CMakeLists.txt`, so the package cannot be configured on its own.
- **Tests:** none. The project harness `recipe/tests/ST-unit-test` (see
  "Testing" in ScienceTools' `docs/architecture/README.md`) has no check
  for this package, and no reference data exists. `azure-pipelines.yml` is a legacy per-repo Azure
  pipeline template (build/test/launch stages via `fermi-lat/ci`). It does
  not validate the help content.

## Invariants and gotchas

- **Help and par files can drift silently.** A new or renamed parameter in
  `<pkg>/pfiles/<tool>.par` needs a matching edit here, in a separate
  repository. Nothing checks this (see the coverage list above for
  current drift).
- **The file name is the lookup key.** `gtalphbkg.txt` was renamed to
  `gtalphabkg.txt` (commit `cce1868`) to match the executable. A
  misspelled file installs without complaint but is never found.
- **Duplicate copy in modelEditor.** `src/modelEditor/` has its own
  `modeleditor.txt` (455 lines), which differs from the copy here (486
  lines). Only this one is installed. Separately, `ModelEditor.py`'s
  Help menu opens `$FERMI_INST_DIR/help/help.txt`. Nothing installs a
  `help.txt` there: `modelEditor/help.txt` is not in its
  `install(FILES ...)`. So that menu item most likely fails (from reading
  the code; not checked at runtime).
- **Mixed encodings and line endings.** `gtdiffrsp`, `gtsrcprob`,
  `gtpulsardb` and `gtsrcmaps` have CRLF line endings. Eleven files
  contain non-ASCII UTF-8 characters (e.g. `gtlike`, `gtdrm`, `gtobssim`).
  Plain-ASCII terminals or viewers may show stray characters.
- **Orphan par files are a different problem, owned by other packages.**
  `gtalign.par` (dataSubselector) and `gtecorrect.par` (irfs/irfLoader)
  are installed, but no executable of that name is built. They have no
  help files here either.

## Extension points

- **Adding help for a new tool:** create `fhelp_files/<exact executable
  name>.txt` using the `NAME`/`USAGE`/`DESCRIPTION`/`PARAMETERS`/
  `EXAMPLES`/`SEE ALSO` layout. Copy a sibling (e.g. `gtbin.txt`) and list
  every parameter from the tool's par file, in par-file order. No CMake
  change is needed, because the whole directory is installed.
- **Retiring a tool:** delete its `.txt` here as well. Otherwise it keeps
  being installed (as with the pulsar tools above).

## Further reading

- The par files in each tool's package, `src/<pkg>/pfiles/`.
- `docs/architecture/README.md` (package index) and
  `docs/architecture/dependency-layers.md` (the `support` tier) in
  ScienceTools.
- HEASoft `fhelp` documentation for the help-file convention.
