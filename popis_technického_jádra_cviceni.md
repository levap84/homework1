# Technické jádro cvičení

## PCAP: obsah a způsob vytvoření
- **Základní charakteristika:** PCAP zachycuje: 
    1) DNS dotaz na phishing homograficky podvrženou doménu (např. „micrοsоft‑update.com“ s cyrilickými „o“), 
    2) HTTP/HTTPS návštěvu podvrženého webu s TLS SNI odpovídajícím doméně, 
    3) OpenVPN handshake (TLS ClientHello/ServerHello, navázání tunelu), 
    4) SMB sezení na IP NAS (např. 10.10.20.15), včetně Tree Connect na sdílení „\NAS\Projects“, 
    5) velké datové přenosy (SMB Read/Write) v časech 21:30–23:00 a o víkendu.

- **Generování PCAP:** Nasimulováno v labu: místní DNS server odpovídá na dotaz škodlivé domény; OpenVPN server akceptuje přihlášení testovacího účtu; NAS poskytuje sdílení; klient provede kopírování velkých souborů (např. .zip, .docx, .xlsx). Zachycení pomocí tcpdump/wireshark na klientovi i VPN serveru; PCAP je následně vyčištěn od osobních dat a anonymizován (náhodné MAC).
- **IOC obsažené v PCAP:** Phishing doména, její IP (např. 203.0.113.45), SNI v TLS, NAS IP (10.10.20.15), názvy SMB sdílení, časová okna přenosů a objemy (součty v MB).

## Windows VM: perzistence a artefakty
- **Konfigurace VM:** Windows 11 Pro, lokální uživatel „analyst“ s admin právy, zapnutý Windows Event Logging, nainstalované Sysinternals (Autoruns), PowerShell 7. Síť izolovaná v labu.
- **Perzistenční artefakty:**
    - Run klíče: HKCU\Software\Microsoft\Windows\CurrentVersion\Run -> „UpdateAssist“ ukazuje na %AppData%\UpdateAssist\updateassist.exe.
    - Scheduled Task: „UpdateAssistDaily“ v \Microsoft\Windows\Maintenance, spouští skript %AppData%\UpdateAssist\assist.ps1 při logonu i denně v 22:00.
    - Služba: Volitelně „WinHelperSvc“ (Automatic), binárka v %ProgramData%\WinHelper\winhlp.exe.
    - Logy: Event ID 4698 (vytvoření úlohy), 106 (Task Scheduler – spuštění), 1006 (GroupPolicy – skript), 4688 (nový proces).
- **Ověření:** Restart VM, sledování Event Log a Process Explorer; hash (SHA‑256) binárek je předem známý pro kontrolu (referenční IOC).

## AD doména: stav, kompromitace a logy
- **Doména:** test.local; řadič domény (Domain Controller) DC1 (10.10.10.10); NAS joined to domain; účty: „jnovak“ (User), „svc_nas“ (Service), „adm1“ a „adm2“ (Domain Admins).
- **Kompromitace:** Phishing získal heslo „jnovak“. Útočník se přes VPN přihlásil a přistupoval k NAS s oprávněními „jnovak“. Následovaly pokusy o přihlášení k „svc_nas“ (Event ID 4625 selhání) a test RDP na workstation „WS‑QA‑02“ (Event ID 4624 úspěch, 4672 – speciální privilegia, pokud simulováno).
- **Logy:**
    - DC Security log: 4624 (Successful Logon) pro „jnovak“ z IP VPN, typ logonu 3 (network) a 10 (RemoteInteractive) při RDP; 4648 (logon with explicit credentials) při PSExec.
    - NAS audit: SMB přístupy, listování složek, kopie souborů, objem přenosu.
    - E‑mail brána: Záznamy o automatickém forwardu a příjmu phishing e‑mailu (simulované logy).

## VPN, NAS a e‑mail: konfigurace - protiopatření
- **VPN (OpenVPN):** Povolit MFA (např. TOTP – časový jednorázový kód), omezit přístup na pracovní hodiny, logovat SNI/DNS dotazy v tunelu, mapovat IP klientů.
- **NAS (SMB):** Aplikovat princip nejmenších oprávnění (least privilege), zapnout audit přístupu, omezit přístup ke sdílením podle skupin, blokovat velké přenosy mimo pracovní dobu (volitelně).
- **E‑mail (DMARC/DKIM/SPF):**
    - SPF (Sender Policy Framework): pravidlo určuje, které servery smí posílat e‑maily jménem domény.
    - DKIM (DomainKeys Identified Mail): podepisování e‑mailů klíčem pro ověření původu.
    - DMARC (Domain‑based Message Authentication, Reporting and Conformance): politika, co dělat s e‑maily neprocházejícími SPF/DKIM a reportování.
