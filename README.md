# vivado-git

Trying to make Vivado more Git-friendly.

## 1. Requirements

- Tested with Vivado 2021.2.1

### 1.1. Windows

- [Git for Windows](https://git-scm.com/download/win)
- Add `C:\Program Files\Git\bin` (or wherever you have your `git.exe`) to your `PATH`

### 1.2. Linux

- Git

## 2. Installation

Add `Vivado_init.tcl` (or append the relevant lines if you already have
something in it) along with the `scripts` directory to:

- `%APPDATA%\Xilinx\Vivado` on Windows
- `~/.Xilinx/Vivado` on Linux

## 3. How it works

Vivado is a pain in the ass to source control decently, so these scripts provide:

- A modified `write_project_tcl_git.tcl` script to generate the project script
  without absolute paths.

- A Git wrapper that will recreate the project script and add it before committing.
  All the Git commands can be used from the Tcl Console on the Vivado GUI.

- A Tcl script (`wproj`) to just create the Tcl project generator script without
  using git. This script can also be called from the Tcl Console on Vivado.
  **This is the prefered way to generate the project generator script if you
  want to handle Git with an external program**
  (like Git from a terminal, SourceTree, VS Code, etc).

## 4. Workflow

 1. When first starting a project, create it in a folder called `vivado_project`
    (e.g. `PROJECT_NAME/vivado_project`). All the untracked files will be under this directory.

 2. Place your source files anywhere you want in your project folder
    (e.g. `PROJECT_NAME/src`).

    Here is an example of a possible project structure:

    ``` txt
    PROJECT_NAME
        ├── .git
        ├── .gitignore
        ├── project_name.tcl         # Project generator script
        ├── src/                     # Tracked source files
        │   ├── design/
        │   │    ├── *.v
        │   │    └── *.vhd
        │   ├── testbench/
        │   │    ├── *.v
        │   │    └── *.vhd
        │   └── ...
        ├── ips/                     # Tracked project-specific IP repository
        │   ├── my_first_ip/
        │   │    ├── src/
        │   │    ├── xgui/
        │   │    └── component.xml
        │   ├── my_second_ip/
        │   └── ...
        └── vivado_project/          # Untracked generated files
            ├── project_name.xpr
            ├── project_name.cache/
            ├── project_name.hw/
            ├── project_name.sim/
            ├── project_name.srcs/
            │    ├── sources_1/
            │    │    ├── bd/             # BDs are regenerated from script
            │    │    │    ├── my_bd/
            │    │    │    └── ...
            │    │    └── imports/hdl/    # BD wrappers are also regenerated
            │    │         ├── my_bd_wrapper.{v,vhd}
            │    │         └── ...
            │    │
            │    └── ...
            └── ...
    ```

 3. Initialize the git repository with `git init` on the Tcl Console. This will
    create the repository, automatically change to your project directory
    (`PROJECT_NAME`), generate the `.gitignore` file and stage it.

 4. Stage your source files with `git add`.

 5. When you are done, `git commit` your project. A `PROJECT_NAME.tcl`
    script will be created in your `PROJECT_NAME` folder and added to your commit.

      - **Note**: Always use the `-m` argument to pass your message when committing.
      This is needed because the Tcl Console on Vivado cannot handle terminal-based
      editors.

 6. Afterwards, when opening the project after cloning it, do it by using
    `Tools -> Run Tcl Script...` and selecting the `PROJECT_NAME.tcl` file
    created earlier. This will regenerate the project so that you can continue to work.

## 5. Sample project

Refer to the `reaction_timer` project under `sample` if you want an
example on how you can structure your project.

## 6. Notes

### 6.1. Block design support

If a block design is present, Tcl procedures will be integrated in the project
generator file to regenerate it. Make sure to specify it as `<Local to Project>`
when creating the Block Design with the GUI, so that it is created inside the
`vivado_project` directory.

The script will also automatically create and add the BD wrapper to the project.

The wrapper of the `.bd` file **must** be called `${bd_name}_wrapper`
(e.g. `my_awesome_bd_wrapper` if your BD is called `my_awesome_bd`),
which is the default when creating in the GUI with `Create HDL Wrapper...`.

The BD wrapper that is automatically generated by Vivado **must not** be
tracked by Git. If you need to manually modify the BD wrapper generated by Vivado,
you can write a handwritten wrapper to the generated wrapper and put only the
handwritten one under source control.

Also note that the layout information of the block design will not be kept.

### 6.2. Board part and IP repositories paths

Only board part and IP repositories inside the project are stored in the project
generator script.

If you have a system wide board part or IP repository, you will need to add it manually
after recreating the project from the Tcl script (e.g. via `Settings --> Board Repository`).
