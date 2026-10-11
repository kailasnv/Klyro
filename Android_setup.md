# Android Setup (Termux — Without Google Services)

Klyro can also be accessed from an Android device without relying on Google Play Services. Instead of using the official Tailscale Android app, this setup uses **Termux + Tailscale CLI** to connect the phone to the same private Tailscale network as the Arch Linux server.

This is useful for Android devices without Google apps or Google Play Services, including lightweight or custom Android installations.

### Requirements

- Android phone running a compatible version of Android.
- Termux installed from a trusted source.
- Internet connectivity.
- An existing Tailscale account and a tailnet containing the Klyro server.
- OpenSSH installed on Termux if SSH access is required.

### 1. Install Termux

Install Termux using one of the following sources:

- [Termux on F-Droid](https://f-droid.org/packages/com.termux/)
- [Termux GitHub releases](https://github.com/termux/termux-app/releases)

Avoid mixing Termux installations from different sources because their packages and signatures may differ.

Open Termux and update the package environment:

```bash
pkg update && pkg upgrade -y
pkg install curl openssh -y
```

### 2. Install Tailscale CLI

This setup uses the community-maintained [bropines/tailscale-termux-cli](https://github.com/bropines/tailscale-termux-cli) project, which provides Tailscale CLI and daemon packages for Termux.

Run the installer's command in Termux:

```bash
curl -fsSL https://raw.githubusercontent.com/bropines/tailscale-termux-cli/main/remote-install.sh | bash
```

Review the installer and project source before executing installation scripts from the internet.

### 3. Connect Android to the Tailnet

Start the Tailscale daemon if necessary, then initiate authentication:

```bash
tailscaled-start
tailscale up
```

Follow the authentication URL printed by the CLI. Open it in a browser and authenticate using an identity provider supported by your Tailscale account. Google Play Services are not required for this CLI-based approach.

Once authenticated, verify the connection:

```bash
tailscale status
tailscale ip -4
```

The phone should appear as a device in the same tailnet as the Linux server.

### 4. Enable SSH Access on Android

To access the phone remotely from another tailnet device, enable the SSH server in Termux.

Set a password for your Termux user:

```bash
passwd
```

Start the SSH daemon on the default Termux SSH port:

```bash
sshd
```

Check the current Termux username:

```bash
whoami
```

Get the phone's Tailscale IPv4 address:

```bash
tailscale ip -4
```

From the Arch Linux computer, connect using:

```bash
ssh -p 8022 <termux-username>@<android-tailscale-ip>
```

Replace the placeholders with the username returned by `whoami` and the phone's actual Tailscale IP.

**Security note:** Prefer SSH key authentication and avoid exposing SSH to the public internet. Tailscale provides the private network path between your devices.

### 5. Start SSH Automatically After Reboot (Optional)

Install the Termux:Boot add-on from the same trusted distribution source as Termux:

- [Termux:Boot](https://github.com/termux/termux-boot)

Create the boot script directory and script:

```bash
mkdir -p ~/.termux/boot
nano ~/.termux/boot/start-ssh
```

Add the following:

```bash
#!/data/data/com.termux/files/usr/bin/sh

termux-wake-lock
tailscaled-start
sshd
```

Make the script executable:

```bash
chmod +x ~/.termux/boot/start-ssh
```

Open Termux:Boot once after installation and reboot to test the setup. Android battery-management policies may still affect background operation.

### 6. Access Klyro from Android

Make sure the Klyro server is running on the Arch Linux computer and that both devices are connected to the same tailnet.

Open a browser on Android and navigate to:

```text
http://<klyro-server-tailscale-ip>:5000
```

Replace the placeholder with the Tailscale IPv4 address of the Arch Linux server.

You can also SSH from Android to the Arch Linux machine if its SSH service is configured and permitted by your tailnet access rules.
 d` | Set or change the Termux user's password |

### Notes

- This configuration uses Termux's userspace Tailscale implementation; root access is not required.
- The phone and Klyro server must be connected to the same tailnet, and the relevant access policies must permit the connection.
- Android may suspend background processes to save battery. Termux:Boot and `termux-wake-lock` can help, but do not guarantee uninterrupted connectivity.
- The Tailscale CLI implementation is maintained by a third party and is separate from the official Tailscale Android application.

 
---