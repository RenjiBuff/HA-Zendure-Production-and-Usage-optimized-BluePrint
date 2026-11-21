# Blueprint « Charge réseau en heures creuses » — prérequis et paramètres

Le blueprint [`blueprints/automation/zendure-charge-reseau-hc.yaml`](../blueprints/automation/zendure-charge-reseau-hc.yaml)
écrit dans **8 helpers d'état** : ils doivent exister **avant** l'import, sinon
l'automatisation générée échoue à l'exécution.

## 1. Helpers d'état à créer

Exemple de configuration (noms proposés préfixés `hc_` pour éviter les collisions ;
à coller sous `input_number:` / `input_datetime:` dans `configuration.yaml`, ou à créer
via l'interface *Paramètres → Appareils et services → Helpers*) :

```yaml
input_number:
  hc_conso_maison_moyenne_wh:
    name: HC - consommation maison moyenne apprise
    min: 0
    max: 30000
    step: 50
    initial: 3000
    unit_of_measurement: Wh
  hc_conso_maison_accumulee_wh:
    name: HC - consommation maison accumulée (jour en cours)
    min: 0
    max: 30000
    step: 1
    initial: 0
    unit_of_measurement: Wh
  hc_conso_matinale_moyenne_wh:
    name: HC - consommation matinale moyenne apprise
    min: 0
    max: 30000
    step: 50
    initial: 2000
    unit_of_measurement: Wh
  hc_conso_matinale_derniere_wh:
    name: HC - consommation matinale dernière mesurée
    min: 0
    max: 30000
    step: 50
    initial: 0
    unit_of_measurement: Wh
  hc_conso_matinale_accumulee_wh:
    name: HC - consommation matinale accumulée (jour en cours)
    min: 0
    max: 30000
    step: 1
    initial: 0
    unit_of_measurement: Wh
  hc_dernier_rendement_pct:
    name: HC - dernier rendement mesuré (diagnostic)
    min: 0
    max: 100
    step: 0.1
    initial: 85
    unit_of_measurement: "%"
  hc_duree_charge_calculee_min:
    name: HC - durée de charge calculée (diagnostic)
    min: 0
    max: 600
    step: 1
    initial: 0
    unit_of_measurement: min

input_datetime:
  hc_depart_charge_calcule:
    name: HC - départ de charge calculé (diagnostic)
    has_date: true
    has_time: true
```

| Helper (champ du blueprint) | Rôle |
|---|---|
| `helper_conso_maison_moyenne` | Moyenne apprise de la consommation journalière. Démarre à 3000 Wh et converge lentement (poids `alpha_apprentissage_conso`). |
| `helper_conso_maison_accumulee` | Accumulation de la consommation du jour en cours (remise à zéro au bilan). |
| `helper_conso_matinale_moyenne` | Moyenne apprise de la consommation de la fenêtre matinale (heures pleines + solaire sous le seuil, avant `fin_fenetre_matinale_h`). |
| `helper_conso_matinale_derniere` | Dernière matinée mesurée ; la cible vaut `max(moyenne, dernière)`. |
| `helper_conso_matinale_accumulee` | Accumulation de la fenêtre matinale du jour en cours. |
| `helper_dernier_rendement` | Diagnostic : dernier rendement aller-retour mesuré (%). |
| `helper_duree_charge_calculee` | Diagnostic : durée de charge calculée (minutes). |
| `helper_depart_charge_calcule` | Diagnostic : date/heure de départ de charge recommandée. |

## 2. Entités à fournir à l'import

| Champ | Domaine | Rôle |
|---|---|---|
| `planning_hc` | `schedule` | Planning des heures creuses. Minuit peut valoir `23:59:59.999999` (tolérance de fusion 1 s). |
| `capteur_soc` | `sensor` | Niveau de batterie (%). |
| `capteur_solaire` | `sensor` | Puissance solaire entrante (W). |
| `capteur_prevision_restante` | `sensor` | Prévision solaire restante aujourd'hui (**kWh**, ×1000 en interne). |
| `capteur_prevision_demain` | `sensor` | Prévision solaire demain (**kWh**). |
| `capteur_conso_reseau` | `sensor` | Consommation résiduelle au compteur réseau (W), peut être négative (export). |
| `capteur_beem` | `sensor` | Autre production non pilotable injectée avant le compteur (W, ex. Beem). |
| `capteur_sortie_maison` | `sensor` | Sortie batterie vers la maison (W, solaire et batterie confondus). |
| `capteur_rendement` | `sensor` | Rendement aller-retour (%, accepte 0-1 ou 0-100 ; repli 85 %). |
| `limite_entree` | `number` | Limite de charge réseau (W). |
| `limite_sortie` | `number` | Limite de décharge (W). |
| `select_mode_ac` | `select` | Mode AC : options détectées par leur contenu en minuscules (`input`/`output`). |

## 3. Paramètres numériques du blueprint

| Champ | Défaut | Rôle |
|---|---|---|
| `capacite_wh` | 1920 | Capacité totale de la batterie (Wh). |
| `seuil_debut_solaire_w` | 30 | Seuil de détection du démarrage solaire (W). |
| `puissance_charge_transition_w` | 0 | Puissance de charge après démarrage du solaire (W) ; 0 = priorité totale au solaire. |
| `puissance_charge_hc_w` | 600 | Puissance de charge réseau en heures creuses (W). |
| `soc_max_charge_pct` | 90 | SOC maximal autorisé via charge réseau (%). |
| `soc_min_confort_pct` | 30 | SOC minimal visé en fin d'heures creuses (%). |
| `marge_reserve_pct` | 10 | Marge de réserve pour l'incertitude de prévision solaire (%). |
| `fin_fenetre_matinale_h` | 14 | Heure limite de fin de fenêtre matinale (8-20). |
| `alpha_apprentissage_conso` | 0.3 | Poids d'apprentissage de la conso journalière (0.05-1). |
| `alpha_apprentissage_matinale` | 0.3 | Poids d'apprentissage de la conso matinale (0.05-1). |
| `tarif_hc` | 0.18252 | Tarif heures creuses (€/kWh). |
| `tarif_hp` | 0.24502 | Tarif heures pleines (€/kWh). |
| `marge_temps_charge_min` | 20 | Marge de sécurité sur la durée de charge (min). |
| `delai_apres_lever_min` | 30 | Délai après le lever du soleil avant solaire utile (min). |
| `heure_bilan` | 23:58:00 | Heure du bilan quotidien (apprentissage des consommations). |

## 4. Unités des prévisions solaires

Les capteurs de prévision solaire (ex. Forecast.Solar :
`sensor.energy_production_today_remaining_2`, `sensor.energy_production_tomorrow_2`)
sont en **kWh**. Le blueprint applique une multiplication par 1000 en interne
(variable `prochaine_prod_solarflow_wh`) avant toute comparaison aux seuils en Wh.
Si vos prévisions sont déjà en Wh, retirez le ×1000 dans cette variable.
