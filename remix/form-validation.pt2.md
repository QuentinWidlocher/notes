---
title: "Les formulaires avec Remix - Partie 2"
subtitle: "Valider un formulaire et gérer les erreurs"
---

## Validation classique

Reprenons notre example précédent et ajoutons quelques champs afin de rentre les choses intéressantes :

```html
<form method="post">
  <label htmlFor="name">Full Name</label>
  <input type="text" name="name" required />

  <label htmlFor="email">Email</label>
  <input type="email" name="email" />

  <label htmlFor="phone">Phone number</label>
  <input type="number" name="phone" />

  <label htmlFor="password">Password</label>
  <input type="password" name="password" />

  <label htmlFor="passwordConfirm">Password confirmation</label>
  <input type="password" name="passwordConfirm" />

  <button type="submit">Submit</button>
</form>
```

Rien de bien méchant, imaginez un simple formulaire d'inscription. (Avec une confirmation de mot de passe à l'ancienne, je déconseille cette approche mais c'est pratique pour mon example)

Quand on utilise l'action que l'on avait créé plus tôt on obtient :

```ts
export let action: ActionFunction = async ({request}) => {
    // On récupère les données du formulaire
    let formData = await request.formData()

    // On transforme ces données en objet:
    let formObj = Object.fromEntries(formData.entries()
    console.debug(formObj)
    /*
     * {
     *      name: "Quentin",
     *      email: "quentin@widlocher",
     *      phone: "06",
     *      password: "hunter2",
     *      passwordConf: "hunter3",
     * }
     */

    return db.user.create(formObj)
}
```

Heu, oups. Ce n'est ni un email valide (qui pourtant a passé la validation coté navigateur), ni un numéro de téléphone valide, ni une confirmation de mot de passe valide...

Vous vous en doutez, il va falloir ajouter une validation de ce formulaire coté serveur.

Un exemple assez simple de validation possible :

```ts
// Ici, FormObject est un type qui ressemble à nos champs
function validateForm(formObj: FormObject) {
  // Valide l'email avec une regex un peu énervée
  if (
    !/^(([^<>()[\]\.,;:\s@\"]+(\.[^<>()[\]\.,;:\s@\"]+)*)|(\".+\"))@(([^<>()[\]\.,;:\s@\"]+\.)+[^<>()[\]\.,;:\s@\"]{2,})$/i.test(
      formObj.email
    )
  ) {
    return false;
  }

  // S'assure que le numéro de téléphone fait 10 chiffre (simpliste)
  if (String(formObj.phone).length != 10) {
    return false;
  }

  // S'assure que les mots de passe sont identiques
  if (formObj.password != formObj.passwordConf) {
    return false;
  }

  return true;
}
```

Bon, on repassera sur l'optimisation mais le code fonctionne comme on l'entend, il permet d'éviter à des données erronées d'êtres insérées en base de données.

Plus qu'a empêcher l'action de se produire en retournant une valeur avant l'ajout en base. Attention Remix a besoin que ses `action` retournent une valeur, même `null`.

```ts
export let action: ActionFunction = async ({request}) => {
    // On récupère les données du formulaire
    let formData = await request.formData()

    // On transforme ces données en objet:
    let formObj = Object.fromEntries(formData.entries()

    // On valide les données
    if (!validateForm(formObj)) {
        return null
    }

    return db.user.create(formObj)
}
```

## Affichage des erreurs classique

Bon, pas mal, mais l'utilisateur n'a aucune idée que son envoi s'est mal passé 😅

Il va juste cliquer sur le bouton "Submit" puis voir son formulaire se vider sans aucune autre information.  
Il faut donc lui afficher la liste des erreurs de son formulaire

Dans notre fonction `validateForm()`, plus question de retourner un simple booléen, on va devoir retourner la liste des erreurs :

```ts
// Ici, FormObject est un type qui ressemble à nos champs
function validateForm(formObj: FormObject) {
  let errors: string[] = [];

  // Valide l'email avec une regex un peu énervée
  if (
    !/^(([^<>()[\]\.,;:\s@\"]+(\.[^<>()[\]\.,;:\s@\"]+)*)|(\".+\"))@(([^<>()[\]\.,;:\s@\"]+\.)+[^<>()[\]\.,;:\s@\"]{2,})$/i.test(
      formObj.email
    )
  ) {
    errors.push("L'email est invalide");
  }

  // S'assure que le numéro de téléphone fait 10 chiffre (simpliste)
  if (String(formObj.phone).length != 10) {
    errors.push("Le numéro de téléphone doit faire 10 chiffres");
  }

  // S'assure que les mots de passe sont identiques
  if (formObj.password != formObj.passwordConf) {
    errors.push("Les mots de passes ne correspondent pas");
  }

  return errors;
}
```

Maintenant, on peux retourner les erreurs

## Validation automatique

## Affichage des erreurs automatique

## Avec un composant d'erreur
