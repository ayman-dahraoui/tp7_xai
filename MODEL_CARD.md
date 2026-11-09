# Model Card - fraude (version servie par l'alias production)

## Modele
Nom et version dans le registre MLflow : fraude, version 1 (alias production)
Famille et reglages : RandomForestClassifier, n_estimators=500, max_depth=16, scikit-learn 1.9.1
Run d'origine (MLflow) : 03728f7a (experience tp2-registre)

## Objectif et usage prevu
Detecter les paiements potentiellement frauduleux a partir de 4 variables connues au moment de la decision : montant, anciennete du client (mois), nombre de transactions sur 24 h, paiement de nuit. Le modele sert d'aide a la decision : il signale les paiements a bloquer ou a verifier.

## Usages exclus
Bloquer definitivement un compte ou refuser un client sans revue humaine. Utiliser le modele sur des donnees reelles ou un autre contexte (autre pays, autre devise) sans nouvelle validation : il a ete entraine et evalue uniquement sur des donnees synthetiques.

## Donnees
Fichier, nombre de lignes, part de fraudes, empreinte (tag donnees_md5) : data/transactions.csv, 5000 lignes, 568 fraudes (11,36 %), md5 cf9f6f86c141e7df32c61715c493483f
Origine des donnees (reelles ou synthetiques) : synthetiques, generees par donnees.py (TP2, graine fixe 7)

## Performances (jeu de test, 1 250 transactions)
F1 : 0,8669   precision : 0,8837   rappel : 0,8507   AUC : 0,9872
Matrice de confusion : [[1101, 15], [20, 114]] : 114 fraudes detectees sur 134, 20 manquees, 15 paiements legitimes bloques a tort.

## Methodes d'explicabilite
Globale : importance par permutation (10 melanges, F1) et SHAP TreeExplainer sur 400 transactions de test.
Locale : SHAP (waterfall) et LIME (2000 voisins, graine 0) sur la transaction de Mme Benali (1 800 MAD, de nuit, P(fraude) = 0,924).
Variables les plus influentes : montant (permutation 0,593 ; SHAP 0,116), puis anciennete et freq_24h (~0,27 en permutation), puis nuit (0,196). Un client recent et un paiement de nuit augmentent le risque ; la contribution du montant devient positive vers 1 000 MAD.

## Comparaison avec un modele interpretable
Regression logistique avec log10(montant) : F1 0,8593, 113 fraudes detectees sur 134 (contre 114 pour la foret), soit 1 fraude d'ecart, dans le bruit du jeu de test. Arbre de profondeur 3 : F1 0,5737, lisible en 2 regles mais il manque pres de la moitie des fraudes. La logistique est lisible sans outil d'explication.

## Limites et risques connus
Donnees synthetiques : les resultats ne se transposent pas tels quels au reel (la logistique est favorisee, car le score de fraude simule utilise log10(montant)). Seulement 134 fraudes dans le jeu de test : les metriques sont bruitees. LIME est instable pour anciennete et freq_24h (le signe change selon la graine) ; SHAP decrit ce que fait le modele, pas la cause reelle de la fraude. Seulement 4 variables ; aucune analyse d'equite realisee.

## Recommandations d'utilisation et de surveillance
Revue humaine avant tout blocage definitif. Ne communiquer au client que les causes stables (montant, nuit). Surveiller la derive du montant, de la part de paiements de nuit et du taux de fraude, et reentrainer si elle apparait. Verifier que le tag modele_version du run d'explications correspond a la version portee par l'alias production.

## Tracabilite
Run des explications (experience tp7-explicabilite) : xai-fraude-v1, 7507e800a9b54d2f9e1aa7e695cb068e
Auteurs : Ayman Dahraoui (5IASD, groupe 8) Date : 09/11/2026 Prochaine revue : 02/2027