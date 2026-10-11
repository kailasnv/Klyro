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

---

Add auto start klyro service on boot on android.
Also start ssh service on boot so its can be remotly controlled and managed

### 4. Now start the server on boot

inside android Termux, Create the boot script directory and script:

    mkdir -p ~/.termux/boot
    nano ~/.termux/boot/start-klyro

Add the following:

    #!/data/data/com.termux/files/usr/bin/sh

    termux-wake-lock

    cd "$HOME/Klyro" || exit 1

    mkdir -p "$HOME/Music"

    exec python app.py >> "$HOME/klyro.log" 2>&1

Make the script executable:

chmod +x ~/.termux/boot/start-klyro

Open Termux once after installation and reboot to test the setup.  

### 5. Access server from other devices

Access locally: (from any device on the wifi network)

    http://<phone-ip>:5000

Acess from anywhere (using tailscale and configured tailnet)

    http://<klyro-server-tailscale-ip>:5000

Replace the placeholder with the Tailscale IPv4 address of the android mobile server.