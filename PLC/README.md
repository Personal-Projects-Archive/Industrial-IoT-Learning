# PLC — Suite de Smart Factory 4.0 (remplacement de l'ESP32 par un automate)

Volet 100% perso, suite directe du projet Smart Factory 4.0 : même Grafana, même
InfluxDB, la source de données passe de l'ESP32 à un automate Siemens.

## Contexte

- TIA Portal V20
- Siemens S7-1500
- PLCSIM Advanced V7
- Communication S7 puis OPC UA vers Node-RED / InfluxDB / Grafana (déjà en place)

## Architecture

```
FB_Machine
   |
FB_Machine_DB
   |
DB_Comm
   |
Communication S7 / OPC UA
   |
Node-RED
   |
InfluxDB
   |
Grafana
```

## Objectif final

```
Factory IO -> PLC Siemens -> Node-RED -> InfluxDB -> Grafana
```

## Ce qui fonctionne

- Architecture PLC terminée
- FB_Machine développé en SCL
- Instance DB automatique
- DB_Comm créé pour exposer les variables
- Architecture logicielle terminée
- Compilation sans erreur
- Variables testées avec Watch Table
- Simulation logique correcte
- Node-RED / InfluxDB / Grafana déjà fonctionnels (repris du volet ESP32)

## Statut : bloqué

Le téléchargement vers PLCSIM Advanced échoue :

```
Error downloading to module "CPUcommon"
Loading aborted
```

Le CPU simulé démarre mais TIA refuse le téléchargement.

**Déjà testé** : recréer le projet, changer de CPU (S7-1200, S7-1500), PLCSIM
standard et Advanced, vérifier les IP, les interfaces, le Virtual Ethernet Adapter,
compiler plusieurs fois, recréer les DB, tester différentes configs réseau.

**Pistes pas encore essayées, à tester à la reprise** (par ordre de probabilité) :

1. Vérifier que le "virtual switch" PLCSIM n'est activé QUE sur le Siemens PLCSIM
   Virtual Ethernet Adapter, nulle part ailleurs (cause la plus fréquente)
2. Forcer manuellement l'interface "PLCSIM" dans Accessible devices côté TIA
3. Comparer le firmware du CPU (Device Config -> CPU -> Général) à la plage
   officiellement supportée par PLCSIM Advanced V7 (à vérifier dans le Function
   Manual Siemens, SiePortal)
4. Désactiver temporairement l'antivirus/pare-feu pour isoler la cause
5. Réparer puis, en dernier recours, réinstaller PLCSIM Advanced
6. Si possible, tester le programme sur un vrai S7-1500 pour confirmer que le
   problème vient bien de l'environnement de simulation et pas du programme

## Pourquoi ce n'est pas grave

Le programme (FB_Machine, DB_Comm) est terminé et compile sans erreur — ce n'est
pas ce qui est cassé. Seul l'environnement de test (PLCSIM Advanced) bloque.