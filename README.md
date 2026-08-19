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


## 3. Calcul de la Value at Risk historique

La série des PnL obtenue constitue la **distribution historique des gains et pertes du portefeuille**.

Pour un niveau de confiance $\alpha$, la Value at Risk est déterminée à partir de la partie gauche de cette distribution, qui correspond aux scénarios de pertes les plus importantes.

Pour un niveau de confiance $\alpha$, la VaR historique est déterminée à partir du quantile de niveau $1-\alpha$ de la distribution des PnL du portefeuille.

On détermine alors le quantile correspondant de la distribution historique des PnL :

$$
Q_{1-\alpha}\left(\mathrm{PnL}_{p}\right)
$$

Et la value at Risk est donc:

$$ \mathrm{VaR}_{\alpha}=- Q_{1-\alpha}\left(\mathrm{PnL}_{p}\right)$$


## 4. Expected Shortfall (ES)

L'**Expected Shortfall (ES)** est une mesure de risque qui complète la **Value at Risk (VaR)**.

La VaR indique un **seuil de perte** associé à un niveau de confiance donné. Elle permet d'identifier le niveau de perte à partir duquel se situent les scénarios les plus défavorables.

Elle répond donc à la question : **À partir de quel niveau considère-t-on qu'une perte devient extrême ?**

Cependant, la VaR ne donne aucune information sur la gravité des pertes situées **au-delà de ce seuil**. Deux portefeuilles peuvent ainsi présenter une même VaR tout en ayant des pertes extrêmes très différentes.

L'**Expected Shortfall** cherche précisément à mesurer cette gravité. Il correspond à la **perte moyenne dans les scénarios où la VaR est dépassée**. Autrement dit, on sélectionne les observations appartenant à la queue gauche de la distribution des PnL, puis on calcule leur moyenne.

Si la VaR est exprimée comme une perte positive, le seuil correspondant dans la distribution des PnL est :

$$
-\mathrm{VaR}_{\alpha}
$$

Les scénarios extrêmes sont alors ceux qui vérifient :

$$
\mathrm{PnL}_{p} \leq -\mathrm{VaR}_{\alpha}
$$

L'Expected Shortfall est donc défini par :

$$ \mathrm{ES}_{\alpha}=-\mathbb{E} \left[\mathrm{PnL}_{p}\mid\mathrm{PnL}_{p} \leq -\mathrm{VaR}_{\alpha}\right]$$

où $\alpha$ représente le niveau de confiance choisi.

Ainsi, la **VaR** détermine le seuil à partir duquel les pertes sont considérées comme extrêmes, tandis que l'**Expected Shortfall** mesure la perte moyenne lorsque ce seuil est dépassé.

```python

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt



#Paramètres

PRICE_FILE = "data/prices.csv"

VALUATION_DATE = "2025-12-31"

LOOKBACK_DAYS = 500

CONFIDENCE_LEVELS = [0.95, 0.99]


#Definition du portefeuille


weights = pd.Series({
    "SP500": 0.25,
    "EUROSTOXX50": 0.15,
    "NASDAQ100": 0.10,
    "UST10Y": 0.15,
    "BUND10Y": 0.10,
    "GOLD": 0.10,
    "OIL": 0.10,
    "EURUSD": 0.05
})

#Valeur totale du portefeuille à la date de valorisation
portfolio_value = 1_000_000

# Expositions actuelles
market_values = weights * portfolio_value



#CHARGEMENT DES DONNÉES

prices = pd.read_csv(
    PRICE_FILE,
    parse_dates=["Date"],
    index_col="Date"
)

prices = prices.sort_index()

print("Aperçu des données :")
print(prices.head())


# ============================================================
# 4. CONTRÔLE DES ACTIFS
# ============================================================

required_assets = weights.index.tolist()

missing_assets = [
    asset
    for asset in required_assets
    if asset not in prices.columns
]

if missing_assets:
    raise ValueError(
        f"Actifs manquants dans le fichier de prix : {missing_assets}"
    )


# ============================================================
# 5. FILTRAGE À LA DATE DE VALORISATION
# ============================================================

valuation_date = pd.Timestamp(VALUATION_DATE)

prices = prices.loc[
    prices.index <= valuation_date,
    required_assets
]

if prices.empty:
    raise ValueError(
        "Aucune donnée disponible avant la date de valorisation."
    )


# ============================================================
# 6. CALCUL DES RENDEMENTS HISTORIQUES
# ============================================================

returns = prices.pct_change().dropna()

# On ne conserve que la fenêtre historique choisie
if len(returns) > LOOKBACK_DAYS:
    returns = returns.tail(LOOKBACK_DAYS)

print("\nNombre de scénarios historiques :")
print(len(returns))


# ============================================================
# 7. EXAMEN DES RENDEMENTS
# ============================================================

print("\nStatistiques des rendements :")
print(returns.describe())


# ============================================================
# 8. CALCUL DES P&L PAR ACTIF
# ============================================================

# P&L actif par actif :
# P&L_i,t = exposition_i * rendement_i,t

asset_pnl = returns.mul(
    market_values,
    axis=1
)

print("\nExemple de P&L par actif :")
print(asset_pnl.head())


# ============================================================
# 9. CALCUL DU P&L DU PORTEFEUILLE
# ============================================================

portfolio_pnl = asset_pnl.sum(axis=1)

print("\nExemple de P&L portefeuille :")
print(portfolio_pnl.head())


# ============================================================
# 10. FONCTION HISTORICAL VAR
# ============================================================

def historical_var(
    pnl: pd.Series,
    confidence_level: float = 0.99
) -> float:

    alpha = 1 - confidence_level

    pnl_quantile = pnl.quantile(alpha)

    var = -pnl_quantile

    return var


# ============================================================
# 11. FONCTION EXPECTED SHORTFALL
# ============================================================

def historical_es(
    pnl: pd.Series,
    confidence_level: float = 0.99
) -> float:

    alpha = 1 - confidence_level

    threshold = pnl.quantile(alpha)

    tail_losses = pnl[
        pnl <= threshold
    ]

    es = -tail_losses.mean()

    return es


# ============================================================
# 12. CALCUL VAR + ES
# ============================================================

results = []

for confidence in CONFIDENCE_LEVELS:

    var_value = historical_var(
        portfolio_pnl,
        confidence
    )

    es_value = historical_es(
        portfolio_pnl,
        confidence
    )

    results.append({
        "Confidence Level": confidence,
        "VaR": var_value,
        "ES": es_value,
        "VaR %": var_value / portfolio_value,
        "ES %": es_value / portfolio_value
    })


risk_results = pd.DataFrame(results)

print("\nRÉSULTATS HISTORICAL VAR & ES")
print(risk_results)


# ============================================================
# 13. AFFICHAGE FORMATÉ
# ============================================================

print("\n=======================================")
print("   HISTORICAL RISK REPORT")
print("=======================================")

print(
    f"Valuation Date      : {VALUATION_DATE}"
)

print(
    f"Portfolio Value     : {portfolio_value:,.2f} €"
)

print(
    f"Historical Scenarios: {len(portfolio_pnl)}"
)

for confidence in CONFIDENCE_LEVELS:

    var_value = historical_var(
        portfolio_pnl,
        confidence
    )

    es_value = historical_es(
        portfolio_pnl,
        confidence
    )

    print(
        f"\nVaR {confidence:.0%}  : "
        f"{var_value:,.2f} €"
    )

    print(
        f"VaR {confidence:.0%} %: "
        f"{var_value / portfolio_value:.2%}"
    )

    print(
        f"ES {confidence:.0%}   : "
        f"{es_value:,.2f} €"
    )

    print(
        f"ES {confidence:.0%} % : "
        f"{es_value / portfolio_value:.2%}"
    )


# ============================================================
# 14. PIRE SCÉNARIO HISTORIQUE
# ============================================================

worst_date = portfolio_pnl.idxmin()

worst_pnl = portfolio_pnl.loc[
    worst_date
]

print("\n=======================================")
print("   WORST HISTORICAL SCENARIO")
print("=======================================")

print(
    f"Worst Date : {worst_date.date()}"
)

print(
    f"Worst P&L  : {worst_pnl:,.2f} €"
)


# ============================================================
# 15. CONTRIBUTIONS AU PIRE SCÉNARIO
# ============================================================

worst_contributions = (
    asset_pnl
    .loc[worst_date]
    .sort_values()
)

print("\nContributions par actif :")
print(worst_contributions)


# ============================================================
# 16. TOP 10 DES PIRES SCÉNARIOS
# ============================================================

worst_10 = (
    portfolio_pnl
    .sort_values()
    .head(10)
)

print("\n10 pires scénarios :")
print(worst_10)


# ============================================================
# 17. VISUALISATION DISTRIBUTION P&L
# ============================================================

var_99 = historical_var(
    portfolio_pnl,
    0.99
)

es_99 = historical_es(
    portfolio_pnl,
    0.99
)

fig, ax = plt.subplots(
    figsize=(12, 6)
)

ax.hist(
    portfolio_pnl,
    bins=60
)

ax.axvline(
    -var_99,
    linestyle="--",
    label="VaR 99%"
)

ax.axvline(
    -es_99,
    linestyle="--",
    label="ES 99%"
)

ax.set_title(
    "Historical Simulation - Portfolio P&L Distribution"
)

ax.set_xlabel(
    "Daily P&L (€)"
)

ax.set_ylabel(
    "Frequency"
)

ax.legend()

plt.show()


# ============================================================
# 18. VISUALISATION DES CONTRIBUTIONS
# ============================================================

fig, ax = plt.subplots(
    figsize=(10, 6)
)

worst_contributions.plot(
    kind="bar",
    ax=ax
)

ax.set_title(
    f"Asset Contributions - Worst Scenario ({worst_date.date()})"
)

ax.set_ylabel(
    "P&L Contribution (€)"
)

plt.xticks(
    rotation=45
)

plt.tight_layout()

plt.show()


# ============================================================
# 19. ROLLING HISTORICAL VAR
# ============================================================

def rolling_historical_var(
    pnl: pd.Series,
    window: int = 250,
    confidence_level: float = 0.99
) -> pd.Series:

    alpha = 1 - confidence_level

    rolling_var = (
        pnl
        .rolling(window)
        .quantile(alpha)
        * -1
    )

    return rolling_var


rolling_var_99 = rolling_historical_var(
    portfolio_pnl,
    window=250,
    confidence_level=0.99
)


# ============================================================
# 20. VISUALISATION ROLLING VAR
# ============================================================

fig, ax = plt.subplots(
    figsize=(12, 6)
)

rolling_var_99.plot(
    ax=ax
)

ax.set_title(
    "Rolling Historical VaR 99%"
)

ax.set_ylabel(
    "VaR (€)"
)

ax.set_xlabel(
    "Date"
)

plt.show()

```
