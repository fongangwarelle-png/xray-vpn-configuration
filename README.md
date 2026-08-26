# 🔒 Xray VLESS VPN Setup — Azure

> Déploiement d'un serveur VPN personnel basé sur le protocole **VLESS** (Xray-core) avec transport **WebSocket + TLS**, géré via **3X-UI**, hébergé sur **Azure**.

---

## 📖 Contexte

Ce projet documente le déploiement complet d'un serveur **VLESS + WebSocket + TLS**, de l'installation du panel jusqu'à une connexion client fonctionnelle — avec les étapes de diagnostic réseau réellement rencontrées en cours de route 

---

## 🧱 Stack utilisée

| Composant | Choix |
|---|---|
| ☁️ Hébergement | Azure — VM `B2ats_v2` (2 vCPU) |
| 🐧 Système | Ubuntu 24.04 LTS |
| 🚀 Serveur proxy | [Xray-core](https://github.com/XTLS/Xray-core) (protocole VLESS) |
| 🎛️ Panel de gestion | [3X-UI](https://github.com/MHSanaei/3x-ui) |
| 🔐 Certificat TLS | Let's Encrypt (Certbot, intégré au panel) |
| 🌐 Nom de domaine | namecheap
| 📱 Client testé | HTTP Injector (Android) |

---

## 📂 Structure du repo

```
config/
├── server-inbound.template.json   # Config inbound VLESS (côté serveur, panel 3X-UI)
└── client.template.json           # Config outbound VLESS (côté client, HTTP Injector)
```

> ⚠️ Les fichiers `.template.json` contiennent des placeholders (`VOTRE_DOMAINE`, `VOTRE_UUID`) à remplacer par vos propres valeurs avant utilisation — voir [Utilisation](#-utilisation).

---

## 🗺️ Architecture réseau

```
Client (HTTP Injector)
   │
   │  VLESS + WebSocket + TLS  (port 8080)
   ▼
Serveur Xray (VM Azure)
   │
   │  freedom (sortie directe)
   ▼
Internet
```

---

## ✅ Prérequis

- Une VM Linux (Ubuntu 24.04 recommandé) avec IP publique
- Un nom de domaine pointant vers cette IP
- Les ports **8080** (VLESS) et **80** (validation Let's Encrypt) ouverts sur le pare-feu / NSG

---

## ⚙️ Installation du panel (résumé)

**1.** Se connecter en SSH à la VM et installe 3X-UI :
```bash
curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh -o install.sh
sudo bash install.sh
```

**2.** Configurer le certificat Let's Encrypt via le menu `x-ui` → *SSL Certificate Management*

**3.** Crée un inbound VLESS sur le port `8080` (WebSocket + TLS) — voir [`config/server-inbound.template.json`](./config/server-inbound.template.json) pour la structure attendue

**4.** Ajoute un client (identifié par un UUID généré automatiquement par le panel)

---

## 🚀 Utilisation

1. Copier [`config/client.template.json`](./config/client.template.json)
2. Remplacer les placeholders par vos propres valeurs (domaine, UUID)
3. Importer le fichier dans un client Xray/V2Ray compatible (testé avec HTTP Injector)

---

## 🐛 Notes de dépannage

Quelques problèmes rencontrés et résolus pendant ce déploiement, susceptibles d'aider d'autres personnes :

| Symptôme | Piste de résolution |
|---|---|
| `context canceled` / `i/o timeout` au handshake WebSocket | Fingerprint uTLS (`chrome`) mal supporté par certains clients — tester sans ce champ |
| `io: read/write on closed pipe` | Rejet d'authentification (UUID incorrect) **ou** proxy intermédiaire tiers devenu inaccessible — tester une connexion directe pour isoler la cause |
| Image Ubuntu dépréciée | Vérifier le statut de dépréciation d'une image avant de la figer dans un script d'automatisation |

---

## ⚠️ Avertissement

Ce projet est fourni à des fins **éducatives et d'usage personnel**. Aucun secret réel (UUID, domaines, certificats) n'est inclus dans ce repo — les fichiers `.template.json` doivent être complétés **localement** avant usage.

---

## 📄 Licence

MIT
