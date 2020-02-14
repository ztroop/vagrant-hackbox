Vagrant.configure("2") do |config|

  config.vm.box = "archlinux/archlinux"

  if Vagrant.has_plugin?("vagrant-disksize")
    config.disksize.size = '40GB'
  end

  config.vm.provider "virtualbox" do |vb|
    vb.name = "archlinux"
    vb.gui = true
    vb.customize ["modifyvm", :id, "--monitorcount", "1"]
    vb.customize ["modifyvm", :id, "--cpus", 2]
    vb.customize ["modifyvm", :id, "--vram", "32"]
    vb.memory = "4096"

    unless File.exist?('./data.vdi')
      vb.customize ['createhd', '--filename', './data.vdi', '--size', 40 * 1024]
    end

    vb.customize ['storageattach', :id,  '--storagectl', 'IDE Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', './data.vdi']
  end

  # virtualbox-guest-utils
  config.vm.provision "shell", inline: "pacman -Rs --noconfirm virtualbox-guest-utils-nox || true"
  config.vm.provision "shell", inline: "pacman -Syu --noconfirm virtualbox-guest-utils xf86-video-vmware"

  # yay
  config.vm.provision "Yay Requirements", type: "shell", inline: <<-SHELL
    pacman -S --noconfirm --needed --noprogressbar base-devel git
  SHELL

  config.vm.provision "Yay", type: "shell", privileged: false, inline: <<-SHELL
    cd /tmp
    git clone https://aur.archlinux.org/yay.git
    cd yay
    makepkg -si --noconfirm
    cd ..
    rm -rf /tmp/yay
  SHELL

  # Docker
  config.vm.provision "Docker", type: "shell", inline: <<-SHELL
    pacman -S --noconfirm --needed --noprogressbar docker
    systemctl enable docker
    usermod -aG docker vagrant
  SHELL

  # Locale
  config.vm.provision "Locale", type: "file", source: "data/locale.gen", destination: "/tmp/locale.gen"
  config.vm.provision "Locale Generate", type: "shell", inline: <<-SHELL
    mv /tmp/locale.gen /etc/locale.gen
    locale-gen
  SHELL

  # KDE
  config.vm.provision "SDDM Configuration", type: "file", source: "data/sddm.conf", destination: "/tmp/sddm.conf"
  config.vm.provision "SDDM", type: "shell", inline: <<-SHELL
    mv /tmp/sddm.conf /etc/sddm.conf
    pacman -S --noconfirm --needed --noprogressbar xorg-server plasma-desktop sddm ark dolphin kate konsole
    systemctl enable sddm
    localectl --no-convert set-x11-keymap us
  SHELL

  # Applications
  config.vm.provision "Firefox", type: "shell", inline: "pacman -S --noconfirm --noprogressbar firefox"
  config.vm.provision "Add Tool List", type: "file", source: "data/package.list", destination: "/tmp/package.list"
  config.vm.provision "Install Tools", type: "shell", inline: "pacman -S --noconfirm --noprogressbar $(cat /tmp/package.list |xargs)"

end
