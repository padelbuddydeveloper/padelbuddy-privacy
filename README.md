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

## Tekninen pino

- **Kotlin** + **Jetpack Compose** + **Material Design 3**
- **Firebase:** Auth, Firestore, Cloud Messaging (FCM)
- **Arkkitehtuuri:** MVVM · ViewModel · StateFlow · Navigation Compose
- **Sijainti:** FusedLocationProviderClient + GeoFire geohash-kyselyt
- **Verkkopyynnöt:** OkHttp + Gson (Playtomic), JSoup (Padelution-scraping)
- **Push-ilmoitukset:** Firebase Cloud Messaging + Cloud Functions

---

## Vaatimukset

- Android Studio Hedgehog (2023.1.1) tai uudempi
- JDK 17
- Google-tili Firebase-projektia varten
- `google-services.json` omasta Firebase-projektista (ks. alla)

---

## Käyttöönotto

### 1. Firebase-projekti

1. Avaa [Firebase Console](https://console.firebase.google.com) → **Add project**
2. Ota käyttöön: **Authentication** (Google Sign-In), **Firestore**, **Cloud Messaging**

### 2. Android-sovelluksen rekisteröinti

1. Firebase Console → Android-kuvake
2. Package name: `com.PadelBuddy`
3. Lataa **`google-services.json`**
4. Korvaa `app/google-services.json` ladatulla tiedostolla

### 3. Firestore-säännöt

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    match /users/{uid} {
      allow read:  if request.auth != null;
      allow write: if request.auth.uid == uid;
    }

    match /locations/{uid} {
      allow read:  if request.auth != null;
      allow write: if request.auth.uid == uid;
    }

    match /matches/{matchId} {
      allow read:  if request.auth != null &&
                      request.auth.uid in resource.data.participantUids;
      allow create: if request.auth != null;
      allow update: if request.auth != null &&
                       request.auth.uid in resource.data.participantUids;

      match /messages/{msgId} {
        allow read, write: if request.auth != null;
      }
      match /members/{uid} {
        allow read, write: if request.auth.uid == uid;
      }
    }
  }
}
```

### 4. Cloud Functions

```bash
cd functions
npm install
firebase deploy --only functions
```

### 5. Käynnistys

```bash
./gradlew assembleDebug
```

---

## Projektirakenne

```
app/src/main/java/com/padelbuddy/
├── data/
│   ├── model/
│   │   ├── User.kt                    # Käyttäjäprofiili
│   │   ├── NearbyUser.kt              # Lähipelaaja + effectivePoints-logiikka
│   │   ├── AvailabilitySlot.kt        # Peliaika (päivä + kellonaika)
│   │   ├── Match.kt                   # Matsi, pelaajat, status
│   │   ├── MatchResult.kt             # Sealed: Loading | GroupFound | NoPlayersFound | Error
│   │   ├── PlayerWithAvailability.kt  # Kandidaatti + yhteiset ajat
│   │   ├── SearchPreferences.kt       # Sijainti + hakusäde
│   │   ├── VenueSlot.kt               # Kenttävuoro (Playtomic / Cintoia)
│   │   └── Message.kt                 # Ryhmächat-viesti
│   └── repository/
│       ├── MatchEngine.kt             # Ryhmänmuodostusalgoritmi (tasoikkuna-haku)
│       ├── AvailabilityMatcher.kt     # Päällekkäisyyslogiikka aikaväleille
│       ├── MatchRepository.kt         # Matsin CRUD + reaaliaikakuuntelu
│       ├── AvailabilityRepository.kt  # Peliaikojen tallennus
│       ├── LocationRepository.kt      # Geohash-kyselyt + sijaintipäivitys
│       ├── UserRepository.kt          # Käyttäjäprofiili + Padelution-tiedot
│       ├── PlayerRepository.kt        # Lähipelaajat Flow:na
│       ├── PlaytomicRepository.kt     # Kenttävuorot Playtomicista
│       ├── CintoiaRepository.kt       # Kenttävuorot Cintoiasta
│       ├── PadelutionRepository.kt    # Pelitason haku padelution.com:sta
│       ├── RejectedPlayersCache.kt    # Hylättyjen pelaajien muisti
│       └── SearchPrefsRepository.kt   # Hakuasetusten tallennus
└── ui/
    ├── navigation/
    │   ├── Screen.kt                  # Reittimäärittelyt
    │   ├── AppNavigation.kt           # Kirjautumisflow + NavHost
    │   └── BottomNavScreen.kt         # Alakirjaston välilehdet
    ├── screen/
    │   ├── MainScreen.kt              # Isäntä: BottomNav + kaikki reitit
    │   ├── NearbyScreen.kt            # Etsi-välilehti: oma aika + lähipelaajat
    │   ├── FindScreen.kt              # Automaattinen ryhmäswipe
    │   ├── FreeSearchScreen.kt        # Vapaa haku omalle ajalle
    │   ├── PendingMatchScreen.kt      # Kutsujen odottaminen + korvauslogiikka
    │   ├── IncomingMatchScreen.kt     # Pelikutsun vastaanotto
    │   ├── GroupChatScreen.kt         # Ryhmächat
    │   ├── ProfileScreen.kt           # Käyttäjäprofiili + Padelution-linkitys
    │   └── ...
    └── viewmodel/
        ├── GroupSwipeViewModel.kt     # Automaattinen ryhmähaku
        ├── FreeSearchViewModel.kt     # Vapaa haku + lomakkeen tilan säilytys
        ├── PendingMatchViewModel.kt   # Matchin tila + pelaajan korvaus
        └── ...
```

---

## Firestore-rakenne

### `users/{uid}`
| Kenttä | Tyyppi | Kuvaus |
|---|---|---|
| `nickname` | string | Nimimerkki |
| `padelutionRating` | number | Padelution-pisteet |
| `padelLevel` | string | Taso: Aloittelija → Ammattilainen |
| `selfRating` | number | Oma arvio 1–5 |
| `availability` | array | Peliaikavaraukset |
| `searchPrefs` | map | Sijainti + hakusäde |
| `fcmToken` | string | Push-ilmoitusavain |
| `rejectedPlayers` | array | Hylättyjen pelaajien UID:t |

### `locations/{uid}`
| Kenttä | Tyyppi | Kuvaus |
|---|---|---|
| `lat` / `lng` | number | GPS-koordinaatit |
| `geohash` | string | GeoFire-geohash geospatiaalikyselyihin |
| `visible` | boolean | Näkyykö muille hakijoille |

### `matches/{matchId}`
| Kenttä | Tyyppi | Kuvaus |
|---|---|---|
| `players` | array | Pelaajat + hyväksymistilat |
| `slot` | map | Peliaika (päivä, alku, loppu) |
| `status` | string | PENDING → CONFIRMED / EXPIRED / CANCELLED |
| `expiresAt` | number | Vanhentumisaika (5 min kutsusta) |

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
