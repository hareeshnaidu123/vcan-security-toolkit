# vcan-security-toolkit
Automotive CAN Bus Security Analysis & Defensive Engineering LabAn end-to-end, zero-hardware automotive cybersecurity laboratory designed for security researchers, embedded systems engineers, and automotive penetration testers. This project virtualizes an in-vehicle Controller Area Network (CAN) bus to demonstrate real-time telemetry broadcast, frequency-based anomaly detection, and cryptographic message authentication.DisclaimerIMPORTANT NOTICE:This project is designed strictly for educational, developmental, and defensive research purposes. The techniques, code, and architectural concepts demonstrated here are intended to assist security engineers in auditing and securing embedded automotive platforms. Do not deploy automated injection, replay attacks, or untrusted tools on live vehicles, production networks, or safety-critical drive-by-wire hardware. The authors assume no liability for misuse or damage resulting from the use of this software.Technologies UsedLanguage: Python 3.9+Automotive Networking: Linux Kernel SocketCAN (CAN 2.0B / vcan)Libraries: python-can (CAN interface abstraction)Cryptographic Primitives: HMAC-SHA256 (Truncated CMAC/HMAC model conforming to AUTOSAR SecOC specifications)Diagnostics & Analysis Utilities: can-utils (candump, cansniffer, canplayer)FeaturesVirtual Powertrain Simulator: Emulates real-time cyclic traffic for multiple Electronic Control Units (ECUs), including Engine Control Module (RPM) and Anti-lock Braking / Transmission Control (Vehicle Speed).Timing-Based Intrusion Detection System (IDS): Monitors message inter-arrival times ($\Delta t$), baseline jitter, frame starvation, and unauthorized arbitration IDs.AUTOSAR SecOC Emulation: Features Secure Onboard Communication mechanisms incorporating truncated Message Authentication Codes (MACs) and monotonically increasing freshness counters to mitigate replay and bit-flipping attacks.Zero Hardware Requirement: Completely self-contained within Linux Kernel Virtual CAN (vcan0) interfaces—no costly adapters (e.g., PEAK-System, ValueCAN, or CANable) required.Learning ObjectivesBy working through this laboratory, you will:Understand the fundamental structure of standard CAN 2.0B frames (Arbitration IDs, DLC, Data Payloads).Identify why native CAN bus networks lack built-in authentication, confidentiality, and anti-replay safeguards.Build and tune statistical and timing-based baseline rules for vehicular Intrusion Detection Systems (IDS).Implement and evaluate AUTOSAR SecOC (Secure Onboard Communication) concepts to enforce payload integrity and sequence freshness.Practice using native Linux CLI diagnostic tools (candump, cansniffer) to reverse-engineer unknown automotive telematics.Architecture & Frame LayoutStandard Telemetry FramesFrame NameArbitration IDCycle TimePayload DefinitionENGINE_RPM0x11020 ms2-byte unsigned integer (Big Endian), 6 bytes reservedVEHICLE_SPEED0x12050 ms2-byte scaled integer (speed * 100), 6 bytes reservedBRAKE_STATUS0x200100 ms1-byte digital status maskSecOC Authenticated Frame Format (0x300)Plaintext+-----------------------+--------------------------+-----------------------+
|  Payload Data (4 B)   | Freshness Counter (2 B)  |  Truncated MAC (2 B)  |
+-----------------------+--------------------------+-----------------------+
| Byte 0  1  2  3       | Byte 4  5                | Byte 6  7             |
+-----------------------+--------------------------+-----------------------+
Installation & Prerequisites1. System RequirementsUbuntu 20.04 LTS / 22.04 LTS / 24.04 LTS (Native or WSL2)Root/sudo privileges (required for loading kernel network drivers)2. Install Host DependenciesBashsudo apt update
sudo apt install -y can-utils git python3 python3-pip
3. Clone Repository & Install Python PackagesBashgit clone https://github.com/<YOUR-USERNAME>/can-security-lab.git
cd can-security-lab
pip3 install -r requirements.txt
Usage GuideThe suite runs via a consolidated CLI dispatcher (can_security_lab.py).Step 1: Initialize Virtual CAN InterfaceBring up the Linux kernel virtual CAN driver (vcan0):Bashsudo python3 can_security_lab.py --setup
Verification: Run ip link show vcan0—the status should report state UNKNOWN or UP.Step 2: Start Background Vehicle TelemetryIn your primary terminal, launch the simulated powertrain bus:Bashpython3 can_security_lab.py --sim
Verification: Open a second terminal and monitor live bus traffic:Bashcandump vcan0
Step 3: Run the In-Line Defensive IDS EngineIn a separate terminal, deploy the anomaly detector:Bashpython3 can_security_lab.py --ids
The engine enforces transmission window baselines. Any frame injection, flood attack, or spoofed ID will trigger an immediate alert:Plaintext[!] IDS ALERT: Injection detected on 0x110 (ENGINE_RPM) | dt=0.0021s (Min: 0.0050s)
[!] IDS ALERT: Unregistered Arbitration ID observed: 0x333
Step 4: Validate Cryptographic Defenses (SecOC)To observe cryptographic verification and replay rejection:Bashpython3 can_security_lab.py --secoc-demo
Expected Output:Plaintext[*] Running Secure Onboard Communication (SecOC) validation test...

1. Transmitting authentic frame (Counter: 100, Hex: 0102030400643a12)
   -> Verification Result: PASS (Authenticated)

2. Replaying the identical frame (Simulated replay attack)...
   -> Verification Result: FAIL (Replay detected (Counter 100 <= 100))

3. Altering payload in transit (Counter: 101, Hex: ff02030400659b81)
   -> Verification Result: FAIL (Cryptographic MAC signature mismatch)
Repository StructurePlaintextcan-security-lab/
├── can_security_lab.py    # Core suite: Simulator, IDS monitor, SecOC engine
├── requirements.txt       # Python dependencies (python-can)
├── LICENSE                # Open-source license (MIT)
└── README.md              # Project documentation and guide
ContributingPull requests are welcome. For major architectural additions (e.g., UDS diagnostics handlers, DoIP simulations, or CAN-FD expansion), please open an issue first to discuss the intended design.LicenseDistributed under the MIT License. See LICENSE for more information.
