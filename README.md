# Ulož.to downloader

Paralelní stahovač z [Ulož.to](http://ulozto.cz) inspirovaný
[Vžum](http://vzum.8u.cz/) (credits to Popcorn) s automatickým louskáním CAPTCHA
kódů pomocí TensorFlow modelu z projektu
[ulozto-captcha-breaker](https://github.com/JanPalasek/ulozto-captcha-breaker)
(credits to Jan Palasek).

Narozdíl od originálního Vžum je tato verze napsaná v Pythonu, dá se provozovat
jednoduše i na Linuxu a zdrojový kód je veřejně dostupný, což umožňuje další
forky a rozšíření do budoucna. Například by mohla vzniknout "serverová" verze
s webovým rozhraním.

**Nápady na další vylepšení (případně rovnou pull requesty) vítány :-)**

## Klíčové vlastnosti

* Sám pozná downloady, kde Ulož.to umožňuje stahovat bez CAPTCHA kódů
* Dokáže přečíst sám CAPTCHA kódy díky projektu
  [ulozto-captcha-breaker](https://github.com/JanPalasek/ulozto-captcha-breaker) (thx Jan Palasek)
  * Louská kódy pomocí natrénovaného TensorFlow modelu
* Download linky získává přes Tor, aby se vyhnul nové limitaci ze strany Uloz.to
* Umí opakovaně využít stejný stahovací link pro více částí
  * Ulož.to nyní (podzim 2020) umožňuje získat jen dva stahovací linky za
    minutu, ale stejný link je možné používat po dostahování původní části
    opakovaně pro stahování dalších částí
* Umí navazovat přerušená stahování (pokud se zachová stejný počet částí)
* Nyní stahuje přímo do jednoho souboru, místo dělení na části a potom spojování
* Konzolový status panel se statistikou úspěšnosti při získávání linků
* Celkový průběh staženo / okamžitá rychlost stahování ve druhém řádku status panelu (save progress monitor)
* Cache soubor download linků pro pokračování nebo opětovné stažení, po restartu se bez nového
  získávání download linků rovnou stahuje a nové download linky se získávají jen když jich není
  v cache souboru dostatek. Vytváří malý textový soubor `.ucache` jenž je možné použít znovu
  a stahovat maximální rychlostí ihned bez získávání linků. Tento soubor má malou velikost
  a lze ho např. sdílet. U velkých souborů (100ky MB) je platnost linku 48 hodin.

## Instalace

Nejjednodušší je využít verzi uveřejněnou na [PyPi](https://pypi.org/project/ulozto-downloader/):

```shell
$ pip3 install --upgrade ulozto-downloader
```

Toto instaluje všechny dependence **vyjma TensorFlow Lite** pro automatické
louskání CAPTCHA kódů (protože repozitář PyPI zakazuje přímé URL dependence).

### Instalace TORu

Program vyžaduje spustitelný tor, protože používá stem a očekává ho v `$PATH`.

### Instalace TensorFlow Lite (automatické louskání CAPTCHA)

Na stránce [TensorFlow Lite](https://www.tensorflow.org/lite/guide/python) si
v tabulce vyberte správnou verzi podle vašeho systému a verze Pythonu (zjistíte
zavoláním `python3 -V`), zkopírujte URL a instalujte pomocí:

```shell
$ pip3 install <URL>
# Například tedy pro Python 3.8 na x86-64 Linuxu:
$ pip3 install https://github.com/google-coral/pycoral/releases/download/release-frogfish/tflite_runtime-2.5.0-cp38-cp38-linux_x86_64.whl
```
Na linuxu je mozné pro novější **python 3.9.x** zkompilovat **tflite_runtime-2.6.0-cp39-cp39-linux_x86_64.whl** a pak nainstalovat pomocí:
```shell
pip install tflite_runtime-2.6.0-cp39-cp39-linux_x86_64.whl
```
Tento soubor pro python 3.9, linux x86-64 a GLIBC_2.33 je nyní také součástí
repozitáře. Pokud potřebujete starší verzi GLIBC (například pro Debian), tak
můžete zkusit následovat postup v <https://github.com/google-coral/pycoral/issues/6>

### Instalace Tkinter (ruční opisování CAPTCHA)

Potřebujete na systému instalovaný Tkinter (bohužel není na PyPI, takže je
potřeba instalovat ručně).

Často už je instalovaný, ale pokud by náhodou nebyl, tak bývá v balíčku
`python3-tk` (případně následujte instrukce na
[webu Tk](https://tkdocs.com/tutorial/install.html)).

## Android - Termux

1.) Stáhněte si aplikaci [Termux](https://termux.com/) (Ne z google play!)
2.) Povolte aplikaci přístup k uložišti pomocí:
```shell
$ termux-setup-storage
```
3.) Aktualizujte balíčky:
```shell
pkg upgrade
```
4.) Nainstalujte Python:
```shell
$ pkg install python3
```
5.) Nainstalujte Numpy:
```shell
$ MATHLIB="m" pip install numpy
```
6.) Nainstalujte libjpeg-turbo:
```shell
pkg install libjpeg-turbo
```
7.) Nainstalujte Tor:
```shell
pkg install tor
```
8.) Nainstalujte Tkinter:
```shell
pkg install python-tkinter
```
9.) Nainstalujte Git:
```shell
pkg install git
```
10.) Nainstalujte opravenou verzi Ulozto-downloaderu:
```shell
pip install git+https://github.com/zbyna/ulozto-downloader.git@multiprocessingdummy#egg=ulozto-downloader
```
11.) Nainstalujte grafické rozhraní, ve kterém budete opisovat captcha kódy, podle návodu [zde](https://wiki.termux.com/wiki/Graphical_Environment).


## Použití

Pro volbu automatického čtení CAPTCHA kódů slouží přepínač `--auto-captcha`,
pro volbu počtu částí slouží přepínač `--parts N`.

```shell
$ ulozto-downloader --auto-captcha --parts 15 "https://ulozto.cz/file/TKvQVDFBEhtL/debian-9-6-0-amd64-netinst-iso"
```

![Ukázka stahování](https://raw.githubusercontent.com/setnicka/ulozto-downloader/master/example-screenshot.png)

Při využití automatického louskání doporučuji využít velký počet částí, klidně
50 (spustíte `ulozto-downloader` a necháte ho pracovat, on si jednou za minutu
louskne další dva stahovací linky a postupně navyšuje počet najednou stahovaných
částí).
