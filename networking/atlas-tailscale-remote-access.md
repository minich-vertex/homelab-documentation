# Remote Access to Ubuntu Server with Tailscale

This guide documents how to connect an Ubuntu Server VM to a private
Tailscale network and access it remotely over SSH.

The example server in this walkthrough is **Atlas**.

## 1. Update Ubuntu

Log in to the Ubuntu Server VM and update its package lists and
installed packages:

``` bash
sudo apt update
sudo apt upgrade -y
```

Confirm that the server has internet connectivity:

``` bash
ping -c 4 google.com
```

## 2. Install Tailscale

Install Tailscale using its official installation script:

``` bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Enable and start the Tailscale service:

``` bash
sudo systemctl enable --now tailscaled
```

## 3. Connect the server to Tailscale

Run:

``` bash
sudo tailscale up
```

Open the authentication URL printed in the terminal, sign in to your
Tailscale account, and authorize the server.

## 4. Verify the connection

Check the Tailscale service:

``` bash
sudo systemctl status tailscaled
```

Check that the service is configured to start automatically after a
reboot:

``` bash
sudo systemctl is-enabled tailscaled
```

Expected output:

``` text
enabled
```

Display the server's Tailscale IPv4 address:

``` bash
tailscale ip -4
```

The address will normally be in the `100.x.x.x` range. Use the actual
address returned by your server; do not copy an example address.

You can also view connected Tailscale devices with:

``` bash
tailscale status
```

## 5. Install Tailscale on your Windows computer

1.  Download the Windows client from
    [Tailscale](https://tailscale.com/download/windows).
2.  Install and open it.
3.  Sign in to the **same Tailscale account** used to authorize Atlas.
4.  Confirm that both devices appear in your Tailscale device list.

## 6. Enable SSH on Ubuntu

Install the OpenSSH server if it is not already installed:

``` bash
sudo apt install openssh-server -y
```

Enable and start SSH:

``` bash
sudo systemctl enable --now ssh
```

Verify that it is running:

``` bash
sudo systemctl status ssh
```

## 7. Connect remotely from Windows

Open PowerShell or Windows Terminal and run:

``` powershell
ssh <ubuntu-username>@<tailscale-ip>
```

Replace:

-   `<ubuntu-username>` with the Linux username created during Ubuntu
    installation.
-   `<tailscale-ip>` with the address returned by `tailscale ip -4` on
    Atlas.

For example, if the username is `olnist`, the command format is:

``` powershell
ssh olnist@100.x.x.x
```

Accept the host-key prompt when connecting for the first time, then
authenticate using your Ubuntu account.

## 8. Reboot behaviour

Tailscale is configured to start automatically because its service is
enabled:

``` bash
sudo systemctl is-enabled tailscaled
```

For unattended recovery after a power interruption, also configure the
Proxmox VM to **Start at boot**:

1.  Select the Atlas VM in Proxmox.
2.  Open **Options**.
3.  Enable **Start at boot**.

The physical server must also power back on after electricity is
restored. If the hardware supports it, configure its BIOS/UEFI
power-recovery setting to restore power after an AC loss.

## What this setup provides

-   A stable Tailscale address for Atlas, even when the ISP changes the
    home's public IP.
-   Encrypted connectivity between devices on the same Tailscale
    network.
-   Remote SSH access without exposing the SSH port directly to the
    public internet.
-   Automatic Tailscale service startup when Ubuntu boots.

**Scope note:** Installing Tailscale on Atlas connects Atlas itself. It
does not automatically provide access to every device on the home LAN or
the Proxmox host. Whole-LAN access requires a separately configured and
approved Tailscale subnet router.

## References

-   [Tailscale: Install on
    Linux](https://tailscale.com/docs/install/linux)
-   [Tailscale: Install on
    Windows](https://tailscale.com/download/windows)
-   [Tailscale: Subnet routers](https://tailscale.com/kb/1019/subnets/)
