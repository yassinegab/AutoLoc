# Stratégie de chargement JPA : LAZY et EAGER

## 1. Contexte

Dans le projet AutoLoc, un contrat peut être associé à plusieurs paiements. Cette relation est définie entre les entités `Contrat` et `Paiement`.

Dans l'entité `Contrat`, l'association est configurée avec :

```java
@OneToMany(
    mappedBy = "contrat",
    fetch = FetchType.LAZY,
    cascade = CascadeType.ALL,
    orphanRemoval = true
)
private List<Paiement> paiements = new ArrayList<>();
```

## 2. FetchType.LAZY

Avec `FetchType.LAZY`, la liste des paiements n'est chargée que lorsque le code y accède explicitement, par exemple avec `contrat.getPaiements()`.

**Avantages :**

* Évite de charger des données inutiles.
* Réduit le coût de certaines lectures.
* Améliore potentiellement les performances lorsque les paiements ne sont pas nécessaires.

**Attention :** l'accès à une collection LAZY après la fermeture du contexte de persistance peut provoquer une `LazyInitializationException`.

## 3. FetchType.EAGER

Avec `FetchType.EAGER`, les paiements doivent être chargés avec le contrat, même si le programme n'en a pas besoin immédiatement.

**Avantage :**

* Les données associées sont disponibles dès le chargement de l'entité.

**Inconvénients :**

* Peut charger des données inutiles.
* Peut alourdir les lectures et augmenter le coût des requêtes.

## 4. Comparaison

| Critère                               | LAZY                                                     | EAGER                                  |
| ------------------------------------- | -------------------------------------------------------- | -------------------------------------- |
| Chargement des paiements              | À la demande                                             | Obligatoire avec le contrat            |
| Données inutiles chargées             | Limitées en général                                      | Risque plus élevé                      |
| Performances                          | Adapté si les paiements ne sont pas toujours nécessaires | Peut alourdir les lectures             |
| Risque de LazyInitializationException | Oui, si l'accès se fait hors du contexte de persistance  | Pas pour cette collection déjà chargée |

## 5. Choix retenu dans AutoLoc

Nous retenons `FetchType.LAZY` pour l'association `Contrat` → `Paiement`, car les paiements ne sont pas nécessairement nécessaires à chaque consultation d'un contrat.

Cette stratégie permet de charger les données associées uniquement lorsque cela est utile. Lorsqu'une application doit retourner les paiements dans une réponse API, leur chargement doit être organisé dans le contexte de persistance approprié.

## Conclusion

`LAZY` privilégie le chargement à la demande, tandis que `EAGER` impose le chargement des données associées. Dans AutoLoc, `LAZY` est retenu pour limiter les chargements inutiles et mieux maîtriser les performances.
