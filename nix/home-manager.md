# Home Manager

### Activation scripts

```nix
home.activation = {
  myScript = lib.hm.dag.entryAfter [ "dependencyName" ] ''
    # Your bash script goes here...
  '';
};
```

The possible values for `lib.hm.dag.entryAfter` are simply the **names of other activation script blocks**, both built-in ones like `writeBoundary` and your own custom ones. The key is understanding that this list is used to create a dependency graph for safe and predictable script execution.

You can specify one or more dependencies in the list.

**The Critical Boundary: `writeBoundary`**

This is a pivotal entry in the activation process. It acts as a separator between a **verification phase** and a **modification phase** .

*   **Before `writeBoundary`**: Scripts should be **read-only**. This is where you would check for preconditions or potential conflicts. If a script fails here, the entire activation is aborted without making changes .
*   **After `writeBoundary`**: Scripts are allowed to make **changes to the filesystem**, such as creating directories, copying files, or reloading services. Most custom scripts should be placed here .

**Other Pre-Defined Entries**

The Home Manager activation process uses a Directed Acyclic Graph (DAG) with several standard nodes . Understanding their sequence can help you correctly position your script.

| Entry Name | Position Relative to `writeBoundary` | Purpose |
| :--- | :--- | :--- |
| `checkLinkTargets` | Before | Verifies no file conflicts exist between the new generation and the filesystem. |
| `writeBoundary` | (The Boundary) | The critical commit point separating verification from modification. Tells Home Manager that your script should be placed directly after the `writeBoundary` entry. |
| `installPackages` | After | Installs the home-manager-path package into the user profile. |
| `linkGeneration` | After | Creates new symlinks for files defined in `home.file`. |
| `onFilesChange` | After | Executes `onChange` shell snippets for files that were modified. |

The list of strings is not limited to built-in entries. You can also reference the names of other **custom scripts** you have defined . This allows you to build a clear and safe execution order for your own complex operations.
