# Présentation générale du projet
Ce projet a pour objectif de développer un moteur complet de gestion et de mesure des risques d'un portefeuille multi-actifs. Il couvre les principales méthodes quantitatives utilisées en gestion des risques financiers, en commençant par l'estimation de la Value at Risk (VaR) et de l'Expected Shortfall (ES) à travers la simulation historique, puis en intégrant des approches plus avancées telles que la VaR paramétrique avec modèles EWMA et GARCH, la simulation de Monte-Carlo multi-actifs et la modélisation des dépendances par copules. La robustesse des modèles sera évaluée à l'aide de techniques de backtesting, notamment les tests de Kupiec et de Christoffersen, tandis que des stress tests et analyses de scénarios permettront d'étudier le comportement du portefeuille dans des conditions de marché extrêmes. L'ensemble de ces méthodes sera finalement intégré dans un Portfolio Risk Engine capable de centraliser les données de marché, mesurer les différentes sources de risque, analyser les pertes potentielles et produire des indicateurs utiles à la prise de décision et au suivi du risque d'un portefeuille.
# Partie 1 — VaR et ES historique avec EWMA et GARCH
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



# Partie 2 — VaR Paramétrique avec EWMA et GARCH

Dans cette partie du projet, nous allons utiliser une **approche paramétrique dynamique** pour calculer la VaR. Contrairement à une VaR paramétrique classique utilisant une volatilité constante, nous allons modéliser la volatilité avec deux méthodes :

- **EWMA-Exponentially Weighted Moving Average**
- **GARCH-Generalized Autoregressive Conditional Heteroskedasticity**

L'objectif est de tenir compte d'une caractéristique fondamentale des marchés financiers :

$$
\boxed{\text{La volatilité évolue dans le temps}}
$$

Les périodes calmes présentent généralement de faibles variations des prix, tandis que les périodes de crise présentent des mouvements beaucoup plus importants.

Les modèles **EWMA** et **GARCH** permettent précisément de capturer cette dynamique.

## 2.1. Principe général de la VaR paramétrique

Soit $P_t$ la valeur d'un actif ou d'un portefeuille à la date $t$.

Le rendement simple est défini par :

$$
r_t = \frac{P_t-P_{t-1}}{P_{t-1}}
$$

Dans la suite du projet, nous utiliserons principalement les rendements logarithmiques:

$$\boxed{r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)}$$

où:

- $P_t$ représente la valeur du portefeuille à la date $t$ ;
- $P_{t-1}$ représente sa valeur à la date précédente ;
- $r_t$ représente le rendement entre $t-1$ et $t$

Dans une approche paramétrique classique, on suppose que le rendement conditionnel suit une loi normale:

$$r_{t+1} \sim \mathcal{N}\left(\mu_{t+1},\sigma_{t+1}^2\right)$$

avec:

- $\mu_{t+1}$ : rendement moyen anticipé ;
- $\sigma_{t+1}$ : volatilité prévue ;
- $\mathcal{N}$ : distribution normale.

Pour un portefeuille de valeur $V_t$, la VaR au niveau de confiance $c$ peut être écrite :

$$\boxed{VaR_{t+1}(c)=V_t\left(-\mu_{t+1}+z_c\sigma_{t+1}\right)}$$

où $z_c$ est le quantile associé au niveau de confiance choisi.

Quelques quantiles usuels:

| Niveau de confiance | Quantile $z_c$ |
|---|---:|
| 90 % | 1.282 |
| 95 % | 1.645 |
| 97.5 % | 1.960 |
| 99 % | 2.326 |

Pour des rendements quotidiens, la moyenne est souvent très faible par rapport à la volatilité. On peut donc parfois utiliser l'approximation :

$$\mu \approx 0$$

ce qui donne :

$$\boxed{VaR_{t+1}(c) \approx V_t z_c \sigma_{t+1}}$$

Le problème principal devient alors l'estimation de la volatilité future:

$$\boxed{\sigma_{t+1}}$$

C'est précisément le rôle des modèles **EWMA** et **GARCH**.


## Pourquoi utiliser une volatilité dynamique ?

Une approche simple consiste à utiliser l'écart-type historique des rendements :

$$\sigma=\sqrt{\frac{1}{N-1}\sum_{i=1}^{N}(r_i-\bar r)^2}$$

Cette méthode donne cependant la même importance à toutes les observations historiques.

Or, sur les marchés financiers, la volatilité n'est pas constante.

On observe généralement des périodes de :

- faible volatilité ;
- forte volatilité ;
- crises ;
- retour progressif vers un régime plus calme.

Ce phénomène est appelé **volatility clustering**.

En pratique :

$$\boxed{\text{forte volatilité aujourd'hui}\Rightarrow\text{probabilité élevée de forte volatilité demain}}$$

Les modèles EWMA et GARCH permettent de tenir compte de cette dépendance temporelle.


## 2.2. Modèle EWMA

## 2.2.1 Définition

Le modèle **EWMA — Exponentially Weighted Moving Average** donne davantage d'importance aux observations récentes qu'aux observations anciennes.

La variance conditionnelle est définie par :

$$ \boxed{ \sigma_t^2=\lambda\sigma_{t-1}^{2}+(1-\lambda)r_{t-1}^{2}}$$

où :

- $\sigma_t^2$ : variance estimée à la date $t$ ;
- $\sigma_{t-1}^2$ : variance estimée à la date précédente ;
- $r_{t-1}^2$ : carré du dernier rendement observé ;
- $\lambda$ : facteur de décroissance.

Une valeur couramment utilisée pour des données quotidiennes est :

$$ \boxed{\lambda = 0.94}$$

L'équation devient alors :

$$ \sigma_t^2=0.94\sigma_{t-1}^{2}+0.06r_{t-1}^{2} $$

Cela signifie que la nouvelle estimation dépend :

- à **94 %** de la variance précédente ;
- à **6 %** du dernier choc observé sur le marché.

## 2.1.2 Prévision EWMA à un jour

La variance prévue pour le jour suivant est :

$$\boxed{ \sigma_{t+1}^{2}=\lambda\sigma_t^2+(1-\lambda)r_t^2}$$

La volatilité correspondante est :

$$\boxed{ \sigma_{t+1} =\sqrt{\lambda\sigma_t^2+(1-\lambda)r_t^2}}$$

Une forte variation récente augmente donc automatiquement la volatilité prévue.



## 2.2.3 VaR avec EWMA

Sous l'hypothèse d'une distribution normale :

$$ VaR_{t+1}(c)=V_t\left(-\mu+z_c\sigma_{t+1}\right)$$

Au niveau de confiance de 99 % :

$$ \boxed{ VaR_{99\%,t+1}= V_t \left(-\mu + 2.326\sigma_{t+1}\right)}$$

Si l'on suppose :

$$ \mu \approx 0 $$

alors:

$$\boxed{ VaR_{99\%,t+1}\approx2.326\,V_t\,\sigma_{t+1}}$$


# 2.3 Modèle GARCH

## 2.3.1 Définition

Le modèle **GARCH-Generalized Autoregressive Conditional Heteroskedasticity** permet de modéliser explicitement l'évolution de la variance dans le temps.

Le modèle le plus couramment utilisé est :

$$
\boxed{\text{GARCH}(1,1)}
$$

Le rendement peut être écrit :

$$
r_t = \mu + \epsilon_t
$$

avec :

$$
\epsilon_t = \sigma_t z_t
$$

où, dans le cas normal :

$$
z_t \sim \mathcal{N}(0,1)
$$

La variance conditionnelle suit :

$$
\boxed{
\sigma_t^2
=
\omega
+
\alpha\epsilon_{t-1}^{2}
+
\beta\sigma_{t-1}^{2}
}
$$

---

## Interprétation des paramètres

Le paramètre $\omega$ représente la composante constante de la variance.

Le paramètre $\alpha$ mesure la réaction de la volatilité aux nouveaux chocs :

$$
\alpha\epsilon_{t-1}^{2}
$$

Un $\alpha$ élevé signifie que la volatilité réagit fortement aux nouvelles variations du marché.

Le paramètre $\beta$ mesure la persistance de la volatilité passée :

$$
\beta\sigma_{t-1}^{2}
$$

Un $\beta$ élevé signifie qu'un épisode de forte volatilité peut persister pendant plusieurs périodes.



## 2.3.2 Persistance du modèle

La persistance de la volatilité est mesurée par :

$$
\boxed{
\alpha+\beta
}
$$

Pour un modèle GARCH(1,1) stationnaire, on recherche généralement:

$$
\boxed{
\alpha+\beta<1
}
$$

Par exemple :

$$
\alpha=0.08
$$

$$
\beta=0.90
$$

donne :

$$
\alpha+\beta=0.98
$$

La volatilité est donc très persistante.

---

## 2.2.3 Variance de long terme

Lorsque:

$$
\alpha+\beta<1
$$

la variance de long terme est :

$$
\boxed{
\sigma_{\infty}^{2}
=
\frac{\omega}
{1-\alpha-\beta}
}
$$

La volatilité de long terme vaut donc :

$$
\boxed{
\sigma_{\infty}
=
\sqrt{
\frac{\omega}
{1-\alpha-\beta}
}
}
$$

Cette propriété constitue une différence importante entre EWMA et GARCH.

# Relation entre EWMA et GARCH

Le modèle GARCH(1,1) est défini par :

$$
\sigma_t^2
=
\omega
+
\alpha\epsilon_{t-1}^{2}
+
\beta\sigma_{t-1}^{2}
$$

Le modèle EWMA est :

$$
\sigma_t^2
=
(1-\lambda)r_{t-1}^{2}
+
\lambda\sigma_{t-1}^{2}
$$

EWMA peut donc être vu comme un cas particulier du modèle GARCH lorsque :

$$
\boxed{
\omega=0
}
$$

$$
\boxed{
\alpha=1-\lambda
}
$$

$$
\boxed{
\beta=\lambda
}
$$

Avec :

$$
\lambda=0.94
$$

on obtient :

$$
\alpha=0.06
$$

et :

$$
\beta=0.94
$$

La différence essentielle est que GARCH permet généralement d'**estimer les paramètres à partir des données**, tandis que le paramètre $\lambda$ d'EWMA est souvent fixé à l'avance.

---

# 2.3.4 VaR avec GARCH

Une fois le modèle estimé, nous obtenons la prévision de volatilité :

$$
\sigma_{t+1}
$$

Sous une distribution normale :

$$
\boxed{
VaR_{t+1}(c)
=
V_t
\left[
-\mu_{t+1}
+
z_c\sigma_{t+1}
\right]
}
$$

Pour un niveau de confiance de 99 % :

$$
\boxed{
VaR_{99\%,t+1}
=
V_t
\left(
-\mu_{t+1}
+
2.326\sigma_{t+1}
\right)
}
$$

# 2.3.5 GARCH avec distribution Student-t

L'hypothèse normale peut sous-estimer les événements extrêmes car les rendements financiers présentent souvent des **queues épaisses**.

Une alternative consiste à supposer :

$$
z_t \sim t_{\nu}
$$

où $\nu$ représente le nombre de degrés de liberté de la distribution de Student.

Nous obtenons alors :

$$
\boxed{
\text{GARCH}(1,1)\text{-Student-t}
}
$$

Cette variante peut être plus adaptée au calcul d'une VaR à des niveaux de confiance élevés tels que **99 %**.


                  |
                  v
         VaR paramétrique
                  |
                  v
         Perte potentielle
