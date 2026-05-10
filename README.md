# Battery Limit
A simple systemd service to limit maximum battery charging level on my personal laptop by writing a value into sysfs:
```
echo 80 > /sys/class/power_supply/BAT0/charge_control_end_threshold
```
This value is present on my laptop, an ASUS ROG STRIX G513QY. I suspect it will be present on other ASUS laptops and possibly many more.

I wanted to publish this to the public in case someone else has the same problem I did. I also thought it might be helpful to make a laymans guide to using a PKGBUILD locally, as the documentation is broad to the point of total vagueness and the community largely preferred to tell people to RTFM instead of providing a basic example.

## Contents
1. [The Whole Process At A Glance](#the-whole-process-at-a-glance)
2. [The Service](#the-service)
3. [The PKGBUILD](#the-pkgbuild)
4. [Packaging and Installing](#packaging-and-installing)
5. [Putting It All Together](#putting-it-all-together)

## The Whole Process At A Glance
1. A `PKGBUILD` file is used to describe how to prepare your software for packaging.
2. Use `makepkg` to process the `PKGBUILD` into a fully staged package.
3. Use `pacman` to install the final package into your system.

## The Service
I conveniently found a StackOverflow discussion about another user of an ASUS laptop trying to do exactly what I was doing, so adapting one of the answers was painless - [check out the discussion here.](https://unix.stackexchange.com/questions/716798/how-can-i-make-sysfs-parameter-value-persist-for-each-reboot/716800#716800) It's pretty simple so I'll spell it out here:
1. The service file simply runs a bash command to write to the sysfs variable
2. The service is run as root by systemd so writing to sysfs will work just fine
3. The service needed to be placed in /etc/systemd/system to be utilised

## The PKGBUILD
Packages come in many shapes and sizes and `PKGBUILD` files have to be robust enough to tackle any of them. This project just needs to copy a single file then run a single command, so almost all of the functionality of a `PKGBUILD` and `makepkg` is not needed.

1. Start with a sample `PKGBUILD` which you can copy from `/usr/share/pacman/PKGBUILD.proto`
2. Set the `pkgname` variable to the name of your package
3. Set the `source` variable to reference your files to be installed
4. Set the `arch` variable to `any` as this package is architecture agnostic (could be x86_64, arm, etc)
5. Remove all functions other than `package()`
6. Use the `package()` function to describe how to stage your files
7. If you need to run any commands to configure your software post installation, write them in a shell script and reference that script in the `install` variable of your `PKGBUILD`
8. If you want to have `makepkg` run file integrity checks, you can include them in the default `sha256sums` variable

## Packaging and Installing
The last part is the easiest - `makepkg` will process the PKGBUILD to create a complete package that `pacman` can use to install.
1. Run `makepkg` in your package directory
2. `makepkg` will generate a package file named using the variables
3. Use `pacman -U your-package.pkg.tar.zst` to install the package

## Putting It All Together
This section will try to put the steps above into context by referencing where/how they're put together in this repository.

1. Set the PKGBUILD variables for my project:
 - set `pkgname=battery-limit` as a name is needed
 - set `pkgdesc` to something useful so that it stands out in pacman
 - set `arch=(any)` as an architecture is needed
 - optional: set `license=('unknown')` because the default GPL was flagging as an issue in my editor (it wasn't)
 - set `install="battery-limit.install"` to bundle my post-install script (more on that later)
 - set `source=("battery-limit.service")` to include my systemd service file (the thing I want to install)
 - remove the `sha256sums` variable - `makepkg` supports a variety of file integrity checks, but you can bundle them into `cksums` instead
 - add and set `cksums=("SKIP")` because I don't care about file integrity checks for a local script

2. Strip out all functions other than `package()`
3. Use `package()` to stage the files for the final package
 - `makepkg` will copy your `source` files into `<package-directory>/src` which it will expose as a variable named `srcdir`
 - The expectation is that you would use the other `PKGBUILD` functions to prepare/compile etc the files in `srcdir`
 - `makepkg` will create a directory at `<package-directory>/pkg` which it will expose in the variable `pkgdir`
 - The expectation is that you move your prepared files into `pkgdir` imitating the directory structure they will be installed to in your live system
 - The destination for my service file is `/etc/systemd/system/battery-limit.service`
 - I use the `install` tool (mostly because almost all PKGBUILDs I referenced used it too) to copy the service from `srcdir` into `pkgdir/etc/systemd/system/battery-limit.service`
4. Write the post install script in `battery-limit.install`
 - It's just a bash script, the convention is to name it package.install but you can call it whatever you like
 - To use the script after installation, systemd needs to be told to reload the list of services it has, so this script does exactly that by running `systemctl daemon-reload`
 - It's supposedly against Arch conventions to automatically start a service, so I maintained that convention here
5. Run `makepkg` to generate the package
 - The generated package file will be named based on the variables you have set in your `PKGBUILD`
 - `pkgdesc-pkgver-pkgrel-arch.pkg.tar.zst` is the format it used on my system
6. Run `pacman -U your-package.pkg.tar.zst` to install your package
