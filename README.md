# Zendure SolarFlow — Charge réseau en heures creuses (Blueprint Home Assistant)

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/RenjiBuff/HA-Zendure-Production-and-Usage-optimized-BluePrint/main/blueprints/automation/zendure-charge-reseau-hc.yaml)

Blueprint Home Assistant qui recharge une batterie **Zendure SolarFlow** (800 Plus par
défaut, adaptable à toute batterie pilotée par des entités `number`/`select`) depuis le
réseau **uniquement pendant les heures creuses**, **au plus tard possible**, jusqu'à un
niveau cible calculé, puis coupe.

Le blueprint est entièrement paramétrable à l'import : aucune entité n'est écrite en
dur dans le fichier, tout passe par les champs du blueprint.

## Fonctionnalités

- **Cible dynamique** : le bilan net n'utilise pas la production brute mais
  *production prévue − consommation matinale apprise* (fenêtre : heures pleines + solaire
  sous le seuil, avant une heure limite réglable). La référence vaut
  `max(moyenne apprise, dernière matinée mesurée)` : en régime stable la moyenne domine,
  mais le lendemain d'une matinée atypique (ex. premier jour de chauffage), la valeur de
  la veille gonfle la cible.
- **Charge au plus tard** : la fin des heures creuses est lue dans le planning
  (`schedule.get_schedule`), plages contiguës recollées (tolérance 1 s car minuit peut
  valoir `23:59:59.999999`), sans heure écrite en dur. Le lever du soleil retenu est
  celui du jour où se termine la période d'heures creuses (pas de charge inutile le
  week-end quand les HC durent toute la journée). Départ = fin visée − durée nécessaire,
  recalculé en continu.
- **Pilotage matériel** : ordre d'une charge = limite de sortie à 0, puis mode AC sur
  `input`, puis 3 s d'attente, puis limite d'entrée. Retour sur `output` dès qu'on ne
  charge plus (sinon la production solaire s'arrête).
- **Rentabilité économique** : la charge n'est visée que si le coût réel du kWh stocké
  (tarif HC / rendement aller-retour) reste inférieur au tarif HP. Sinon, cible réduite
  au SOC minimal de confort.

## Prérequis

- Les **8 helpers d'état** à créer avant l'import (7 `input_number` +
  1 `input_datetime`) — YAML fourni dans [`docs/blueprint-charge-hc.md`](docs/blueprint-charge-hc.md).
- Un helper `schedule` définissant les heures creuses.
- Les entités de la batterie (SOC, puissance solaire, limite d'entrée/sortie, mode AC)
  et les capteurs réseau / prévisions solaires.
- Les capteurs de prévision solaire (ex. Forecast.Solar) sont attendus en **kWh** : la
  conversion ×1000 est appliquée en interne.

## Installation

1. Créer les 8 helpers d'état (YAML dans
   [`docs/blueprint-charge-hc.md`](docs/blueprint-charge-hc.md)).
2. Cliquer sur le badge « Import blueprint » ci-dessus (ou *Paramètres → Automatisations
   et scènes → Blueprints → Importer un blueprint* et coller l'URL raw du fichier).
3. Choisir les entités demandées et régler les paramètres numériques (capacité,
   tarifs, seuils… — des valeurs par défaut sont proposées).

## Contenu du dépôt

```
blueprints/automation/zendure-charge-reseau-hc.yaml   le blueprint importable
docs/blueprint-charge-hc.md                            prérequis détaillés (helpers, entités, paramètres)
```

## Avertissements

- Désactiver le **HEMS** de l'application Zendure pour les appareils pilotés (indication
  du support Zendure pour forcer une charge réseau).
- Ne pas piloter simultanément depuis l'application Zendure et Home Assistant : des
  réinitialisations de valeurs ont été rapportées.
