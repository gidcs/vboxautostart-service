# vboxautostart-service
vboxautostart-service for Fedora

## Setup vboxautostart
```
# set environment variables
sudo tee -a /etc/default/virtualbox <<'EOF'
VBOXAUTOSTART_DB=/etc/vbox
VBOXAUTOSTART_CONFIG=/etc/vbox/vboxauto.conf
EOF

# set vboxautostart policy
sudo tee -a /etc/vbox/vboxauto.conf << "EOF"
default_policy = deny
$USER = {
	allow = true
}
EOF

sudo chgrp vboxusers /etc/vbox
sudo chmod 1775 /etc/vbox
sudo usermod -aG vboxusers $USER
VBoxManage setproperty autostartdbpath /etc/vbox

# replace the original vboxautostart-service.sh
sudo wget -O /usr/lib/virtualbox/vboxautostart-service.sh "https://raw.githubusercontent.com/gidcs/vboxautostart-service/master/vboxautostart-service.sh"

# install selinux policy
wget -O my-vboxautostarts.te -c "https://raw.githubusercontent.com/gidcs/vboxautostart-service/master/my-vboxautostarts.te"
checkmodule -M -m -o my-vboxautostarts.mod my-vboxautostarts.te
sudo semodule_package -o my-vboxautostarts.pp -m my-vboxautostarts.mod
sudo semodule -X 300 -i my-vboxautostarts.pp
rm my-vboxautostarts.*

sudo systemctl enable vboxautostart-service # maybe enabled already
sudo reboot
```

## Enable autostart for Your VM
```
VBoxManage -nologo list vms
VM="Name_Of_Your_VM"
VBoxManage modifyvm "${VM}" --autostart-enabled on --autostop-type acpishutdown
sudo systemctl stop vboxautostart-service
sudo systemctl start vboxautostart-service
```
