# nixos

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
