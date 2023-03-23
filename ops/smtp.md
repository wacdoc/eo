# Konstruu vian propran SMTP-poŝt-sendservilon

## preambulo

SMTP povas rekte aĉeti servojn de nubaj vendistoj, kiel ekzemple:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali nubo retpoŝta puŝo](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Vi ankaŭ povas konstrui vian propran poŝtservilon - senlima sendo, malalta ĝenerala kosto.

Malsupre, ni pruvas paŝon post paŝo kiel konstrui nian propran poŝtservilon.

## Elekto de servilo

La memgastigita SMTP-servilo postulas publikan IP kun havenoj 25, 456, kaj 587 malfermitaj.

Ofte uzataj publikaj nuboj blokis ĉi tiujn havenojn defaŭlte, kaj eble eblas malfermi ilin per ordono de laboro, sed ĝi estas ja tre ĝena.

Mi rekomendas aĉeti de gastiganto, kiu havas ĉi tiujn havenojn malfermitaj kaj subtenas agordi inversajn domajnajn nomojn.

Ĉi tie, mi rekomendas [Contabo](https://contabo.com) .

Contabo estas gastiganta provizanto bazita en Munkeno, Germanio, fondita en 2003 kun tre konkurencivaj prezoj.

Se vi elektas Eŭron kiel la aĉetan moneron, la prezo estos pli malmultekosta (servilo kun 8GB-memoro kaj 4 CPUoj kostas ĉirkaŭ 529 juanojn jare, kaj la komenca instalado-kotizo estas senpaga dum unu jaro).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Kiam vi faras mendon, rimarku `prefer AMD` , kaj la servilo kun AMD CPU havos pli bonan rendimenton.

En la sekvanta, mi prenos la VPS de Contabo kiel ekzemplon por montri kiel konstrui vian propran poŝtservilon.

## Ubuntu-sistema agordo

La operaciumo ĉi tie estas Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Se la servilo sur ssh montras `Welcome to TinyCore 13!` (kiel montrite en la suba figuro), tio signifas, ke la sistemo ankoraŭ ne estis instalita. Bonvolu malkonekti ssh kaj atendi kelkajn minutojn por ensaluti denove.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Kiam `Welcome to Ubuntu 22.04.1 LTS` aperas, la inicialigo estas kompleta, kaj vi povas daŭrigi kun la sekvaj paŝoj.

### [Laŭvola] Komencu la evoluan medion

Ĉi tiu paŝo estas laŭvola.

Por komforto, mi metis la instaladon kaj sisteman agordon de ubuntu-programaro en [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Rulu la sekvan komandon por instali per unu klako.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Ĉinaj uzantoj, bonvolu uzi la sekvan komandon anstataŭe, kaj la lingvo, horzono ktp. estos aŭtomate agordita.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo ebligas IPV6

Ebligu IPV6 por ke SMTP ankaŭ povas sendi retpoŝtojn kun IPV6-adresoj.

redakti `/etc/sysctl.conf`

Modifi aŭ aldonu la sekvajn liniojn

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Sekvu [la lernilon de contabo: Aldonante IPv6-konektecon al via servilo](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Redaktu `/etc/netplan/01-netcfg.yaml` , aldonu kelkajn liniojn kiel montrite en la suba figuro (Kontabo VPS-defaŭlta agorda dosiero jam havas ĉi tiujn liniojn, simple malkomentu ilin).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Tiam `netplan apply` por efikigi la modifitan agordon.

Post kiam la agordo estas sukcesa, vi povas uzi `curl 6.ipw.cn` por vidi la ipv6-adreson de via ekstera reto.

## Klonu la agordajn deponejajn operaciojn

```
git clone https://github.com/wactax/ops.soft.git
```

## Generu senpagan SSL-atestilon por via domajna nomo

Sendi poŝton postulas SSL-atestilon por ĉifrado kaj subskribo.

Ni uzas [acme.sh](https://github.com/acmesh-official/acme.sh) por generi atestojn.

acme.sh estas malfermfonta aŭtomatigita atestila ilo,

Enigu la agordan magazenon ops.soft, rulu `./ssl.sh` , kaj `conf` dosierujo estos kreita en **la supra dosierujo** .

Trovu vian DNS-provizanton de [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , redaktu `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Poste rulu `./ssl.sh 123.com` por generi `123.com` kaj `*.123.com` atestiloj por via domajna nomo.

La unua ekzekuto aŭtomate instalos [acme.sh](https://github.com/acmesh-official/acme.sh) kaj aldonos planitan taskon por aŭtomata renovigo. Vi povas vidi `crontab -l` , ekzistas tia linio jene.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

La vojo por la generita atestilo estas io kiel `/mnt/www/.acme.sh/123.com_ecc。`

Renovigo de atestilo vokos `conf/reload/123.com.sh` skripton, redaktu ĉi tiun skripton, vi povas aldoni komandojn kiel `nginx -s reload` por refreŝigi la atestilon de rilataj aplikoj.

## Konstruu SMTP-servilon kun chasquid

[chasquid](https://github.com/albertito/chasquid) estas malfermfonta SMTP-servilo skribita en la lingvo Go.

Kiel anstataŭaĵo de la antikvaj poŝtservilaj programoj kiel Postfix kaj Sendmail, chasquid estas pli simpla kaj pli facile uzebla, kaj ĝi ankaŭ estas pli facila por sekundara disvolviĝo.

Rulu `./chasquid/init.sh 123.com` estos instalita aŭtomate per unu klako (anstataŭigi 123.com per via sendadomajna nomo).

## Agordu Retpoŝtan Subskribon DKIM

DKIM estas uzata por sendi retpoŝtajn subskribojn por malhelpi leterojn esti traktataj kiel spamo.

Post kiam la komando funkcias sukcese, oni petos vin agordi la DKIM-rekordon (kiel montrite sube).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Nur aldonu TXT-rekordon al via DNS (kiel montrite sube).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Rigardu servostaton kaj protokolojn

 `systemctl status chasquid` Vidi servostaton.

La stato de normala operacio estas kiel montrita en la figuro malsupre

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` aŭ `journalctl -xeu chasquid` povas vidi la erarprotokolon.

## Inversa domajna nomo agordo

La inversa domajna nomo estas permesi la IP-adreson esti solvita al la responda domajna nomo.

Agordi inversan domajnan nomon povas malhelpi retpoŝtojn esti identigitaj kiel spamo.

Kiam la poŝto estas ricevita, la ricevanta servilo faros inversan domajnan nomon analizon sur la IP-adreso de la senda servilo por konfirmi ĉu la sendanta servilo havas validan inversan domajnan nomon.

Se la senda servilo ne havas inversan domajnan nomon aŭ se la inversa domajna nomo ne kongruas kun la IP-adreso de la senda servilo, la ricevanta servilo povas rekoni la retpoŝton kiel spamon aŭ malakcepti ĝin.

Vizitu [https://my.contabo.com/rdns](https://my.contabo.com/rdns) kaj agordu kiel montrite sube

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Post agordo de la inversa domajna nomo, memoru agordi la antaŭan rezolucion de la domajna nomo ipv4 kaj ipv6 al la servilo.

## Redaktu la gastigan nomon de chasquid.conf

Modifi `conf/chasquid/chasquid.conf` al la valoro de la inversa domajna nomo.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Poste rulu `systemctl restart chasquid` por rekomenci la servon.

## Rezerva konf al git-deponejo

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Ekzemple, mi kopias la conf-dosierujon al mia propra github-procezo jene

Kreu privatan magazenon unue

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Enigu la conf-dosierujon kaj sendu al la magazeno

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Aldoni sendinto

kuri

```
chasquid-util user-add i@wac.tax
```

Povas aldoni sendinton

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Kontrolu, ke la pasvorto estas ĝuste agordita

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Post aldoni la uzanton, `chasquid/domains/wac.tax/users` estos ĝisdatigita, memoru sendi ĝin al la magazeno.

## DNS aldonas SPF-rekordon

SPF (Sender Policy Framework) estas retpoŝta konfirmteknologio uzata por malhelpi retpoŝtan fraŭdon.

Ĝi kontrolas la identecon de poŝtsendanto kontrolante ke la IP-adreso de la sendinto kongruas kun la DNS-rekordoj de la domajna nomo, kiun ĝi asertas esti, malhelpante fraŭdulojn sendi falsajn retpoŝtojn.

Aldonante SPF-rekordojn povas malhelpi retpoŝtojn esti identigitaj kiel spamo kiel eble plej multe.

Se via domajna nomservilo ne subtenas SPF-tipo, simple aldonu TXT-tipan rekordon.

Ekzemple, la SPF de `wac.tax` estas kiel sekvas

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF por `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Notu, ke mi `include:_spf.google.com` , ĉar mi poste agordos `i@wac.tax` kiel sendadreson en la Gugla leterkesto.

## DNS-agordo DMARC

DMARC estas la mallongigo de (Domajna-bazita Mesaĝa Aŭtentigo, Raportado kaj Konformo).

Ĝi estas uzata por kapti SPF-resaltojn (eble kaŭzitaj de agordaj eraroj, aŭ iu alia ŝajnigas esti vi por sendi spamon).

Aldonu TXT-rekordon `_dmarc` ,

La enhavo estas kiel sekvas

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

La signifo de ĉiu parametro estas kiel sekvas

### p (Politiko)

Indikas kiel trakti retpoŝtojn kiuj malsukcesas SPF (Sender Policy Framework) aŭ DKIM (DomainKeys Identified Mail) konfirmo. La p-parametro povas esti agordita al unu el tri valoroj:

* neniu: Neniu ago estas prenita, nur la konfirmrezulto estas redonita al la sendinto per la retpoŝta raporta mekanismo.
* Kvaranteno: Metu la poŝton kiu ne pasis la konfirmon en la spam-dosierujon, sed ne malakceptos la poŝton rekte.
* malakcepti: Rekte malakcepti retpoŝtojn, kiuj malsukcesas kontroladon.

### fo (Malsukcesaj Opcioj)

Specifas la kvanton de informoj resendita de la raporta mekanismo. Ĝi povas esti agordita al unu el la sekvaj valoroj:

* 0: Raporti validigajn rezultojn por ĉiuj mesaĝoj
* 1: Raportu nur mesaĝojn kiuj malsukcesas konfirmon
* d: Raportu nur malsukcesojn pri kontrolado de domajna nomo
* s: nur raportu malsukcesojn pri kontrolo de SPF
* l: Raportu nur malsukcesojn pri DKIM-kontrolado

### rua & ruf

* rua (Reporting URI for Agregate reports): Retadreso por ricevi kunigitaj raportoj
* ruf (Raporta URI por Krimmedicinaj raportoj): retadreso por ricevi detalajn raportojn

## Aldonu MX-rekordojn por plusendi retpoŝtojn al Google Mail

Ĉar mi ne povis trovi senpagan kompanian leterkeston kiu subtenas universalajn adresojn (Catch-All, povas ricevi ajnajn retpoŝtojn senditajn al ĉi tiu domajna nomo, sen limigoj pri prefiksoj), mi uzis chasquid por plusendi ĉiujn retpoŝtojn al mia Gmail-leterkesto.

**Se vi havas vian propran pagitan komercan leterkeston, bonvolu ne modifi la MX kaj preterlasi ĉi tiun paŝon.**

Redaktu `conf/chasquid/domains/wac.tax/aliases` , agordu plusendan leterkeston

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` indikas ĉiujn retpoŝtojn, `i` estas la retadresa prefikso de la sendanto kreita supre. Por plusendi poŝton, ĉiu uzanto devas aldoni linion.

Poste aldonu la MX-rekordon (mi montras rekte al la adreso de la inversa domajna nomo ĉi tie, kiel montrite en la unua linio en la suba figuro).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Post kiam la agordo estas kompleta, vi povas uzi aliajn retadresojn por sendi retmesaĝojn al `i@wac.tax` kaj `any123@wac.tax` por vidi ĉu vi povas ricevi retpoŝtojn en Gmail.

Se ne, kontrolu la chasquid protokolo ( `grep chasquid /var/log/syslog` ).

## Sendu retmesaĝon al i@wac.tax per Google Mail

Post kiam Google Mail ricevis la poŝton, mi nature esperis respondi per `i@wac.tax` anstataŭ i.wac.tax@gmail.com.

Vizitu [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) kaj alklaku "Aldoni alian retadreson".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Poste, enigu la konfirmkodon ricevitan per la retpoŝto al kiu estis plusendita.

Fine, ĝi povas esti agordita kiel la defaŭlta senditadreso (kune kun la opcio respondi per la sama adreso).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Tiamaniere ni kompletigis la starigon de la SMTP-poŝtservilo kaj samtempe uzas Google Mail por sendi kaj ricevi retpoŝtojn.

## Sendu provan retpoŝton por kontroli ĉu la agordo sukcesas

Enigu `ops/chasquid`

Rulu `direnv allow` instali dependecojn (direnv estis instalita en la antaŭa unu-klava inicialigo kaj hoko estis aldonita al la ŝelo)

poste kuru

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

La signifo de la parametroj estas kiel sekvas

* uzanto: SMTP uzantnomo
* pasi: SMTP-pasvorto
* al: ricevanto

Vi povas sendi provan retpoŝton.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Oni rekomendas uzi Gmail por ricevi testajn retmesaĝojn por kontroli ĉu la agordoj sukcesas.

### TLS-norma ĉifrado

Kiel montrite en la suba figuro, ekzistas ĉi tiu malgranda seruro, kio signifas, ke la SSL-atestilo estis sukcese ebligita.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Poste alklaku "Montri Originalan Retpoŝton"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Kiel montrite en la suba figuro, la originala retpoŝta paĝo de Gmail montras DKIM, kio signifas, ke la DKIM-agordo sukcesas.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Kontrolu la Ricevitan en la kaplinio de la originala retpoŝto, kaj vi povas vidi, ke la sendinta adreso estas IPV6, kio signifas, ke IPV6 ankaŭ estas agordita sukcese.
