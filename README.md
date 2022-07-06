# installation podman on MacOS

you can not install Podman on MacOS via popular way as Brew or drag-and-dropping .app file to Applications dir, for example. You can install official Podman __client__ on MacOS, but podman server is another component and should be run on some Linux.

so, let's start to configure Podman client and Podman server. We will configure Podman server installing Fedora via Vagrant to VirtualBox. So, in result, you will can use Podman absolutely locally.

#### create work dir:
```
mkdir fedora
cd fedora
```

#### download and Install VirtualBox:
`brew install cask virtualbox`

#### install Vagrant:
`brew install cask vagrant`

#### install Podman:
`brew install podman`

#### create Vagrantfile
```
cat <<EOF > Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "fedora/34-cloud-base"

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    dnf update
    dnf install podman -y
  SHELL
end
EOF
```

#### start Vagrant:
`vagrant up`

#### create SSH-keys:
```
rm -rf /Users/"${USER}"/.ssh/known_hosts    # ATTENTION!
ssh-keygen -t rsa -b 4096 -C "podman" -f /Users/"${USER}"/.ssh/podman
ssh-copy-id -i /Users/"${USER}"/.ssh/podman.pub -p 2222 vagrant@127.0.0.1
```

#### test SSH-connection and configure Podman server:
```
# password: vagrant

ssh -p '2222' 'vagrant@127.0.0.1' systemctl --user enable podman.socket
ssh -p '2222' 'vagrant@127.0.0.1' sudo loginctl enable-linger $USER
```

#### configure Podman client:
```
podman system connection list
podman system connection add --identity /Users/"${USER}"/.ssh/podman podman ssh://vagrant@127.0.0.1:2222
podman system connection list
```

#### test works:
`podman info`

### links:
https://www.redhat.com/sysadmin/podman-clients-macos-windows
