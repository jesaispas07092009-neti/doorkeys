<div align="center">

<img src="icon-512.png" width="120" alt="DoorKeys icon" />

# Door*Keys*

**Ton carnet d'accès privé — digicodes, étages, interphones.**

Tout ce qu'il faut pour entrer quelque part, chiffré sur ton appareil, jamais sur un serveur.

[![PWA](https://img.shields.io/badge/PWA-installable-gold?style=flat-square&logo=pwa&logoColor=white&labelColor=080a0e)](https://github.com/17092009-neti/doorkeys)
[![Crypto](https://img.shields.io/badge/AES--GCM-256_bits-gold?style=flat-square&labelColor=080a0e)](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto)
[![PBKDF2](https://img.shields.io/badge/PBKDF2-200k_itérations-gold?style=flat-square&labelColor=080a0e)](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/deriveKey)
[![WCAG](https://img.shields.io/badge/WCAG-AA-gold?style=flat-square&labelColor=080a0e)](https://www.w3.org/WAI/WCAG21/quickref/)
[![Licence](https://img.shields.io/badge/licence-MIT-gold?style=flat-square&labelColor=080a0e)](LICENSE)

</div>

---

## Pourquoi DoorKeys ?

Tu connais ce moment — devant une porte, à chercher le digicode dans tes notes, ton historique WhatsApp, ou ta mémoire. DoorKeys règle ça : toutes tes entrées en un seul endroit, verrouillées par un PIN chiffré, accessibles en deux secondes.

Aucune inscription. Aucun serveur. Aucune synchronisation cloud. Tes données ne quittent jamais ton appareil.

---

## Fonctionnalités

### 🔐 Sécurité locale
- Chiffrement **AES-GCM 256 bits** de toutes les données
- Dérivation de clé **PBKDF2 SHA-256 — 200 000 itérations**
- Comparaison PIN en **temps constant** via double-HMAC (anti timing-attack)
- Déverrouillage **biométrique** via WebAuthn (Face ID, empreinte digitale)
- Lockout progressif : 3 essais → 30s → 2min → 10min, persisté entre onglets
- Backup chiffré exportable avec clé **HKDF** dérivée indépendamment
- **Content-Security-Policy** stricte

### 🗂 Carnet d'accès
- Nom, adresse, digicode, étage, interphone, contact, téléphone, notes, photo, emoji
- Catégories : Famille · Amis · Boulot · Autre
- Favoris avec accès rapide épinglé sur l'écran de verrouillage
- Reveal "toucher pour voir" avec masquage automatique après 5 secondes
- Copie du code en un tap

### 🗺 Cartes & stats
- **Carte interactive** avec géolocalisation de toutes tes adresses (Leaflet + OpenStreetMap)
- Vue **proximité** — les adresses autour de toi, triées par distance
- **Fréquence de visite** — classement des lieux les plus fréquentés
- **Timeline** des visites par mois
- Vues par quartier et par catégorie

### 📤 Partage sécurisé
- Génération de lien de partage **chiffré de bout en bout** (AES-GCM, clé dans le fragment URL — jamais transmise au serveur)
- Choix des champs partagés : adresse, code, étage, notes
- QR Code du lien de partage
- Partage social désactivé automatiquement si le digicode est inclus

### 📱 PWA
- Installable sur iOS, Android, macOS, Windows — **aucune app store**
- Fonctionne **hors ligne** grâce au Service Worker
- Fichier unique `index.html` — zéro dépendance serveur, déployable sur n'importe quel hébergement statique

---

## Démarrage rapide

```bash
# Cloner
git clone https://github.com/17092009-neti/doorkeys.git
cd doorkeys

# Servir localement (n'importe quel serveur statique)
npx serve .
# ou
python3 -m http.server 8080
```

Ouvre `http://localhost:8080` — crée ton espace, choisis un PIN, c'est prêt.

**Déploiement** : copie les 4 fichiers (`index.html`, `manifest.json`, `icon-192.png`, `icon-512.png`) sur GitHub Pages, Netlify, Vercel ou tout hébergement statique.

---

## Architecture

```
doorkeys/
├── index.html      # App complète (HTML + CSS + JS, ~334 Ko)
├── manifest.json   # Manifest PWA
├── icon-192.png    # Icône launcher Android / notifications
└── icon-512.png    # Icône splash screen / maskable
```

DoorKeys est un fichier unique intentionnel — aucune étape de build, aucune dépendance npm, déployable en glisser-déposer. Les bibliothèques tierces (Leaflet, QRCode.js, Tabler Icons) sont chargées depuis des CDN avec versions épinglées.

### Modèle de sécurité

```
PIN utilisateur
    │
    ▼
PBKDF2 (SHA-256, 200k iter, salt aléatoire)
    ├─► pinHash     → vérifié à chaque déverrouillage
    └─► encKey      → déchiffre l'état AES-GCM
              │
              ▼
         state (JSON chiffré) → localStorage
              │
              ├─ addresses
              ├─ shares
              └─ settings

Photos → IndexedDB (hors quota localStorage)
Backup → HKDF(encKey, backupSalt) → AES-GCM
Partage → AES-GCM 256 bits, clé dans le fragment #k= (jamais envoyée au serveur)
```

### Note sur la biométrie

La clé AES qui chiffre le PIN biométrique est exportée en `localStorage` (`doorkeys_bio_key`). C'est un compromis conscient, nécessaire dans une PWA sans backend : la clé ne donne accès qu'au PIN chiffré, lui-même vérifié par PBKDF2. Un avertissement est affiché lors de l'activation. Ne pas utiliser sur un appareil partagé.

---

## Accessibilité

- Sémantique native : `<button>`, `role="radiogroup"`, `aria-live`, focus trap sur toutes les modales et sheets
- Contraste WCAG AA sur tous les textes (ratio minimum ~4.6:1)
- Compatible lecteurs d'écran (VoiceOver iOS, TalkBack Android)
- Respect de `prefers-reduced-motion`
- `user-scalable` non restreint (WCAG 1.4.4)

---

## Dépendances

| Bibliothèque | Version | Usage |
|---|---|---|
| [Tabler Icons](https://tabler.io/icons) | 3.19.0 | Icônes UI |
| [Leaflet](https://leafletjs.com) | 1.9.4 | Carte interactive |
| [QRCode.js](https://github.com/soldair/node-qrcode) | 1.5.4 | Génération QR |
| [Outfit](https://fonts.google.com/specimen/Outfit) | — | Police principale |
| [Instrument Serif](https://fonts.google.com/specimen/Instrument+Serif) | — | Police titres |
| [JetBrains Mono](https://www.jetbrains.com/lp/mono/) | — | Codes d'accès |

Géocodage : [API Adresse (data.gouv.fr)](https://adresse.data.gouv.fr/api-doc/adresse) + [Nominatim](https://nominatim.org) — aucune clé API requise.

---

## Licence

MIT — fais-en ce que tu veux, sans garantie.

---

<div align="center">
<sub>Fait avec soin · Aucune donnée collectée · Aucun tracking · Aucun serveur</sub>
</div>
