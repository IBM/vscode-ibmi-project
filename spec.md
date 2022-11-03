This project has a few goals in mind, but the goal is to make workspace development for RPGLE, COBOL, CL, etc, a breeze.

Some high level goals for this project are as follows:

## Workspace build tools

We are going to leverage ILE language servers in order to build project dependency trees. This base extension will provide a `registerDepProvider` component so each language server can provide dependencies for a given Uri (based on source).

### Fetching deps

We need to utilise existing language servers here so it is extensible and because we don't want to ram all language implementations into one large LSP.

![image](https://user-images.githubusercontent.com/3708366/199818173-0755f7e0-0575-4156-a08c-0383c690ee54.png)

Each language server would call this extension command to register a dependency provider for a given language idea.

```js
# All interfaces are subject to change - this is not final
commands.executeCommand(`vscode-ibmi-project`, `rpgle`, async (uri: string): Promise<ObjectDepInfo> => {
  // Do work to get the possible object name and any object this 'object' this dependson
  return {
    objectName: path.basename(uri),
    dataDeps: [`EMPS`, `EMPLOYEE`],            // any database object
    programDeps: [`EMPLOYEE`],                 // programs or service programs
    sourceDeps: [`qrpgleref/constants.rpgle`]  // usually header files
  }
});
```

Then, this project extension can each out to each dep provider when it needs to find out specifics for that source.

We also need to somehow support registering many providers for one language. It's likely we only want one provider running for each language ID, so we need to implement some kind of 'priority' in which provider should be called for each language ID if there are many.

### Building data

With the data from our dependency providers, we can create entire builds and also track how changes can impact other parts of an application (the classic 'impact analysis').

First goals would include:

* Automate the creation of Rules.mk
* Automate the creation of makefiles
* Automatically update those build files when sources are saved
* Ability to support many different build tools

### Dependency UI

We should make use of webviews within Visual Studio Code to provide the user with simple diagrams to show deps.

* Right clicking on a file to show a dep tree
* Project wide dep tree
   * This likely isn't a good idea. You have any idea how big those projects can get?

Some nice UX touches for those dep trees

* Clicking on a node should also open the source code in another view without taking photos
* For nodes with many children, nodes should expand and collapse

## UI for project initialisation

* A UI/wizard to help the user initialise a new 'i project'
* Help the user setup which build tool the project should use
* When creating a new project, the ability to import source from a library

### Migrating from library notes

* `makei cvtsrcpf` has the ability to quickly copy sources to the IFS, which the extension could download to a local folder.
* We will have to somehow figure out how to best rename sources
* When migrating from library and generating initial build scripts, we can use out dep providers AND informatiom from the OS via SQL

### Possible wizard functionality

* Select folder for project
* Project Name
* Description
* Configure Build Tool Defaults (also created actions.json)
   * ibmi-bob
   * GNU Make
   * None
* Create initial linter config
   * Yes
   * No
* Start from
   * Blank
   * Import Sources From Library
* Initialise git repo
   * Yes
   * No

If importing from library:

* User enter source library name
* User enter object library name
* Fix source names (lowercase, correct usage of .pgm)
   * Yes
   * No
* Generate build information (if build selected)
   * Yes
   * No
