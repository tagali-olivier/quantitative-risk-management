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
- $r_{i,t}$ représente le rendement de l'actif $i$ entre les dates $t-1$ et $t$.

Pour chaque actif $i$, le PnL à la date $t$ est calculé à partir de la valeur de la position et du rendement observé :

$$
\mathrm{PnL}_{i,t} = V_{i,t-1} r_{i,t}
$$

où $V_{i,t-1}$ représente la valeur de la position dans l'actif $i$ à la date $t-1$, et $r_{i,t}$ son rendement entre les dates $t-1$ et $t$.

Le PnL total du portefeuille est obtenu en additionnant les PnL de l'ensemble des actifs :

$$
\mathrm{PnL}_{p,t} = \sum_{i=1}^{N} \mathrm{PnL}_{i,t}
$$

En remplaçant le PnL de chaque actif par son expression, on obtient :

$$
\mathrm{PnL}_{p,t} = \sum_{i=1}^{N} V_{i,t-1} r_{i,t}
$$
