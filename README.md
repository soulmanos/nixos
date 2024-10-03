# nixos

## [Install on Hetzner Cloud](https://nixos.wiki/wiki/Install_NixOS_on_Hetzner_Cloud#:~:text=From%20NixOS%20minimal%20ISO%201%20Create%20a%20new,x86%20instances%205%20Unmount%20the%20ISO%20and%20reboot)

> [Script to do all this and more](https://gist.github.com/thilobillerbeck/b15edf42886be7e5004265e8771608ce) e.g. add ssh key

> [Script to install BUT with disk encryption](https://gist.github.com/alicebob/ef9b162adc0760683209508daeaa7278)

```
sudo -i
parted /dev/sda -- mklabel msdos
parted /dev/sda -- mkpart primary 1MB -8GB
parted /dev/sda -- set 1 boot on
parted /dev/sda -- mkpart primary linux-swap -8GB 100%
mkfs.ext4 -L nixos /dev/sda1
mkswap -L swap /dev/sda2
mount /dev/disk/by-label/nixos /mnt
nixos-generate-config --root /mnt
# Think it might make sense to add more in here before bootstrapping flakes e.g. a user other than root, enable ssh etc
sed -i 's/# boot.loader.grub.device/boot.loader.grub.device/g' /mnt/etc/nixos/configuration.nix
sed -i 's/# networking.networkmanager.enable/networking.networkmanager.enable/g' /mnt/etc/nixos/configuration.nix
sed -i 's/# time.timeZone/time.timeZone/g' /mnt/etc/nixos/configuration.nix
sed -i 's/Amsterdam/London/g' /mnt/etc/nixos/configuration.nix
sed -i 's/# services.openssh.enable/services.openssh.enable/g' /mnt/etc/nixos/configuration.nix
sed -i '94i\   services.openssh.settings.PermitRootLogin = \"yes\";' /mnt/etc/nixos/configuration.nix
sed -i '98i\   networking.firewall.allowedTCPPorts = [ 22 ];' /mnt/etc/nixos/configuration.nix
nixos-install
reboot
```

- Enable experimental features so we can use flakes
  
```
cd ~
mkdir -p ./.config/nix/
cat << EOM >> ./.config/nix/nix.conf
experimental-features = nix-command flakes
EOM
sudo nixos-rebuild switch
nix show-config | grep experimental-features # check experimental features is enabled
nix-shell -p vim
```

- install `jq` from a flake

```
nix profile install github:NixOS/nixpkgs#jq
nix profile list
```

## Existing `/etc/nixos/configuration.nix`

```
{ config, lib, pkgs, ... }:
{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];
  nix.settings.experimental-features = [ "nix-command" "flakes" ];
  boot.loader.grub.enable = true;
  boot.loader.grub.device = "/dev/sda"; # or "nodev" for efi only
  networking.hostName = "nix1"; # Define your hostname.
  time.timeZone = "Europe/London";

  users.users.as = {
    isNormalUser = true;
    extraGroups = [ "wheel" "docker" ]; # Enable ‘sudo’ for the user.
    packages = with pkgs; [
      tree
      vim
      mlocate
      neofetch
      git
      jq
    ];
  };
  environment.systemPackages = with pkgs; [
    vim # Do not forget to add an editor to edit configuration.nix! The Nano editor is also installed by default.
    wget
    tree
    curl
    tmux
  ];
  security.sudo.extraRules= [
    {
      users = [ "as" ];
      commands = [
        {
          command = "ALL" ;
          options= [ "NOPASSWD" ]; # "SETENV" # Adding the following could be a good idea
         }
      ];
    }
  ];
  services.openssh.enable = true;
  services.openssh.settings.PermitRootLogin = "yes";
  services.locate.enable = true;
  services.locate.package = pkgs.mlocate;
  services.locate.localuser = null;
  virtualisation.docker.enable = true;
  networking.firewall.allowedTCPPorts = [ 22 ];
  system.stateVersion = "24.05"; # Did you read the comment?
}
```
