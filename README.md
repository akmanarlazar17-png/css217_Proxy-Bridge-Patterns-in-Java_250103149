#Lazar Akmanar 04-N
Overview

This repository contains six runnable Java programs that demonstrate:

Pattern	Problem it solves	Tasks
Proxy	Control or mediate access to an object without changing its interface	1, 2, 3
Bridge	Decouple an abstraction from its implementation so both can vary independently	4, 5, 6
Each task is a single file with its own main() method and produces console output that matches the lab specification exactly. No build tool or external library is required.

Requirements

Tool	Version	Notes
JDK	17 or later	JDK 8+ will also work; no preview features used
Terminal / IDE	any	IntelliJ IDEA, Eclipse, VS Code, or plain javac
Verify your install:

bash
java -version
javac -version
Project Structure

text
proxy-bridge-lab/
├── README.md
├── src/
│   ├── Task1Main.java    # Protection proxy (role guard + lazy loading)
│   ├── Task2Main.java    # Auditing proxy (view counter)
│   ├── Task3Main.java    # Firewall proxy (blocklist)
│   ├── Task4Main.java    # Bridge: new implementer (SmartLight)
│   ├── Task5Main.java    # Bridge: new abstraction (PartyRemote)
│   └── Task6Main.java    # Bridge: RPG hero/weapon system
└── out/                  # compiled .class files (git-ignored)
Getting Started

Option A — Compile and run a single task

bash
# from the repository root
javac -d out src/Task1Main.java
java -cp out Task1Main
Option B — Compile everything at once

bash
javac -d out src/*.java
Important: Tasks 4 and 5 both declare types named Device and BasicRemote. Compiling all files into a single output directory will cause a duplicate class error. See Troubleshooting — either compile them separately or move each task into its own package/directory.
Option C — IDE

Create a new Java project.
Copy the six .java files into src/.
Run the desired TaskXMain class directly.
Tasks

Task 1 — The Bouncer (Protection Proxy)

File: src/Task1Main.java

A ProxyImage wraps a RealImage and checks the caller's role before any expensive work happens.

Key mechanism: the guard clause returns immediately for non-ADMIN roles, so RealImage is never instantiated and no disk I/O occurs.

java
if (!"ADMIN".equals(userRole)) {
    System.out.println("[ACCESS DENIED] You lack permission to view: " + fileName);
    return;
}
if (realImage == null) realImage = new RealImage(fileName);   // lazy loading
realImage.show();
Run:

bash
javac -d out src/Task1Main.java && java -cp out Task1Main
Expected output:

text
[ACCESS DENIED] You lack permission to view: salary_sheet.png
--> [SLOW] Reading salary_sheet.png from hard drive...
Displaying salary_sheet.png on screen!
Task 2 — Auditing Proxy (Video View Counter)

File: src/Task2Main.java

ProxyVideo counts how many times a video is played and logs an audit line on every call, while the underlying RealVideo is created only once.

Key mechanism: the counter lives in the proxy, keeping the real object unaware of auditing.

java
viewCount++;
System.out.println("--> [AUDIT] Video has been played " + viewCount + " time(s).");
if (realVideo == null) realVideo = new RealVideo(fileName);
realVideo.play();
Run:

bash
javac -d out src/Task2Main.java && java -cp out Task2Main
Expected output:

text
--> [AUDIT] Video has been played 1 time(s).
--> [DOWNLOADING] Connecting to server & buffering 4K video: Cat_Memes.mp4
Playing video: Cat_Memes.mp4
--> [AUDIT] Video has been played 2 time(s).
Playing video: Cat_Memes.mp4
--> [AUDIT] Video has been played 3 time(s).
Playing video: Cat_Memes.mp4
Task 3 — School Wi-Fi Firewall (Mini-Challenge)

File: src/Task3Main.java

A complete proxy built from scratch: Internet (subject), RealInternet (real subject), SchoolProxyInternet (proxy) with a blocklist policy.

Key mechanism: blocked sites are rejected before the real internet object is ever created.

java
if ("gaming.com".equals(site) || "socialmedia.com".equals(site)) {
    System.out.println("[BLOCKED BY FIREWALL] Cannot access: " + site);
    return;
}
if (realInternet == null) realInternet = new RealInternet();
realInternet.connectTo(site);
Run:

bash
javac -d out src/Task3Main.java && java -cp out Task3Main
Expected output:

text
Connected to site: wikipedia.org
[BLOCKED BY FIREWALL] Cannot access: gaming.com
Task 4 — Smart Light (Bridge: Implementer Extension)

File: src/Task4Main.java

Adds a new SmartLight implementer to an existing device hierarchy and pairs it with an unmodified AdvancedRemote.

Key mechanism: zero lines of remote-controller or Device interface code were changed. Only a new class was added — satisfying the Open/Closed Principle.

Run:

bash
javac -d out src/Task4Main.java && java -cp out Task4Main
Expected output:

text
Smart Light glows bright white.
[Remote] Mute button pressed!
Smart Light brightness set to: 0%
Task 5 — Party Remote (Bridge: Abstraction Extension)

File: src/Task5Main.java

Adds a new PartyRemote abstraction extending BasicRemote, demonstrating the other extension axis of the Bridge: the abstraction grows while device code stays untouched.

Key mechanism: partyMode() composes existing delegated calls (turnOn, setVolume) without knowing anything about Radio.

java
public void partyMode() {
    System.out.println("[Party Remote] Pumping up the party!");
    device.turnOn();
    device.setVolume(100);
}
Run:

bash
javac -d out src/Task5Main.java && java -cp out Task5Main
Expected output:

text
[Party Remote] Pumping up the party!
Radio starts playing static/music.
Radio volume knob turned to: 100
Task 6 — RPG Weapon System (Mini-Challenge)

File: src/Task6Main.java

A full Bridge built from scratch: Weapon implementers (Sword, MagicWand) and Hero abstractions (Knight, Wizard).

Key mechanism: composition replaces combinatorial inheritance, collapsing M × N classes into M + N and allowing any hero to wield any weapon — including at runtime.

Run:

bash
javac -d out src/Task6Main.java && java -cp out Task6Main
Expected output:

text
Slash! Dealing 20 physical damage.
Zap! Dealing 35 magic damage.
Pattern Comparison

Both patterns use composition and delegation, but serve different intents:

Aspect	Proxy	Bridge
Intent	Control access to an object	Decouple abstraction from implementation
Interfaces	Proxy and real subject share the same interface	Abstraction and implementer have different interfaces
Cardinality	Typically 1:1	Typically 1:N
Client awareness	Client often unaware of the proxy	Client aware of both sides
Typical uses	Lazy loading, security, auditing, caching	Multi-platform UIs, drivers, game systems
Extension axis	Adds behaviour around an object	Adds a second dimension of variation
Rule of thumb: if the wrapper's method signatures are identical to the wrapped object's, it's likely a Proxy; if it translates between two vocabularies (remote "mute" → device "setVolume"), it's likely a Bridge.

