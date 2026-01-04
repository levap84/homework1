# **Úkoly pro účastníky**

## Úkol 1: Analýza PCAP – phishing, VPN tunel a exfiltrace přes SMB
- **Vstupy:**   
    - Soubor PCAP
    - Seznam používaných protokolů – DNS, HTTP/HTTPS, TLS, SMB, OpenVPN
    - Seznam IP adres z firemní infrastruktury

- **Úkol:**   
*Najít v PCAP klíčové události: přístup na podvrženou doménu, ověření zda-li došlo k: získání přihlašovacích údajů (indikace v HTTP/HTTPS toku); navázání spojení VPN tunelu (OpenVPN/TLS); komunikaci se síťovým úložištěm NAS přes SMB a patrnoá exfiltrace (velké objemy přenosů v neobvyklých časech).*

    - Nalezení phishing domény – v PCAP prohledat DNS dotazy. Hledat doménu, která vypadá jako „microsoft-update“, ale má podvržené znaky.
    - Zjistit přihlášení na podvržený web – v komunikaci HTTPS/TLS ověřit, zda-li se počítač připojil na falešnou doménu.
    - Nalezení spojení na VPN – v PCAP ověřit, zda se navázalo spojení přes firemní openVPN
    - Najít komunikaci s NAS - Přes SMB protokol zjistit, zda-li se čtou nebo zapisují soubory.
    - Zjistit exfiltraci dat – ppodívat se, jestli se přenášelo hodně dat v časech, kdy zaměstnanec obvykle nepracuje (večer, víkend). To naznačuje, že útočník vynášel data ven.

- **Výstup:**  
    - Časová osa:
        - Kdy se objevil phishing doménový dotaz.
        - Kdy začalo VPN spojení.
        - Kdy se přistupovalo k NAS.
        - Kdy a kolik dat se přeneslo.
    - Seznam IOC (indikátorů kompromitace):
        - Název falešné domény.
        - IP adresa útočníka.
        - IP adresa NAS.
        - Hash souboru (otisk souboru, pokud je k dispozici).
    - Popis laterálního pohybu
    - Závěr o exfiltraci

- **Způsob hodnocení:** 
    - Ve zprávě je přesný název phishing domény.
    - Je tam čas začátku openVPN tunelu.
    - Je tam IP NAS a SMB sdílení.
    - Identifikované soubory přenosu
    - Jsou uvedeny 3+ IOC.
    - Je tam stručná analýza, že data odcházela mimo pracovní dobu.

- **Automatizace kontroly:**  
Vzhledem k tomu, že python umí číst pomocí knihoven soubor PCAP dalo by se porovnat, zdali účastník odpověděl správně na povinné položky včetně např. součtů přenosů dat z NAS, časových oken, nalezené závadné domény, SMB komunikace apod.


## Úkol 2: Ověření perzistence malwaru ve Windows
- **Vstupy:**  
    - Virtuální Windows počítač (VM), který je infikovaný s lokálním admin účtem pro analýzu
    - Nástroje: Autoruns, Event Viewer, PowerShell

- **Úkol:**  
Identifikovat mechanismus, kterým se malware udržuje po restartu: zjistit podezřelé Run klíče, naplánované úlohy, služby, inicializační skripty PowerShell, a ověřit, že po restartu se spouští škodlivý proces.

    - Co se spouští po startu:
        - V Autoruns zjistit přítomnost podezřelého programu
    - Naplánované úlohy:
        - V Task Scheduler zjistit, jestli se spouští nějaký skript nebo program každý den nebo při přihlášení
    - Služby:
        - V Services zjistit, jestli běží nějaká neznámá služba, která startuje automaticky
    - Ověření v logů událostí:
        - V Event Viewer najít záznamy, zda-li se program spustil po restartu
    - Hash souboru:
        - V PowerShellu vygenerovat otisk souboru, pro jednoznačnou identifikaci
    - Navrhnout odstranění:
        - Deaktivace položky, smazání a ověření, že po restartu se nespouští.

- **Výstup:**  
Stručná zpráva popisující nalezené položky (cesty k binárkám, názvy úloh), jejich čas vzniku, podpisy/hashe souborů, mechanismus perzistence + návrh bezpečného odstranění

    - obsah zprávy:
        - Název podezřelého programu nebo úlohy
        - Cesta k souboru
        - Hash souboru
        - Způsob perzistence
        - Události z logu dokazující spuštění po restartu
        - Návrh deaktivace a odstranění

- **Způsob hodnocení:**  
Zpráva musí uvést konkrétní klíče registru, přesný název naplánované úlohy, cestu ke škodlivému souboru, hash (SHA‑256) souboru a důkaz, že po restartu dochází k jeho spuštění např Event ID z logu. Splněno, pokud je mechanismus korektně popsán i s ověřením.

    - Ve zprávě je konkrétní název perzistence
    - Je tam cesta k souboru
    - Je tam hash souboru
    - Jsou tam události z logu, které dokazují spuštění po restartu
    - Je tam návrh odstranění 

- **Automatizace kontroly:**  
Pomocí python se zkontroluje závadná naplánovaná úloha její název případně ID a cesta závadné binárky, která se spouští po restartu. Následně se porovná s výstupem od účastníka. Dále by se dalo kontrolovat, zdali došlo k odstranění závadné úlohy (kontrola její existence po restartu)



## Úkol 3: Řešení kompromitovaných účtů v AD doméně
- **Vstupy:**  
Přístup k testovací AD doméně, exporty AD událostí, seznam účtů, NAS servisní účet, a doménový admin účet.  
Nástroje: Active Directory Users and Computers, PowerShell

    - Active Directory (AD) – systém ve Windows, který spravuje uživatele, počítače a jejich oprávnění.
    - Seznam účtů: běžný uživatel „jnovak“, servisní účet „svc_nas“, dva administrátorské účty „adm1“ a „adm2“.
    - Logy z Domain Controlleru - události o přihlášení
    - Event ID 4624 = úspěšné přihlášení.
    - Event ID 4625 = neúspěšné přihlášení.
    - Event ID 4648 = přihlášení s explicitními pověřeními (někdo zadal heslo ručně při vzdálené akci).

- **Úkol:**  
Zjistit, které účty byly zneužity, a provést bezpečnou změnu hesel. Nastavit MFA tam a zneplatnit aktivní tokeny/sessions


    - Zjisti, které účty byly zneužity:
        - V logu najít, že se „jnovak“ přihlásil z podezřelé IP
        - Podívat se na pokusy o neúspěšné přihlášení na NAS
        - Zkontrolovat, jestli se někdo pokoušel přihlásit jako admin
    - Deaktivovat aktivní relace:
        - Pokud je kompromitovaný účet právě přihlášený (např. přes RDP), force logoff
        - odříznutí útočníka
    - Reset hesel kompromitovaných účtů:
        - V AD nastavit kompromitovaným uživatelům reset (při dalším přihlášení změna hesla)
        - U admin účtů provést reset koordinovaně (podřezání větve)
    - Dočasné omezení přístupu kompro. účtů k síťovým komponentám např. NAS dokud nebude situace vyřešena
    - Aktivace MFA (vícefaktorové ověření) pokud dostupné

- **Výstup:**  
Seznam kompromitovaných účtů, důkazy (události s časem/IP), provedené kroky (reset hesla, force logoff, destroy sessions) a návrh dočasných omezení (přístup k NAS pro kompro. účty)


    - Seznam kompromitovaných účtů
    - Důkazy: konkrétní Event ID z logu, časy, IP adresy
    - Popsané kroky nápravy: odhlášení relací, reset hesel, vynucení změny hesla, omezení přístupu, zapnutí MFA.
    - Závěr: např. „Útočník zneužil účty jnovak,... pokusil se o získání přístupu k servisnímu účtu NAS v AD, který slouží ke komunikaci NAS vs Doména, provedl jsem reset hesel a zneaktivnil relace.“

- **Způsob hodnocení:**  
Musí být uvedeny správné Event ID a odpovídající časy/IP, korektně provedený reset hesel pro dotčené účty, potvrzení o vynucení změny při dalším přihlášení. Pokud reset u admin účtů tak uvést důvod. Splněno, pokud jsou kroky bezpečné a pořadí je logické (nejprve zneplatnit relace, pak reset).


    - Ve zprávě je správně uvedený kompromitovaný účet/y.
    - Jsou tam Event ID s časy a IP adresami.
    - Je tam reset hesla s vynucením změny při dalším přihlášení.
    - Je tam odhlášení aktivních relací.
    - Kroky jsou seřazeny logicky (nejdřív odříznout, pak resetovat, obnova spojení se síťovými komponenty).

- **Automatizace kontroly:**  

    powershell skript ověří:
    - zdali se u daných účtů změnil atribut čas poslední změny hesla
    - Ověří např., že už není aktivní RDP relace
    - Zkontroluje, že u účtů je nastaveno, že uživatel musí změnit heslo při přihlášení

## Úkol 4: Nápravná opatření – blokace IOC, zvýšení odolnosti VPN a NAS, prevence phishingu
- **Vstupy:**  
Seznam IOC z Úkolu 1–3 (domény, IP, hash), konfigurační přístup k testovacím komponentám: OpenVPN server, firewall, NAS (SMB nastavení), e‑mailová brána (simulovaná).

    - Seznam IOC - škodlivá doména, IP adresa útočníka, hash souboru
    - Přístup ke konfiguraci: DNS, firewall, OpenVPN server, NAS, e‑mailová brána
    - Cíl: Zavést praktická opatření, aby se útok už nemohl opakovat a aby se podobné hrozby daly rychleji odhalit

- **Úkol:**  
Blokovat známé IOC na DNS/firewallu, omezit VPN přístup (povinná MFA, časová okna), zvýšit odolnost NAS (nejmenší nutná oprávnění, omezení sdílení, nastavení auditů). Nastavit pravidla proti phishingu - posílení emailové ochrany.

    - Blokovat škodlivou doménu a IP adresu:
        - V DNS nastav, aby se dotazy na závadnou doménu směrovaly na NXDOMAIN (neexistující) 
        - Ve firewallu zablokuj komunikaci na IP útočníka
    - Zvýšit odolnost VPN přístupu:
        - Zapni MFA (vícefaktorové ověření)
        - Omezit přístup jen na pracovní dobu
        - Logování, kdo se připojuje a kdy
    - Zabezpečit NAS:
        - Odebrání zbytečných oprávnění
        - Zapnutí auditu – logování, kdo čte nebo zapisuje soubory
        - Omezení velkých přenosů mimo pracovní dobu
    - Posílení e‑mailovou ochranu proti phishingu:
        - SPF = seznam povolených serverů pro odesílání e‑mailů
        - DKIM = digitální podpis e‑mailu, který ověřuje jeho pravost
        - DMARC = politika, která určuje, co dělat s e‑maily, které neprojdou SPF/DKIM

- **Výstup:**  
Krátký „playbook“: co a kde nastavit, příklady konfigurací (např. blokace domény na DNS, povolení MFA na VPN, audit SMB – zapisování Event ID u přístupu), a seznam metrik pro ověření (snížení objemu provozu mimo pracovní dobu, zablokované dotazy na škodlivou doménu).

    - Playbook: stručný seznam kroků, co a kde nastavit
    - Příklady: blokace domény na DNS, pravidlo na firewallu, zapnutí MFA na VPN, audit na NAS, nastavení SPF/DKIM/DMARC
    - Metriky: jak poznat, že je to funkční – např. DNS logy ukazují blokované dotazy, firewall logy ukazují odmítnuté spojení, VPN logy ukazují MFA, NAS logy ukazují auditované přístupy, e‑mail brána odmítá podvržené zprávy

- **Způsob hodnocení:**  
Playbook je praktický a konzistentní s incidentem; obsahuje konkrétní kroky a příklady. Splněno, pokud konfigurační změny vedou k detekovatelné blokaci IOC a omezení rizikových přístupů.

    - Ve zprávě jsou konkrétní kroky
    - Je vidět funkčnost v logách
    - Opatření jsou navázaná na zjištěné IOC
    - Kroky dávají smysl a jsou proveditelné

- **Automatizace kontroly:**  

    Skript ověří:
    - Jestli DNS vrací „NXDOMAIN“ pro škodlivou doménu
    - Že firewall má pravidlo blokující IP útočníka
    - Zkontroluje, že VPN vyžaduje MFA
    - Ověří, že NAS loguje přístupy
    - Zkontroluje, že e‑mail brána odmítá zprávy, které neprojdou SPF/DKIM/DMARC

