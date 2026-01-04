# Úvaha nad náročností a úpravami pro různé úrovně
**Pro juniory**
- Zjednodušení:
- PCAP: Přidat „hinty“ (nápovědy) – předpřipravené filtry Wireshark (dns, tls, openvpn, smb2) a seznam hledaných položek.
- Windows: Poskytnout seznam typických míst perzistence a příkladový postup s Autoruns; přidat screenshoty.
- AD: Připravit šablonu reportu s předvyplněnými záhlavími „Event ID“, „Account“, „IP“.
- Cíl: Soustředit se na rozpoznání artefaktů a sepsání jasné časové osy; méně nástrojů, více vedení.

**Pro pokročilé**
- Zvýšení obtížnosti:
- PCAP: Přidat šifrovaný phishing přes HTTPS bez jasného obsahu, nutnost použít SNI, JA3 fingerprinting (otisk TLS klienta).
- Windows: Více perzistenčních vrstev (Scheduled Task + WMI Event Subscription + Service); skrýt binárky v neobvyklých cestách.
- AD: Simulovat krádež tokenů (Pass‑the‑Ticket), otestovat detekci laterálního pohybu přes WinRM/SMB a psát detekční pravidla (Sigma) pro logy.
- Cíl: Komplexní korelace více zdrojů, návrh detekcí a preventivních politik (MFA povinně, segmentace sítě).
