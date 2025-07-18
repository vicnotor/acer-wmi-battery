# NixOS Installation

If you're on NixOS, add the following code snippet to a file named
`acer-wmi-battery.nix`:

```nix
{
  stdenv,
  lib,
  kernel,
  fetchgit,
}:
stdenv.mkDerivation {
  pname = "acer-wmi-battery";
  version = "${kernel.modDirVersion}";

  src = fetchgit {
    url = "https://github.com/frederik-h/acer-wmi-battery";
    branchName = "main";
    sha256 = lib.fakeHash; # REPLACE THIS AFTER FAILING SYSTEM REBUILD
  };

  hardeningDisable = ["pic" "format"];
  nativeBuildInputs = kernel.moduleBuildDependencies;

  buildPhase = ''
    make -C ${kernel.dev}/lib/modules/${kernel.modDirVersion}/build \
         M=$(pwd) \
         modules
  '';
  cleanPhase = ''
    make -C ${kernel.dev}/lib/modules/${kernel.modDirVersion}/build \
         M=$(pwd) \
         clean
  '';
  installPhase = ''
    mkdir -p $out/lib/modules/${kernel.modDirVersion}
    cp -r *.ko $out/lib/modules/${kernel.modDirVersion}/
  '';

  meta = {
    description = "A kernel module for setting Acer device battery settings";
    homepage = "https://github.com/frederik-h/acer-wmi-battery";
    license = lib.licenses.gpl2;
    maintainers = ["Frederik Harwath"];
    platforms = lib.platforms.linux;
  };
}
```

To install the kernel module into your system, add the following snippet to
your `configuration.nix` or another module that is imported into your
system configuration:

```nix
boot.extraModulePackages = [(config.boot.kernelPackages.callPackage ./path/to/acer-wmi-battery.nix {})];

boot.kernelModules = ["acer-wmi-battery"];
```

where `./path/to/acer-wmi-battery.nix` is the path to the Nix file
described above. Make sure the arguments of the module that you put this
snippet in contains the `config` argument. The first line of the module
(like `configuration.nix`) should something look like:

```nix
{config, ...}: {
```

When rebuilding your system with something like `nixos-rebuild switch`, the
build will fail and tell you which hash you should use in the
`acer-wmi-battery.nix` file. For example:

```
specified: sha256-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
got:       sha256-mI6Ob9BmNfwqT3nndWf3jkz8f7tV10odkTnfApsNo+A=
```

To fix this, replace `lib.fakeHash` in the `acer-wmi-battery.nix` file with
the hash after "got: " in the error message you receive, with surrounding
quotation marks.

## Setting kernel options declaratively

The following snippet shows an example for setting kernel options
declaratively in your configuration:

```nix
# (Optionally) set options immediately
boot.extraModprobeConfig = ''
  options acer-wmi-battery enable_health_mode=1
'';
```

See [README.md](./README.md) for option references.
