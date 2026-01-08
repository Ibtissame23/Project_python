# Stratégie Core–Satellite pilotée par régimes de marché (HMM)

Ce repository contient le code et les notebooks associés au **backtest d’une stratégie core–satellite** sur le S&P 500, où l’**allocation est pilotée par des régimes de marché** détectés via un **Hidden Markov Model (HMM)**.

- **Core** : exposition structurelle à l’ETF SPY  
- **Satellite / overlay** : sur-/sous-pondération d’un panier d’actions US en fonction du régime (Bull / Bear / Neutre)  

---

## 1. Organisation du projet


### `01_HMM_101.ipynb` – Introduction pratique au HMM

Notebook pédagogique qui détaille pas à pas :

- la modélisation d’une série de rendements avec un **Hidden Markov Model** ;
- l’estimation des paramètres (moyennes de régimes, variances, matrice de transition) ;
- le décodage des **régimes Bull / Bear / Neutre** ;
- des visualisations type :  
  - probabilité filtrée des régimes,  
  - matrice de transition,  
  - trajectoires des états cachés.

> Objectif : comprendre *comment* fonctionne le modèle avant de l’utiliser en trading.

---

### `02_Trading_Strategy_HMM.ipynb` – Stratégie core–satellite & backtests

Notebook “appliqué marché” qui contient :

1. **Data loading**
   - Téléchargement des prix de **SPY** et d’un univers de grandes capitalisations US (Tech, Finance, Santé, Consommation, Énergie…)
   - Calcul des **rendements log** quotidien, alignement des séries.

2. **Détection de régimes**
   - Calibration d’un HMM à 2 états sur les rendements de SPY.
   - Classification des jours en **Bull / Bear / Neutre** via un seuil sur les probabilités de régimes.
   - Analyse des paramètres de régimes (moyennes, volatilités, matrice de transition).

3. **Scoring des actions (overlay)**
   On attribue à chaque action un **poids “cyclique”** en fonction de sa performance relative dans les régimes :
   - **Méthode 1 – Moyennes empiriques**  
     \( \mu_i^{bull}, \mu_i^{bear} \) et score \( = \max(\mu_i^{bull} - \mu_i^{bear}, 0) \)
   - **Méthode 2 – Espérance corrigée des probabilités de régimes**  
     utilisation des probabilités de régimes au lieu d’un simple comptage.
   - **Méthode 3 – Espérance ajustée au risque**  
     même approche que 2 mais divisée par la volatilité (Sharpe “bull vs bear” simplifié).

   Les scores sont ensuite **normalisés** pour obtenir un vecteur de poids relatifs sur le satellite :
   \[
   w_i^{rel} = \frac{\text{score}_i}{\sum_j \text{score}_j}
   \]

4. **Stratégie core–satellite**
   - Core : **100 % SPY** en permanence.
   - Satellite :
     - en **Bull** : overlay long \( +\alpha \) sur les actions cycliques (par ex. +30 % du portefeuille),
     - en **Bear** : overlay short \( -\beta \) (ex : -20 %) pour être underweight vs SPY,
     - en **Neutre** : overlay réduit ou nul.
   - Exposition totale potentielle : entre ~70 % et 130 % du portefeuille (levier / désengagement).

   Le poids de chaque action dans le satellite est proportionnel à son **score de régime**.

5. **Backtests**
   - **Expanding window** : on ré-estime le HMM sur tout l’historique disponible jusqu’à \(t-1\), puis on teste à \(t\).
   - **Rolling 3 ans / 3 ans** :  
     - fenêtre d’entraînement fixe (ex. 3 ans),  
     - fenêtre de test suivante (3 ans),  
     - walk-forward jusqu’à aujourd’hui.
   - Prise en compte de la structure suivante :
     - rendements core (SPY),
     - rendements overlay,
     - rendements de la stratégie totale,
     - exposition nette.

6. **Analyse vs benchmark**
   - Benchmark : **Buy & Hold SPY**, avec une **exposition ajustée dynamiquement** pour neutraliser autant que possible le gap de levier.
   - Indicateurs :
     - rendement annualisé (CAGR),
     - volatilité annualisée,
     - Sharpe ratio,
     - max drawdown,
     - exposition moyenne.

   Des graphiques Plotly sont générés :
   - courbe d’equity (stratégie vs Buy & Hold),
   - drawdown,
   - volatilité / Sharpe mobiles,
   - exposition moyenne \(|signal|\),
   - régimes HMM sur l’axe du temps.

---


