# PadelBuddy

Android-sovellus padelpelaajien ryhmänmuodostukseen. Käyttäjä asettaa vapaat peliaikansa tai hakee tiettyä aikaa, ja sovellus etsii sopivat pelaajat tason ja sijainnin perusteella. Ryhmä vahvistetaan swipe-eleellä, jonka jälkeen pelaajat voivat sopia kentästä ryhmächatissa.

---

## Ominaisuudet

| Ominaisuus | Kuvaus |
|---|---|
| Google Sign-In | Firebase Authentication |
| Sijainninhaku | GPS tai manuaalinen kaupunkivalinta, säädettävä hakusäde |
| Automaattinen ryhmähaku | Hakee vapaat kenttäajat neljästä hallista ja etsii tasoltaan sopivat pelaajat |
| Vapaa haku | Käyttäjä määrittää oman ajan (esim. vakiovuoro klo 19–20.30) ja haettavien pelaajien määrän (1–3) |
| Tasopohjainen algoritmi | ±50 Padelution-pisteen ikkuna, laajenee kunnes sopiva ryhmä löytyy |
| Swipe-hyväksyntä | Pyyhkäise oikealle = hyväksy, vasemmalle = hylkää |
| Ryhmäkutsu (FCM) | Push-ilmoitukset pelinkutsuista ja vahvistuksista |
| Ryhmächat | Sisäinen viestintä kentänvarausta varten |
| Kenttäintegraatiot | Playtomic (Poropadel GOS & Hiukkavaara), Padelkeskus Oulu, Nallisport Oulu |
| Padelution-integraatio | Pelitason haku padelution.com-profiilista |
| Tietosuoja | Tilin ja kaikkien tietojen poisto suoraan sovelluksesta |

---
## Versiohistoria

| Versio | Muutokset |
|---|---|
| 1.2 | Vapaa haku omalle ajalle (1–3 pelaajaa), hakuehtojen muisto |
| 1.1 | Kenttäintegraatiot, ryhmächat, Padelution-linkitys |
| 1.0 | Ensijulkaisu: automaattinen ryhmähaku, swipe-hyväksyntä |

---

## Tietosuoja

[Tietosuojaseloste](privacy-policy.html) · Ikäraja: 15+
