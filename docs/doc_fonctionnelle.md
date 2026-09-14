# Documentation fonctionnelle — MyData Monitoring

## Objectif
Outil interne (Streamlit) permettant à l'équipe DataOps de composer et diffuser par email des alertes sur la disponibilité des données (Power BI / Décisionnel) auprès de listes de destinataires.

## Flux utilisateur
1. Sélection de la **date des données** → calcul automatique du libellé `J-X` et de la date française.
2. Choix du **statut** :
   - `Tout OK ✅` → format "Tableau complet" (5 domaines tous disponibles) ou "Liste de rapports" (sélection).
   - `Partiel ⚠️` → multiselect des rapports KO + saisie libre + message d'alerte personnalisable.
   - `Retard Global 🚨` → tableau interactif (`st.data_editor`) pour cocher/décocher Power BI et Décisionnel par domaine.
3. **Aperçu** HTML du mail (sujet + rendu) dans un expander.
4. **Diffusion** : sélection de listes officielles (`st.secrets["DESTINATAIRES"]`) + emails manuels → champ `Bcc`.
5. **Envoi** via SMTP Gmail (`starttls`, port 587).

## Domaines métier suivis
Vente / Digital, Stock, Bornes, Détaxe, Productivité entrepôt.

## Composants clés (app.py)
- `generer_html_tableau()` : mail avec tableau statut par domaine (couleurs vert/orange).
- `generer_html_liste_ok()` : mail liste verte "intégralement disponible".
- `generer_html_orange()` : mail liste orange "partiellement disponible".
- `sauvegarder_historique(date_donnees, impact_utilisateur, ...)` : écriture d'une ligne dans `historique_alertes.csv` (schéma calé sur un Google Sheets prod). **Branchée** : appelée après un envoi SMTP réussi. `DateQS` = date des données sélectionnée ; `Date Correctif` = jour d'envoi (heure de Paris). Échec d'écriture → warning non bloquant.

## Données de référence
- `DOMAINES` : dict domaine → sous-titre HTML.
- `LISTE_RAPPORTS_BRUTE` : ~85 rapports Power BI/datasets, triés alphabétiquement + option "AUTRE".
- `LISTE_IMPACTS` : typologie d'impact utilisateur.

## Dépendances externes
- Google Sheets (via `GSheetsConnection`, importé mais usage à confirmer).
- SMTP Gmail pour l'envoi.
- Logo hébergé sur GitHub raw.

## Persistance de l'historique
- Backend actuel : CSV local (`historique_alertes.csv`), **éphémère sur Streamlit Cloud** (perdu au redémarrage du conteneur). Ignoré par git.
- Évolution recommandée : écrire l'historique via `GSheetsConnection` (déjà importé) pour un stockage durable partagé.

## Architecture cible (CLAUDE.md)
Non appliquée à ce jour : le projet est un script Streamlit unique. Une éventuelle refonte devrait extraire la logique HTML, la persistance et l'envoi mail dans des modules dédiés sous `src/` (ce qui permettrait aussi des tests unitaires, aujourd'hui impossibles car tout s'exécute au niveau module).
