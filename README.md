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

## The Whole Process At A Glance
A `PKGBUILD` file is used to describe how to prepare your software for packaging.
Use `makepkg` to process the `PKGBUILD` into a fully staged package.
Use `pacman` to install the final package into your system.

## The Service
I conveniently found a StackOverflow discussion about another user of an ASUS laptop trying to do exactly what I was doing, so adapting one of the answers was painless - [check out the discussion here.](https://unix.stackexchange.com/questions/716798/how-can-i-make-sysfs-parameter-value-persist-for-each-reboot/716800#716800) It's pretty simple so I'll spell it out here:
1. The service file simply runs a bash command to write to the sysfs variable
2. The service is run as root by systemd so writing to sysfs will work just fine
3. The service needed to be placed in /etc/systemd/system to be utilised

## The PKGBUILD
Packages come in many shapes and sizes and `PKGBUILD` files have to be robust enough to tackle any of them. This project just needs to copy a single file then run a single command, so almost all of the functionality of a `PKGBUILD` and `makepkg` is not needed.

1. Start with a sample `PKGBUILD` which you can copy from `/usr/share/pacman/PKGBUILD.proto`
2. Set the `source` variable to reference your files to be installed
3. Set the `arch` variable to `any` as this package is architecture agnostic (could be x86_64, arm, etc)
4. Remove all functions other than `package()`
5. Use the `package()` function to describe how to stage your files
 - `makepkg` will expect your files to be in the same structure as your host filesytem, eg a file that goes into `/etc/example/file` will be in `your-package-directory/pkg/etc/example/file`

## Packaging and Installing
The last part is the easiest - `makepkg` will process the PKGBUILD to create a complete package that `pacman` can use to install.
1. Run `makepkg` in your package directory
2. A package file will be compiled - on my system it was called `battery-limit-1-1-any.pkg.tar.zst` and likely will be the same or similar for you
3. Run `pacman -U battery-limit-1-1-any.pkg.tar.zst` and pacman will install your package, and run the post install script so that systemd can see the service
4. To start the service, run the command `systemctl enable --now battery-limit.service"
