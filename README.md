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
