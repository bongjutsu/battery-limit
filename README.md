# Battery Limit
A simple systemd service to limit maximum battery charging level on my personal laptop

## Contents
1. [Why make this?](#why-make-this)

## Why make this?
I forgot to install the ASUS laptop tools last time I set up Arch Linux on my laptop. I discovered that the only setting I really cared about - limiting the battery charge - is a setting exposed in sysfs, at `/sys/class/power_supply/BAT0/charge_control_end_threshold`. After researching ways to persist sysfs settings after reboots, I settled on a basic systemd service that runs once on boot.

## Why publish this?
In the past, when I've done things like make edits in /etc or made other similar behind the scenes changes to my system, I would forget about them since they were out of sight, out of mind. So I thought I would use this little service as a vehicle to teach myself how to make a PKGBUILD so that I could reinstall the service in the future if I need to reinstall my OS for whatever reason. In my research I found the PKGBUILD documentation to be extremely broad and verbose, making it require a staggering amount of reading to do something very, very basic. When trying to find some light examples in the community, most of the discussion whittled down to good ol' RTFM. I don't think it should be such a mission to find a bare bones **working** example, so I thought I would publish my findings here just in case it was useful to someone.

## The Whole Process At A Glance
Some of you reading this may just need a generalised abstract of the steps required to do something as simple as use a PKGBUILD to install a single file in your system, so I'll start off with a simplified short version of the process right here.
1. Copy the example PKGBUILD from `/usr/share/pacman/PKGBUILD.proto`
2. Add your file to be installed into the `source` variable
3. Remove all functions from the PKGBUILD except `package()`
4. Use `package()` to replicate the directory/file structure of where your file will be installed
5. Run `makepkg` to process the PKGBUILD into a package file for `pacman`
6. Run `pacman -U your-generated-package.pkg.tar.zst` to install your package
7. If you need pacman to run any commands after installation to configure your software, you can make a shell script and reference it in the `install` variable within the PKGBUILD

## Part 1: The Service
I conveniently found a StackOverflow discussion about another user of an ASUS laptop trying to do exactly what I was doing, so adapting one of the answers was painless - [check out the discussion here.](https://unix.stackexchange.com/questions/716798/how-can-i-make-sysfs-parameter-value-persist-for-each-reboot/716800#716800) It's pretty simple so I'll spell it out here:
1. The service file simply runs bash and supplies an echo command that will write the sysfs variable
2. The service is run as root by systemd so writing to sysfs will work just fine
3. The service needed to be placed in /etc/systemd/system to be utilised

## Part 2: The PKGBUILD - Short Version
The only function that's required in PKGBUILD is `package()`. `makepkg` will stage your source files in `$srcdir`. Use `package()` to copy the service from `$srcdir` to `$pkgdir/etc/systemd/system/battery-limit.service` to represent where the files go in your real filesystem after installation. Then, run the post-install script `battery-limit.service` which makes systemd reload available service files so that the service is made available to the user/system.

## Part 3: The PKGBUILD - Long Version
All I needed my PKGBUILD to do was copy the systemd service to where it needed to be, and make systemd aware of its presence. It would be nice to also automatically start the service, but according to the Arch documentation I read, starting services should be up to the user. I decided to go with that same philosophy (partially because I couldn't actually make the service start as part of the install, lol). Here is what I did to make it work:

1. Copy the Arch Linux provided sample PKGBUILD to my project directory (find it in /usr/share/pacman/PKGBUILD.proto)
2. Set the pkgdesc variable to describe the package
3. Set the arch variable to any as it needs to be set to something for the PKGBUILD to be valid
4. Set the license to unkown because I don't really care about licensing
5. Set the install variable to reference the post-install script
 - For systemd to use the service, you need to run the command `systemctl daemon-reload`
 - `makepkg` can bundle in a post install script that is the most logical place to run this command
7. Set the source variable to include the service file
8. Remove the sha256sums variable and replace it with cksums variable, and set it to SKIP as I don't care about local integrity checks
9. Strip out all provided functions except package()
 - There are a lot of functions included so that you can prepare/build/etc your package before packaging
 - The only function that's required by makepkg is package()
 - My service doesn't need to be prepared or built in any way like some packages, so I stripped out all the functions other than package()
9. Use the package() function to put the service file where it needs to be
 - makepkg makes two folders during package creation: src and pkg
 - src contains source files (the files in the source variable/array) and is intended as a staging ground for building and processing where you would run make commands/compile etc
 - pkg is where the finished files go, and should replicate the directory structure of your system as it will be, for lack of a better term, copy pasted into your root partition
 - the package() function therefore simply copies `src/battery-limit.service` to `pkg/etc/systemd/system/battery-limit.service` to demonstrate where the file will go after installation

## Part 4: Packaging and Installing
The last part is the easiest - `makepkg` will process the PKGBUILD to create a complete package that `pacman` can use to install.
1. Run `makepkg` in your package directory
2. A package file will be compiled - on my system it was called `battery-limit-1-1-any.pkg.tar.zst` and likely will be the same or similar for you
3. Run `pacman -U battery-limit-1-1-any.pkg.tar.zst` and pacman will install your package, and run the post install script so that systemd can see the service

4. 
modern practice is to not charge batteries past a certain level to maintain their lifespan
the percentage to stop at is exposed at /sys/class/power_supply/BAT0/charge_control_end_threshold
the default value is 100 and changing it does not persist at boot
there are many ways around this problem but the solution i settled on (mostly due to portability) is a systemd service
battery-limit.service is a system wide systemd service that runs once on boot and sets the threshold to 80
in order to persist the setting across a potential reinstall i wrapped it in a PKGBUILD
the PKGBUILD documentation is insanely verbose so making a basic one was a pain in the ass
put simply, the only required PKGBUILD function is package()
makepkg creates a src directory to be used for building/patching/preparing your package
it then creates a pkg directory which basically (i think) gets copy pasted over /
	-> the service should be installed to /etc/systemd/system/battery-limit.service so pkg contains that same structure
you can also specify a post install script to be run
for systemd to detect the service, you need to run systemctl daemon-reload, which i put into the post install script
then the user just needs to enable the service and we're off to the races
	-> apparently arch/pacman et al don't want you to automatically start services with packages
	-> so the user has to do it themselves
	-> don't see the harm in following the same practice
