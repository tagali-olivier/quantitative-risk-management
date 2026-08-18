# Présentation générale du projet
Ce projet a pour objectif de développer un moteur complet de gestion et de mesure des risques d'un portefeuille multi-actifs. Il couvre les principales méthodes quantitatives utilisées en gestion des risques financiers, en commençant par l'estimation de la Value at Risk (VaR) et de l'Expected Shortfall (ES) à travers la simulation historique, puis en intégrant des approches plus avancées telles que la VaR paramétrique avec modèles EWMA et GARCH, la simulation de Monte-Carlo multi-actifs et la modélisation des dépendances par copules. La robustesse des modèles sera évaluée à l'aide de techniques de backtesting, notamment les tests de Kupiec et de Christoffersen, tandis que des stress tests et analyses de scénarios permettront d'étudier le comportement du portefeuille dans des conditions de marché extrêmes. L'ensemble de ces méthodes sera finalement intégré dans un Portfolio Risk Engine capable de centraliser les données de marché, mesurer les différentes sources de risque, analyser les pertes potentielles et produire des indicateurs utiles à la prise de décision et au suivi du risque d'un portefeuille.

# 1. Définition du portefeuille

Le projet repose sur un **portefeuille multi-actifs** afin d'étudier différentes sources de risque de marché ainsi que les effets de diversification.

Le portefeuille est composé d'**actions américaines et européennes**, d'**obligations souveraines**, de **matières premières** et d'une exposition au **marché des changes (FX)**.

## Composition du portefeuille

| Actif | Classe d'actifs | Poids |
|---|---|---:|
| S&P 500 | Actions US | 25 % |
| EURO STOXX 50 | Actions Europe | 15 % |
| NASDAQ 100 | Actions Tech US | 10 % |
| US Treasury 10Y | Obligations US | 15 % |
| Bund 10Y | Obligations Europe | 10 % |
| Or | Matières premières | 10 % |
| Pétrole | Matières premières | 10 % |
| EUR/USD | Change (FX) | 5 % |
| **Total** | | **100 %** |

Cette allocation permet de prendre en compte plusieurs catégories de risque :

- **Risque actions**: S&P 500, EURO STOXX 50 et NASDAQ 100 ;
- **Risque de taux d'intérêt**: US Treasury 10Y et Bund 10Y ;
- **Risque sur matières premières**: Or et Pétrole ;
- **Risque de change**: EUR/USD.

## 2. Calcul des rendements et de la valeur du portefeuille

A partir des données historiques de prix, nous calculons dans un premier temps le **rendement de chaque actif** du portefeuille.

Pour un actif $i$, le rendement entre les dates $t-1$ et $t$ est défini par :

$$
r_{i,t} = \frac{P_{i,t}}{P_{i,t-1}} - 1
$$

où :

- $P_{i,t}$ représente le prix de l'actif $i$ à la date $t$ ;
- $P_{i,t-1}$ représente le prix de l'actif $i$ à la date précédente ;
- $r_{i,t}$ représente le rendement de l'actif $i$ à la date $t$.

Une fois les rendements calculés pour l'ensemble des actifs, le **rendement du portefeuille** à la date $t$ est obtenu en tenant compte du poids $w_i$ de chaque actif :

$$
r_{p,t} = \sum_{i=1}^{N} w_i r_{i,t}
$$

où $N$ représente le nombre d'actifs composant le portefeuille et $w_i$ le poids de l'actif $i$.

La **valeur du portefeuille à la date $t$** peut ensuite être déterminée à partir de sa valeur à la période précédente :

$$
V_t = V_{t-1}(1+r_{p,t})
$$

où $V_{t-1}$ représente la valeur du portefeuille à la période précédente et $r_{p,t}$ son rendement entre $t-1$ et $t$.

Si les quantités $q_i$ détenues de chaque actif sont connues, la valeur du portefeuille peut également être calculée directement à partir des prix :

$$
V_t = \sum_{i=1}^{N} q_i P_{i,t}
$$

Ces calculs permettent de suivre l'évolution historique de la valeur du portefeuille et serviront ensuite à construire la distribution des gains et pertes utilisée pour le calcul de la **Value at Risk (VaR)** et de l'**Expected Shortfall (ES)**.
