# Intercept-Flutter-Apps
# Intercepting traffic on Android and iOS Flutter applications


## Android:


**1. ProxyDroid:**
- First and easiest way is using the **ProxyDroid** application and set up the proxy settings on it. The application can be found on Play Store.
- In Host: insert the local IP address of the machine that hosts Burpsuite (if a VM, set it in bridge and find the IP with `ifconfig` -> eth0).
- Port: `8080` (or whatever port is set up on Burp listener)
- Proxy Type: `HTTP`
- Enable Global Proxy (this setting needs root permission).
- From Burp: enable a listener on all interfaces on port 8080, and enable invisible proxy (Proxy settings -> edit listener -> Request handling -> flag Support invisible proxying)
- Once you enable the proxy on the application, you can intercept HTTP requests of your Flutter app.

- ## iOS:

**1. OpenVPN :**

Intercepting HTTP connections on iOS is more complicated since you can't use iptables on the device. Instead, you can use OpenVPN and run a VPN server on your Kali machine, connecting the iOS device to the VPN.

1. Run the following commands on Kali:
    ```shell
    wget https://git.io/vpn -O openvpn-install.sh
    sed -i "$(($(grep -ni "debian is too old" openvpn-install.sh | cut  -d : -f 1)+1))d" ./openvpn-install.sh
    chmod +x openvpn-install.sh 
    sudo ./openvpn-install.sh
    ```
   - Options:
     - Which IPv4 address should be used? [choose your local IP address]
     - This server is behind NAT. What is the public IPv4 address or hostname? Public IPv4 address / hostname [still you local IP address]
     - Which protocol should OpenVPN use? 1 [UDP]
     - What port should OpenVPN listen to? Port [1194]: 1194
     - Select a DNS server for the clients: 3 [I personally chose 1.1.1.1]
     - Enter a name for the first client: [choose a name]


2. Confirm the setup by running `ifconfig` and observing the addition of a `tun0` interface.

3. Start the OpenVPN service with `sudo service openvpn start`.

4. To install the OpenVPN client on iPhone, start a Python HTTP server in the client folder (/root by default):
    ```shell
    sudo python3 -m http.server 8080 --directory /root/
    ```
   - Navigate to *kalilocalip*:8080 on your iPhone with a browser and download the `.ovpn` file.

5. Open the file in the download folder with the OpenVPN app and add the configuration. Connect to the VPN.

6. You can navigate, but to intercept requests, set rules with iptables on Kali:
    ```shell
    sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 192.168.1.131 <KalilocalIPaddress>
    ```
    
   - Intercept requests with Burp on port 443 and enable invisible proxy from the proxy settings.

