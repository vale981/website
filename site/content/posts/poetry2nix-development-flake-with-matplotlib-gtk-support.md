+++
title = "Poetry2Nix Development Flake with Matplotlib GTK Support"
author = ["Valentin Boettcher"]
date = 2024-05-11T17:57:00-04:00
categories = ["Hacks"]
draft = false
+++

I recently had the pleasure to dive back into python for work. In the
past, I was happily using `org-babel` notebooks through
[emacs-jupyter](https://github.com/nnicandro/emacs-jupyter). However, I have since switched to a more REPL/script
driven workflow as I find that programming notebooks require a great
deal of discipline to not end up as a horrible mess. For my new
workflow, I need interactive plotting to work.

So let's get straight to the meat. The following `Flake` dives you a
development shell that tries to replicate the underlying [poetry](https://python-poetry.org/)
project in full nix using [poetry2nix](https://github.com/nix-community/poetry2nix).

```nix
{
  description = "[your description]";

  inputs = {
    flake-utils.url = "github:numtide/flake-utils";
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable-small";
    poetry2nix = {
      url = "github:nix-community/poetry2nix";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs = inputs @ { self, nixpkgs, flake-utils, ... }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = nixpkgs.legacyPackages.${system};
        poetry2nix = inputs.poetry2nix.lib.mkPoetry2Nix { inherit pkgs; };
      in
      {
        packages = {
          yourPackage = poetry2nix.mkPoetryApplication {
            projectDir = self;

            # set this to true to use premade wheels rather than the source
            preferWheels = false;

            # this enables interactive plotting support with GTK
            overrides = poetry2nix.overrides.withDefaults (final: prev: {
              matplotlib = with pkgs; prev.matplotlib.override
                {
                  passthru.args.enableGtk3 = true;
                };
            });
          };
          default = self.packages.${system}.yourPackage;
        };

        # Shell for app dependencies.
        #
        #     nix develop
        #
        # Use this shell for developing your app.
        devShells.default = pkgs.mkShell {
          inputsFrom = [ self.packages.${system}.yourPackage ];
          package = with pkgs; [
            # any development dependencies that you might have in nixpkgs
            ruff
            pyright
          ];
        };

        # Shell for poetry.
        #
        #     nix develop .#poetry
        #
        # Use this shell for changes to pyproject.toml and poetry.lock.
        devShells.poetry = pkgs.mkShell {
          packages = [ pkgs.poetry ];
        };
      });
}
```

The workflow is as follows. Running `nix develop .#poetry` will give you
a shell with poetry available. You can then `poetry init` and `poetry add`
and `poetry lock` (not install) to your hearts content. A plain `nix
develop` will then set up the environment according to the `poetry.lock`
that poetry generates. Note that [this pull request](https://github.com/nix-community/poetry2nix/pull/1651) has to be resolved
before the above works with `preferWheels = true`.

You might want to checkout [direnv](https://direnv.net/) and [nix-direnv](https://github.com/nix-community/nix-direnv) for added convenience.
