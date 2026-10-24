# Moje Položnice – nadgradnja uporabniškega vmesnika

## Kaj je vključeno
- Puščici za prejšnji/naslednji mesec.
- Klik na naslov meseca odpre izbirnik meseca.
- Gumb **Danes** vrne na tekoči mesec.
- Enoten obrazec za urejanje naziva in zneska.
- Povzetek plačanega in še neplačanega zneska.
- Kratka obvestila o uspešnem shranjevanju ali napaki.
- Večji dotikalni elementi in prilagoditve za manjše zaslone.
- Nova različica predpomnilnika PWA.
- Brisanje je začasno onemogočeno, dokler se ne posodobi Worker, da ne izbriše zgodovine.

## Objavljanje
1. Razširi ZIP.
2. Naloži vse datoteke iz mape `moje-poloznice` v GitHub repozitorij, pri čemer prepiši obstoječe datoteke.
3. Počakaj na GitHub Pages deployment.
4. Na telefonu ali računalniku osveži aplikacijo. Če je nameščena kot PWA in še kaže staro različico, jo zapri in ponovno odpri.

## Pomembne omejitve, ki zahtevajo spremembo Workerja
- Trenutni Workerjev PATCH spremeni naziv predloge za vse mesece, medtem ko znesek spreminja od izbranega meseca naprej. Vmesnik to pojasni.
- Brisanje je v aplikaciji namenoma onemogočeno, ker trenutni Worker briše tudi vse mesečne zapise. Ne vklopi brisanja, dokler Worker ne dobi varnega arhiviranja/prenehanja veljavnosti, ki ohranja plačane mesece.
- Aplikacija še vedno zahteva aktivni Worker in D1 z obstoječima tabelama `poloznice` in `poloznice_meseci`.

## Varnost
Ne deli `APP_PASSWORD`, `AUTH_SECRET` ali prijavnega žetona. Vrednosti skrivnosti morajo ostati v Cloudflare Worker Secrets.
