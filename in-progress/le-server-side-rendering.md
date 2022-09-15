---
title: Le Server Side Rendering
subtitle: C'est quoi, et pourquoi ?
createdAt: '2022-09-10T00:00:00'
cover:
  src: https://images.pexels.com/photos/26681/pexels-photo-26681.jpg
  alt: Unrelated picture of the sea, by Clem Onojeghuo
lang: fr
withMermaid: true
---

Si vous suivez les dernières méthodes pour créer des sites et des applications web, vous avez sans doute déjà entendu parler de _Server Side Rendering_, de _Hydration_ et autres _Island Architecture_. Si ce n'est pas le cas, c'est le bon article pour vous. Si c'est le cas, restez si vous voulez en apprendre un peu plus !

## Avant de se lancer dans le SSR

Parlons d'abord de CSR et de SPA (j'adore les acronymes, ça rend tout tellement plus clair 🙃)

### Les acronymes très clairs

CSR, ça veux dire _Client Side Rendering_, donc des applications web qui n'ont pas besoin d'un serveur pour afficher leurs données une fois qu'elles sont téléchargée sur un navigateur. (Il faut forcément un serveur dans l'histoire, ne serais-ce que pour récupérer les scripts et les assets)

SPA, ça veux dire _Single Page Application_, mais ça ne veux pas dire que toute votre application tient sur une seule page. Enfin presque.  
Une SPA c'est une seule page HTML, le plus généralement vide, qui vient avec des scripts Javascript qui s'exécutent afin de mettre en fonction l'application.

C'est ce principe qui est très utilisé depuis la démocratisation des frameworks JS. Angular fonctionne sur ce principe et les autres copains (React, Vue etc.) peuvent aussi faire pareil si ils ne sont pas utilisé comme des librairies façon JQuery.

### Fonctionnement d'une SPA

A ma connaissance, il existe très peu d'applications web CSR qui ne sont pas des SPA donc par la suite j'utiliserais ces deux termes de manière interchangeable.

Comme dit plus haut, pour qu'une SPA fonctionne, il faut que l'utilisateur récupère une page HTML vierge, puis que cette page demande à récupérer le script pour lancer l'application.

Une fois récupéré, ce script va pouvoir commencer à afficher des informations à l'écran et peut-être aussi récupérer encore d'autres scripts pour afficher d'autres pages, composants etc.

On peux faire un diagramme pour se représenter le fonctionnement :

::: noparse div mermaid
sequenceDiagram
  participant Utilisateur
  participant Application
  participant Navigateur
  participant Serveur
  Utilisateur->>Navigateur: Ouvre le lien de l'application
  Navigateur->>Serveur: Demande la page
  Serveur->>Navigateur: Renvoi une page HTML
  Note right of Navigateur: Le HTML contient une balise script
  Navigateur->>Serveur: Demande le script
  Serveur->>Navigateur: Renvoi le script
  Navigateur->>Application: Exécute le script
  Note over Application: L'application s'affiche à l'écran
  Utilisateur->>Application: Change de page
  Note over Application: L'application affiche une nouvelle page
:::

Ici, je fait la distinction entre le _navigateur_ et _l'application_ :

- Le navigateur c'est le fonctionnement "de base" d'un page web, tel que le navigateur l'exécute (charger des scripts, se rafraîchir, naviguer vers une URL etc.)
- L'application, c'est le fonctionnement particulier que permet du Javascript seulement. Parfois, on cherche à émuler le fonctionnement naturel du navigateur.

Quand un utilisateur _change de page_, on imagine qu'il clique sur un lien ou un onglet ou quoi que ce soit qui l'amène vers un contexte différent.  
Il faut noter qu'ici, le navigateur ne change pas de page ! C'est simplement l'application qui va changer son contenu pour afficher autre chose (et la plupart du temps, elle va aussi changer l'URL pour émuler un changement de page traditionnel)

Si vous avez fait des applications web ces dix dernières années, il y a de fortes chances pour que ce soit ce système que vous avez utilisé.

## Le Server Side Rendering

Pour revenir aux bases, le SSR c'est un nom très ~~pompeux~~ savant pour parler du fonctionnement **de base** d'Internet. C'est comme ça que les sites web fonctionnaient et aurait continué de fonctionner s'il n'y avait jamais eu de script coté client. (d'ailleurs, pour faire le parallèle avec les SPA, on appelle maintenant ce système MPA pour _Multi Page Application_ -> un site internet donc)

Avant Javascript, c'était PHP qui était sur le devant de la scène, et PHP c'était... Du SSR ! Si vous faites des sites en PHP depuis 20 ans vous faites **aussi** parti des _cool kids_ 😎

Si on reprend le diagramme du dessus à la mode PHP (SSR) ça nous donne :

<div className="mermaid">
sequenceDiagram
    participant Utilisateur
    participant Navigateur
    participant Serveur
    Utilisateur->>Navigateur: Ouvre le lien de l'application
    Navigateur->>Serveur: Demande la page
    Serveur->>Navigateur: Renvoi une page HTML formée
    Note over Navigateur: L'application s'affiche à l'écran
    Utilisateur->>Navigateur: Change de page
    Navigateur->>Serveur: Demande la page
    Serveur->>Navigateur: Renvoi une page HTML formée
    Note over Navigateur: L'application affiche une nouvelle page
</div>

Contrairement à une SPA, je n'ai pas fait mention de "l'application" car le contenu retourné par le serveur au navigateur est déjà fonctionnel.  
Un changement de page est géré nativement par le navigateur et réitère une demande au serveur.

Lors de l'arrivée du Javascript (mais avant le "boom"), il y avait aussi des script exécutés sur les pages pour rendre le tout un peu dynamique.

<details>
<summary>Ici le diagramme si on prend ces scripts en compte</summary>

<div className="mermaid">
sequenceDiagram
    participant Utilisateur
    participant Navigateur
    participant Serveur
    Utilisateur->>Navigateur: Ouvre le lien de l'application
    Navigateur->>Serveur: Demande la page
    Serveur->>Navigateur: Renvoi une page HTML formée
    Note over Navigateur: L'application s'affiche à l'écran
    Navigateur->>Serveur: Demande le script
    Serveur->>Navigateur: Renvoi le script
    Note over Navigateur: Exécute le script
    Utilisateur->>Navigateur: Change de page
    Navigateur->>Serveur: Demande la page
    Serveur->>Navigateur: Renvoi une page HTML formée
    Note over Navigateur: L'application affiche une nouvelle page
    Navigateur->>Serveur: Demande le script
    Serveur->>Navigateur: Renvoi le script
    Note over Navigateur: Exécute le script
</div>

A noter que dans la plupart des cas, ces scripts ne sont pas bloquant donc l'utilisateur peut changer de page avant de recevoir les scripts.

</details>

## Pourquoi on revient en arrière alors ?

C'est vrai ça, on a quitté les applications SSR parce qu'elles étaient trop statiques (principalement), et si saupoudrer du Javascript par dessus aidait un peu, ça ne permettait pas d'offrir des applications aussi riche que ce qu'il est possible d'obtenir avec ~~trop~~ plus de Javascript.

Maintenant, est-ce qu'on ne va pas retrouver les mêmes problématiques si on revient au SSR ? Et pourquoi ça n'irait pas les SPA hein ?

Dans nos diagrammes, on peut voir que les deux techniques ont des avantages et des inconvénients.  
Une application CSR a besoin de charger une page HTML, qui va charger un script, qui va s'exécuter avant d'afficher la page là où une application SSR va afficher directement la page toute prête  
A coté de ça, une application CSR peux changer de page instantanément sans être obligé de faire un aller retour par le réseau comme doit le faire une application SSR.

Le problème c'est que notre cas d'exemple ne se concentre que sur une application CSR qui ne fonctionnerais qu'en isolement. Si vous avez fait une application CSR, vous l'avez très probablement branchée à une API pour accéder à une base de données par exemple.

### Un exemple plus réel

Reprenons les deux diagrammes avec une lecture et une sauvegarde d'une entité en base de données (Et une redirection vers une liste à la sauvegarde).

Tout d'abord en SSR :

<div className="mermaid">
sequenceDiagram
    participant Utilisateur
    participant Navigateur
    participant Serveur
    Utilisateur->>Navigateur: Ouvre le lien de l'application
    Navigateur->>Serveur: Demande la page
    Note over Serveur: Récupère la liste des entités en base
    Serveur->>Navigateur: Renvoi une page HTML avec la liste
    Utilisateur->>Navigateur: Ouvre la page d'une entité
    Navigateur->>Serveur: Demande la page
    Note over Serveur: Récupère l'entité en base
    Serveur->>Navigateur: Renvoi une page HTML avec les informations
    Note over Utilisateur: Modifie les champs
    Utilisateur->>Navigateur: Clique sur Sauvegarder
    Navigateur->>Serveur: Envoi les nouvelles données
    Note over Serveur: Modifie l'entité en base
    Note over Serveur: Récupère la liste des entités en base
    Serveur->>Navigateur: Renvoi une page HTML avec la liste
</div>

On peut voir que le changement n'est pas très conséquent. Puisqu'on fait un aller-retour serveur pour changer de page, on en profite pour récupérer les informations depuis la base de données pour former la page qu'on va renvoyer.

Regardons maintenant ce fonctionnement avec une SPA :

<div className="mermaid">
sequenceDiagram
    participant Utilisateur
    participant Application
    participant Navigateur
    participant Serveur
    Utilisateur->>Navigateur: Ouvre le lien de l'application
    Navigateur->>Serveur: Demande la page
    Serveur->>Navigateur: Renvoi une page HTML
    Note right of Navigateur: Le HTML contient une balise script
    Navigateur->>Serveur: Demande le script
    Serveur->>Navigateur: Renvoi le script
    Navigateur->>Application: Exécute le script
    Note over Application: L'application s'affiche à l'écran
    Note over Application: Affiche un spinner
    Application->>Serveur: Demande la liste des entités
    Note over Serveur: Récupère la liste en base
    Serveur->>Application: Envoi la liste des entités
    Note over Application: Affiche la liste
    Utilisateur->>Application: Ouvre la page d'une entité
    Note over Application: Affiche un spinner
    Application->>Serveur: Demande l'entité
    Note over Serveur: Récupère l'entité en base
    Serveur->>Application: Envoi l'entité
    Note over Application: Affiche l'entité
    Note over Utilisateur: Modifie des champs
    Utilisateur->>Application: Clique sur Sauvegarder
    Note over Application: Affiche un spinner
    Application->>Serveur: Envoi les nouvelles données
    Note over Serveur: Modifie l'entité en base
    Serveur->>Application: Renvoi l'entité mise à jour
    Application->>Serveur: Demande la liste des entités
    Note over Serveur: Récupère la liste en base
    Serveur->>Application: Envoi la liste des entités
    Note over Application: Affiche la liste
</div>

J'entend déjà : "Quentin tu es de mauvaise foi, tu veux prouver quelque chose alors tu exagère"

Sauf que non, promis ! Voyons ce que j'aurais pu exagérer :

- En CSR on redirige après la sauvegarde on ne fait pas un aller retour supplémentaire pour récupérer la liste contrairement à la MPA
  - C'est vrai, sauf que 90% du temps si vous utilisez une API REST avec votre SPA, vous n'allez pas renvoyer la liste après une mise à jour. Il va vous falloir faire deux appels distinct ET séquentiels (puisqu'il faut récupérer la liste après la sauvegarde et pas avant)
  - En SSR, vous n'avez pas un _endpoint_ générique de mise à jour d'une entité, vous avez une action à faire lors d'une sauvegarde donc si vous savez coté serveur que vous allez faire une redirection, vous êtes déjà du bon coté du diagramme pour récupérer la list en base de données.
- Pourquoi écrire à chaque fois "Affiche un spinner" en CSR mais pas en SSR ?
  - Parce qu'afficher un spinner en CSR c'est du code à rajouter et probablement une librairie à télécharger. En SSR, c'est le navigateur qui s'occuper de remplacer votre favicon par un spinner lors d'une requête POST.
  - Il faut bien avouer que c'est moins joli et ça ne fonctionne pas lors du chargement de la page entité. Les spinners en CSR sont bien plus sympa, aussi manuels soient-ils

En SSR : 3 actions de l'utilisateur -> 3 aller-retours avec le serveur.  
En CSR : 3 actions de l'utilisateur -> 6 aller-retours avec le serveur.

### Là où le client blesse

Alors ok, en SSR on a moins de flexibilité sur l'affichage du contenu avec une interface utilisateur parfois appauvrie, mais ça n'empêche qu'on a des performances bien supérieures ! (et qui ne dépendent pas des performances du client puisque les calculs sont faits coté serveur, au pire on dépend de la connexion internet, mais pas plus qu'une SPA finalement)

Ce que je trouve le plus dommage dans le diagramme, les deux points de frictions, c'est le fait de devoir attendre le téléchargement de la page HTML (très court) pour télécharger le script de l'application (plus long) pour ensuite pouvoir télécharger les données à afficher dans l'application (plutôt court).  
Là où une application SSR attend simplement le téléchargement de la page finale.

#### Note sur les metrics

J'en profite pour vous mettre en garde si vous utilisez des systèmes de metrics comme Lighthouse : le FCP n'a pas du tout le même intérêt en SSR !

Le FCP, ou _First Contentful Paint_ c'est le temps que met le premier élément du DOM à apparaître sur la page.

En SSR il est presque toujours beaucoup plus long qu'en CSR. Pourquoi ? Parce que le FCP en CSR représente l'arrivé de la page HTML visuellement vierge, alors qu'en SSR c'est l'arrivé de la page totalement formée. Et forcément ça prend plus de temps.

Il est bon de comparer le FCP au LCP, _Largest Contentful Paint_ qui représente le temps que met la page à afficher sont contenu principal (imaginez une liste). En SSR le FCP == le LCP puisque la page arrive déjà chargée.

Ca ne veux pas dire que le FCP n'est pas à prendre en compte en SSR, mais qu'il ne faut pas forcément les comparer de manières identiques car ils n'indiquent pas la même chose. Un bon FCP n'indique pas un site qui charge plus vite. (enfin si, mais une page blanche)

## Retour au PHP c'est ça ?

Non, pas retour au PHP. Enfin [si vous voulez une Lamborghini](https://twitter.com/taylorotwell/status/1534178479201259520) pourquoi pas.

Javascript a pris une place très importante dans l'écosystème du web, et les outils SSR modernes se reposent dessus. Quitte à faire un serveur Node, autant qu'il serve des pages plutôt que du JSON pourquoi pas ?

### C'est quoi du SSR moderne ?

On n'est pas simplement revenu à un fonctionnement classique comme PHP était, si on reviens vers du SSR c'est parce qu'on a maintenant moyen de l'améliorer pour combiner le meilleur des deux mondes !

Livrer des pages, mais conserver la navigation coté client et avoir des pages dynamiques.  
Partager du code entre le client et le serveur (des [validateurs de formulaires](<[https://](https://quentin.widlocher.com/blog/les-formulaires-avec-remix-pt-1)>) pourquoi pas)  
Limiter les aller-retours avec le serveur et limiter les erreurs de sérialisation.

Le SSR moderne, c'est du SSR avec de **l'hydratation**. 🚰

### L'eau c'est bon

L'hydratation, en plus d'être ce qui nous maintient en vie, c'est ce qui permet de transformer du lait en poudre en lait.

C'est ce même système qui peut transformer une page statique en application dynamique. _Just add ~~water~~ Javascript_ !

Si vous demandez une page à un serveur et qu'il vous la donne directement (SSR) mais qu'en parallèle il télécharge de quoi transformer cette page en application client (comme si elle avait toujours été une SPA) vous profitez du meilleur des deux mondes.

Et si chaque page possède une fonction coté serveur pour savoir quelles données aller chercher et vers quelles page rediriger, vous évitez les allers-retours des API REST.

Mettons à jour notre diagramme en prenant en compte l'hydratation :

<div className="mermaid">
sequenceDiagram
    participant Utilisateur
    participant Application
    participant Navigateur
    participant Serveur
    Utilisateur->>Navigateur: Ouvre le lien de l'application
    Navigateur->>Serveur: Demande la page
    Note over Serveur: Récupère la liste des entités en base
    Serveur->>Navigateur: Renvoi une page HTML avec la liste
    Note right of Navigateur: Le HTML contient une balise script
    Utilisateur->>Navigateur: Ouvre la page d'une entité
    Note over Application: L'utilisateur clique sur le lien avant que le script soit chargé
    Navigateur->>Serveur: Demande la page
    Note over Serveur: Récupère l'entité en base
    Serveur->>Navigateur: Renvoi une page HTML avec les informations
    Note right of Navigateur: Le HTML contient une balise script
    par En parallèle
      Note over Utilisateur: Modifie les champs
      Navigateur->>Serveur: Demande le script
      Serveur->>Navigateur: Renvoi le script
      Navigateur->>Application: Exécute le script
    end
    Note over Application: L'application est désormais fonctionelle coté client
    Utilisateur->>Application: Clique sur Sauvegarder
    Note over Application: Affiche un spinner
    Application->>Serveur: Envoi les nouvelles données
    Note over Serveur: Modifie l'entité en base
    Note over Serveur: Récupère la liste des entités en base
    Serveur->>Application: Renvoi la liste sérialisée
    Note over Application: L'application affiche la liste
</div>

3 actions de l'utilisateur -> 4 aller-retour dont 1 optionnel (le script pour l'hydration)

Si la page est hydratée, elle profite des qualité d'un SPA : elle peut afficher un joli spinner lors de la sauvegarde (sinon tant pis, on fait sans comme avant)

Cette notion d'avoir une hydratation optionnelle qui rend l'application plus sympathique s'appelle du _Progressive Enhancement_, soit de l'amélioration progressive, sous entendu que c'est mieux si possible, mais sinon ça marche quand même.

Grâce à l'hydratation, on est passé discrètement d'une MPA à une SPA sans que l'utilisateur ne souffre d'un temps de chargement et de requêtes HTTP en cascade.

### On peut aller encore plus loin

Si on imagine que seules quelques partie d'une application ont besoin d'être hydratées, on parle alors d'hydratation partielle. Ca permet de limiter la quantité de script à envoyer au client. Si c'est le développeur qui indique à une page statique quels sont ses parties dynamiques à hydrater, on parle d'architecture par îles (**island architecture**) -> [voir l'explication par Astro](https://docs.astro.build/en/concepts/islands/)

Encore plus fou, si on troque une hydratation (même partielle) contre la possibilité d'envoyer une page avec les données serialisées déjà dedans (ça paraît simple mais pas du tout) on parle de résumabilité (_resumability_). -> [voir l'article de Ryan Carniato (créateur de SolidJS)](https://dev.to/this-is-learning/resumability-wtf-2gcm)

## On passe tout en SSR c'est ça

Non, on passe pas tout en SSR. Enfin si vous voulez vous faire recruter par Remix.run pourquoi pas.

Il y a des cas où c'est tout à fait normal de se tourner vers une SPA :

- Si vous avez besoin d'embarquer votre application sur un appareil (application hybride)
- Si vous avez besoin d'un mode hors ligne
- Si vous ne voulez pas déployer un serveur node
- Si vous n'avez pas d'accès à une base de données ou à une API externe
- Si l'application est juste un front connecté à une API existante
- Si on sait qu'il n'y aura pas d'autre front à faire (donc pas besoin d'un API)

Dans ces cas de figures, l'approche SSR n'apportera pas grand chose ou au pire freinera votre développement, les SPA ne sont pas morte bien au contraire.

Pour tout les autres cas de figure, il est temps de se pencher sur cette approche qui permet en une seule codebase de livrer une application efficace et performante, sans rien sacrifier au passage (pour moi les seuls "sacrifices" c'est lors du choix SSR ou pas, si l'application est compatible SSR il n'y a plus de sacrifices à faire)

### Moi je veux bien mais je commence par quoi ?

Il y a a BEAUCOUP de frameworks SSR qui sont arrivé ces derniers mois. Je ne sais pas lequel est arrivé en premier mais je sais que **Remix** faisait parti des premiers et que c'est un framework React donc je le recommande.

Sinon petit topo des frameworks SSR disponibles :

| Framework SSR                                           | Basé sur      | Environnement          | Hydratation partielle | Gère les mutations SSR | Mon avis en un emoji |
| ------------------------------------------------------- | ------------- | ---------------------- | --------------------- | ---------------------- | -------------------- |
| [Remix](https://remix.run/)                             | React         | Node, Deno, CF Workers | Non                   | Oui                    | ❤️                   |
| [Astro](https://astro.build/)                           | Un peu tout\* | Node, Deno, CF Workers | Oui                   | Non                    | 💪                   |
| [Next.js](https://nextjs.org/)                          | React         | Node                   | Non                   | Non                    | 😕                   |
| [Nuxt.js](https://v3.nuxtjs.org/)                       | Vue           | Node                   | Non                   | Non                    | 🫤                    |
| [SvelteKit](https://kit.svelte.dev/)                    | Svelte        | Node                   | Non                   | Oui                    | 😮                   |
| [Fresh](https://fresh.deno.dev/)                        | Preact        | Deno                   | Oui                   | Oui                    | 🙂                   |
| [SolidStart](https://github.com/solidjs/solid-start)    | Solid         | Node                   | Probablement          | Oui                    | ⏳                   |
| [Angular Universal](https://angular.io/guide/universal) | Angular       | Node                   | Non                   | Non                    | 😂                   |

\* React, Preact, Svelte, Vue, Solid, Lit et plus

Quelques précisions :

- _CF Worker_ c'est un environnement spécial qui n'est pas Node et qui est fait pour tourner sur des CDN.
- _Gère les mutations SSR_ ça sous entend : "possède un mécanisme pour envoyer des données au serveur et pas seulement en recevoir" (ce qui est très important pour profiter des avantages du SSR selon moi)
- SolidStart est loin d'être prêt encore, mais il promet beaucoup
- SvelteKit c'est super cool, j'aime pas trop Svelte mais l'approche est super
- J'ai ajouté Angular Universal, mais sans blague, c'est pas fait pour faire du SSR comme je viens de le présenter, si vous utiliser Angular c'est du CSR qu'il vous faut quoi qu'il arrive.
-
