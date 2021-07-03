# Installation podman on MacOS
### You can not just install Podman like Docker on MacOS or Windows. You can install official Podman client on MacOS. Podman server should be run on some Linux.
### In this example, we will configure Podman client and configure Podman server installing Fedora via Vagrant to VirtualBox. You can use Podman absolutely locally.

#### Create work dir:
```
mkdir fedora
cd fedora
```

#### Download and Install VirtualBox:
`brew install cask virtualbox`

#### Install Vagrant:
`brew install cask vagrant`

#### Install Podman:
`brew install podman`

#### Create Vagrantfile
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

#### Start Vagrant:
`vagrant up`

#### Create SSH-keys:
```
rm -rf /Users/"${USER}"/.ssh/known_hosts    # ATTENTION!
ssh-keygen -t rsa -b 4096 -C "podman" -f /Users/"${USER}"/.ssh/podman
ssh-copy-id -i /Users/"${USER}"/.ssh/podman.pub -p 2222 vagrant@127.0.0.1
```

#### Test SSH-connection and configure Podman Server:
```
# password: vagrant

ssh -p '2222' 'vagrant@127.0.0.1' systemctl --user enable podman.socket
ssh -p '2222' 'vagrant@127.0.0.1' sudo loginctl enable-linger $USER
```

#### Configure Podman client:
```
podman system connection list
podman system connection add --identity /Users/"${USER}"/.ssh/podman podman ssh://vagrant@127.0.0.1:2222
podman system connection list
```

#### Test works:
`podman info`
