# 🏭 Smart Factory 4.0 — Learning

Projet IIoT (Industrial IoT) : chaîne complète de la capture de données machine jusqu'à leur affichage dans un dashboard de supervision.

## Principe

```text
ESP32 (simulé sur Wokwi)
   ├── Température
   ├── Humidité
   ├── Etat machine
   ├── Moteur
   ├── Production
   ├── Energie
   └── Mode
        │  Wi-Fi
        ▼
   Broker MQTT public
        │  publish / subscribe
        ▼
     Node-RED
        │
        ▼
    InfluxDB
        │
        ▼
     Grafana
```

- Un **ESP32** est simulé sur [Wokwi](https://wokwi.com/) (pas de matériel physique pour cette V1) et publie l'état complet d'une machine simulée : température, humidité, état machine, moteur, production, énergie et mode. Simulation à voir ici : **[https://wokwi.com/projects/468465716738809857]**
- L'ESP32 se connecte en Wi-Fi et **publie** ces mesures sur un topic MQTT, envoyées à un **broker public** sur Internet — Wokwi en version gratuite ne permet pas de se connecter à un broker local, d'où ce choix pour l'instant.
- **Node-RED** est **abonné** à ce topic : il reçoit les données dès qu'elles sont publiées, puis les répartit en deux groupes avant de les transmettre à InfluxDB :
  - les **fields** : les valeurs mesurables (`temperature`, `humidity`, `production`, `energy`, `motor`)
  - les **tags** : les informations descriptives (`machine`, `status`, `mode`)

  C'est l'organisation classique recommandée par InfluxDB : les fields servent aux calculs et courbes, les tags servent à filtrer et regrouper.
- Node-RED transmet ces données à **InfluxDB** (measurement `telemetry`, bucket `telemetry`), une base de données conçue pour stocker des séries temporelles (mesures horodatées).
- **Grafana** lit InfluxDB et affiche les données sous forme de dashboard.

Tout tourne dans des conteneurs **Docker** (`compose.yaml` à la racine).

## Dashboard Grafana

7 widgets pour cette V1 :

| Widget | Type | Donnée |
|---|---|---|
| Machine / Etat / Mode | Stat | `machine`, `status`, `mode` |
| Température actuelle | Stat | `temperature` |
| Humidité actuelle | Stat | `humidity` |
| Production totale | Stat | `production` |
| Consommation énergie | Stat | `energy` |
| Courbe température | Time series | `temperature` (historique) |
| Courbe humidité | Time series | `humidity` (historique) |

## Aperçu

**1. Simulation Wokwi (ESP32)**
![Wokwi ESP32](Images/wokwi-esp32.png)
Le circuit simulé et le code embarqué qui publie l'état de la machine en MQTT.

**2. Node-RED**
![Node-RED](Images/node-red-flow.png)
Le flow qui s'abonne au topic MQTT, sépare fields et tags, et transmet vers InfluxDB.

**3. InfluxDB**
![InfluxDB](Images/influxdb-data.png)
Les mesures stockées dans le bucket `telemetry`, horodatées.

**4. Grafana**
![Grafana](Images/grafana-dashboard.png)
Le dashboard final avec les 7 widgets décrits ci-dessus.

## Installation

1. Cloner le dépôt
2. Lancer la stack : `docker compose up -d`
3. Configurer InfluxDB (première utilisation uniquement) : ouvrir [http://localhost:8086](http://localhost:8086), créer le compte admin, une organisation et un bucket nommé `telemetry`, puis générer un token API — c'est ce token qui permettra à Node-RED et à Grafana de s'y connecter
4. Ouvrir la simulation ESP32 sur [Wokwi](https://wokwi.com/projects/468465716738809857) et lancer la simulation
5. Importer `NodeRED/flows.json` dans Node-RED, puis renseigner l'URL d'InfluxDB, l'organisation, le bucket `telemetry` et le token dans le nœud de sortie InfluxDB
6. Dans Grafana, ajouter InfluxDB comme source de données (URL, organisation, bucket `telemetry`, token), puis importer `Grafana/SmartFactoryDashboard.json`

## Statut — V1

- [x] ESP32 simulé (Wokwi) : température, humidité, état machine, moteur, production, énergie, mode
- [x] MQTT (broker public)
- [x] Node-RED (séparation fields / tags)
- [x] InfluxDB (measurement `telemetry`)
- [x] Grafana (dashboard 7 widgets)
- [x] Docker

## Prochaines étapes

Passage à une architecture plus proche de l'industriel : broker MQTT local (Mosquitto), ajout d'un PLC Siemens S7-1200 en OPC UA, dashboard de supervision plus complet.

## Auteur

**Yassine El Yazami** — Étudiant en Automatisation (EPHEC TECH)