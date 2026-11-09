# Připsáno – adaptér

**Adaptér propojuje ERP s aplikací [Připsáno](https://pripsano.cz/).** Doklady k úhradě (typicky faktury na dobírku nebo
k platbě na místě) předá z ERP do Připsáno a výsledek — zaplaceno převodem nebo QR platbou, převzato
hotově, zboží vydáno, zrušeno — zapíše zpět k dokladu v ERP.

Tento repozitář obsahuje jen **vydání** (instalátor a kontrolní součet).

➡ **[Poslední vydání](../../releases/latest)**

---

## Co adaptér dělá

```
 ERP (MS SQL Server) ◄──► adaptér (služba Windows) ◄──HTTPS──► Připsáno
```

- **Odeslání:** doklady, které ERP zařadí k odeslání, adaptér každých ~15 s předá do Připsáno.
  Obsluha je vidí na obrazovce (rozvoz, výdejna, tablet), zákazník platí převodem, QR platbou nebo hotově.
- **Výsledky:** každých ~5 s si adaptér vyzvedne změny (platby, hotovost, výdej, zrušení, upravená částka)
  a zapíše je k dokladu v ERP, včetně auditu. Pro ERP, které zobrazuje QR kód, ukládá i aktuální
  řetězec QR Platby.
- **Změny dokladů:** zrušení, změnu částky (dobropis) a časové okno odeslání převede do Připsáno sám.
- **Spojení začíná vždy adaptér.** Na serveru nic nenaslouchá a Připsáno se do databáze ERP nepřipojuje.

## Podporované systémy

| Systém | Jak |
|---|---|
| **HELIOS iNuvio** | instalace do HELIOSu balíkem (*Nástroje přizpůsobení › Externí akce › Instalace ext. řešení*), odeslání externí akcí nad fakturami, automatická kontrola změn faktur |
| **Jiné ERP na MS SQL Serveru** (např. HELIOS Nephrite) | vlastní databáze Připsáno na stejném SQL Serveru; ERP zařazuje a ruší doklady dvěma procedurami a čte výsledky z tabulky. Návod pro dodavatele ERP poskytne HD Consulting. |

ERP, které chce Připsáno volat samo, může místo adaptéru použít přímo REST API Připsáno.

## Požadavky

- Windows Server 2016 / Windows 10 (64bit) nebo novější. **.NET se neinstaluje** — adaptér ho má v sobě.
- Přístup k MS SQL Serveru přes TCP; spojení je vždy šifrované (TLS) a certifikát serveru se ověřuje.
- Odchozí HTTPS na adresu Připsáno. Pro aktualizace z konfiguračního programu navíc `api.github.com`
  a `github.com`.
- Účet služby: virtuální účet Windows nebo gMSA (bez hesla), případně SQL uživatel.

## Stažení a ověření

Každé vydání (`adapter-<verze>`) obsahuje:

| Soubor | |
|---|---|
| `PripsanoAdapter-Setup-<verze>.exe` | instalátor služby a konfiguračního programu |
| `PripsanoAdapter-Setup-<verze>.exe.sha256` | kontrolní součet SHA-256 |

Před spuštěním ověřte kontrolní součet:

```powershell
Get-FileHash .\PripsanoAdapter-Setup-<verze>.exe -Algorithm SHA256
```

Výsledek musí odpovídat obsahu souboru `.sha256`.

> ⚠ Instalátor a programy zatím **nejsou digitálně podepsané** — Windows SmartScreen při spuštění
> varuje. Spouštějte jen soubor s ověřeným kontrolním součtem.

## Instalace a aktualizace

1. **Instalace:** spusťte instalátor jako správce. První instalace se zeptá na účet služby a službu
   nespustí — nejdřív nastavení v konfiguračním programu (*Připsáno – nastavení adaptéru* v nabídce Start).
2. **Nastavení:** záložka **ERP** (systém, server, databáze, přihlášení), záložka **Připsáno** (adresa,
   ověření certifikátu, provozovny, klíč API), pak **Otestovat** a **Uložit a spustit**.
3. **SQL v databázi:** tlačítka *Instalační skript…* a *Zkontrolovat SQL…* připraví a ověří objekty
   v databázi (spouští je správce pod svým účtem). U HELIOSu iNuvio nejdřív *Balík pro HELIOS…* a jeho
   import v HELIOSu.
4. **Aktualizace:** nový instalátor přes stávající instalaci, nebo v konfiguračním programu
   *Pokročilé › Program › Aktualizace…* (stáhne poslední vydání z tohoto repozitáře, ověří SHA-256
   a spustí instalátor). Nastavení a klíče zůstávají.

Tichá instalace: `PripsanoAdapter-Setup-<verze>.exe /S [/UCET=DOMENA\ucet$] [/D=cesta]`.

## Zabezpečení

- Klíč API Připsáno a heslo SQL jsou uložené šifrovaně (Windows DPAPI) a nikde se nezobrazují.
  Pro ostrý provoz si klíč vygeneruje správce; HD Consulting dostane jen jeho otisk.
- Účet adaptéru má v databázi jen právo spouštět svých pět procedur — žádný přístup k tabulkám ERP.
  Adaptér jeho práva hlídá a změnu ohlásí; bez nutného práva nic neodešle ani nezapíše.
- HTTPS jen TLS 1.2 / 1.3, ověření certifikátu přes CA nebo připnutý otisk; přesměrování se nesleduje.
- Stav adaptéru (běží, chybí práva, chyba spojení) může hlásit do dohledu (Uptime Kuma).

## Kontakt

**Připsáno:** [pripsano.cz](https://pripsano.cz/)

**HD Consulting s.r.o.** · [hdconsulting.cz](https://hdconsulting.cz) · nikoleta.poislova@hdconsulting.cz

Licenci, zprovoznění u zákazníka, klíče pro ostrý provoz a podporu zajišťuje HD Consulting.
