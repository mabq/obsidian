# Ultimate nix course (vimjoyer)


## Nix Rabbithole Rim (Introduction)

#### [What is Nix](https://www.vimjoyer.com/course/what-nix-is/) 

- Nix is a language and a package manager. NixOS is a Linux distribution built out of both. nixpkgs is where the packages live.
- Declarative means describing the result rather than the commands that reach it.
- Reproducible means the same inputs give the same result on another machine.
- Nix evaluates everything before it builds anything.


## The Evaluation Shallows (Beginner)

#### [Nix Language Basics](https://www.vimjoyer.com/course/nix-language/)

- Nix describes one value, and everything is an expression (no statements).
- Numbers and strings use familiar operators, and brackets change precedence.
- Lists use spaces, not commas.
- `++` joins two lists together into a new one.

#### [Sets and decisions](https://www.vimjoyer.com/course/sets-and-decisions/)

- Attribute sets pair names with values. Dots select values from them.
- Nested dots walk through nested sets one name at a time.
- `let ... in` creates local names and returns the expression after `in`.
- Lazy evaluation wakes only the values the result needs.
- Comparisons produce booleans.
- `if ... then ... else ...` always has both outcomes because it is one expression.

#### [Functions](https://www.vimjoyer.com/course/functions/)

- `input: result` creates a function.
- `function value` calls one.
- `first: second: result` handles inputs one at a time.
- `${...}` drops a calculated value into a string.
- Use a `let` binding to make a function reusable.

#### [Named function inputs](https://www.vimjoyer.com/course/function-arguments/)

- `{ item, room }:` takes one attribute set and names the inputs it needs.
- `servings ? 2` supplies a default when the caller leaves `servings` out.
- `...` deliberately allows names the function does not use.
- Functions can return anything, including an entire configuration.
- Lazy evaluation leaves unused inputs asleep.

#### [Meet nixpkgs](https://www.vimjoyer.com/course/nixpkgs-basics/)

- Nixpkgs is one enormous repository of Nix code, and `pkgs` is what it evaluates to.
- It holds more than packages: the NixOS modules and `lib` live there too.
- Nixpkgs moves, so reproducing a build needs a particular revision rather than only the repository’s name.
- Inside a module you are normally handed `pkgs` as a named function input.
- Channels and `<nixpkgs>` are worth being able to read, because older guides are full of them. Lockfiles are what this course builds toward.

#### [NixOS configuration](https://www.vimjoyer.com/course/nixos-basics/)

- A declarative configuration describes the machine you want.
- NixOS options have names, documentation, and expected value types.
- Lists hold repeated values such as packages and groups.
- Related option paths can be grouped into an attribute set.
- `system.stateVersion` is set once and then left alone.

#### [Try nix without installing](https://www.vimjoyer.com/course/nix-cli/)

- Modern commands use `source#attribute` to identify what they should use.
- `nix shell` lends tools to a temporary shell.
- `nix run` runs one selected app and returns.
- `nix search nixpkgs words` finds attributes from human descriptions.
- Temporary environments change `PATH`, not `/usr/bin` or a global package list.

#### [Reading nix errors](https://www.vimjoyer.com/course/reading-errors/)

- Start with the cause after `error:`, then find the nearest source line you own.
- Dependency frames show the route. Project and `/etc/nixos` paths are destinations.
- Option errors put the message above Definition values and the file inside it.
- Use `--show-trace` only when the filtered trace never returns to your code.
- A missing attribute is a failed lookup. A type error means a value did not fit.
- Shrink the failure before you try to solve it.


## The Module Caverns (Getting started)

#### [Install NixOS](https://www.vimjoyer.com/course/nixos-install/)

- This chapter simulates a graphical nixos installation and a basic `configuration.nix` setup.

#### [Set shorthand](https://www.vimjoyer.com/course/nix-language-more/)

- `inherit name;` is `name = name;`, and `inherit (set) name;` is `name = set.name;`.
- `rec { }` lets a set’s values use the set’s own names.
- `a // b` lays `b` over `a`, at the top level only.

#### [Build functions](https://www.vimjoyer.com/course/builtins/)

- Builtins belong to the language, not nixpkgs.
- `typeOf` inspects a value’s kind.
- `length`, `head`, `tail`, `filter`, and `map` work with lists.
- `attrNames` and `attrValues` inspect attribute sets.
- `toString` prepares simple values for interpolation.
- Nixpkgs brings a far bigger `lib` toolbox along later. These builtins are the small, reliable set that is available absolutely everywhere.

#### [Paths and imports](https://www.vimjoyer.com/course/paths-imports/)

- A path literal and a string are different types, and the type is what lets tooling follow it.
- A relative path is relative to the file containing it, not to where you ran the command. A Nix file carries its own neighbourhood around with it. Move the folder somewhere else and everything it points at goes along.
- `import` evaluates a file’s one expression and returns the value.
- An imported file sees none of your local names, so it asks for what it needs as a function argument.
- Importing a directory evaluates its `default.nix`. A bare path does not.
- Putting a path in a string, or handing it to `src`, copies it into the store.


#### [Lookups and long strings]()
#### [nixpkgs lib]()

#### [Hardware and filesystems]()
#### [Users and NixOS]()
#### [How NixOS modules fit together]()

#### [Custom options]()

#### [NixOS services]()


## The flake fall (Daily driver)

#### []()
#### []()
#### []()