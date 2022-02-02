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
  // S'assure qu'un nom a été donnée
  if (!formObj.name) {
    return;
  }

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
    let formData = await request.formData()

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

  // S'assure qu'un nom a été donnée
  if (!formObj.name) {
    errors.push("Un nom est requis");
  }

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

Maintenant, on peux retourner les erreurs à notre utilisateur via la réponse de l'action

```ts
export let action: ActionFunction = async ({request}) => {
    let formData = await request.formData()

    let formObj = Object.fromEntries(formData.entries()

    // On valide les données
    let errors = validateForm(formObj)
    if (errors.length > 0) {
        // On retourne un objet avec les erreurs
        return {
          success: false,
          errors,
        }
    }

    // On retourne un objet avec le résultat
    return {
      success: true,
      data: await db.user.create(formObj)
    }
}
```

Maintenant, avec un peu de _magie Typescript_ on peux récupérer nos erreurs proprement coté client

```tsx
// On déclare un type avec une union pour gérer le succès et l'échec
// Comme ça quand on regarde juste `obj.success`, Typescript sait s'il
// doit chercher une propriété `data` ou `errors`
type ActionPayload =
  | {
      success: true;
      data: User; // on imagine un type lié à celui retourné par la bdd
    }
  | {
      success: false;
      errors: string[];
    };

export default function SimpleFormPage() {
  // On récupère ce que l'action nous renvoi
  let actionData = useActionData<ActionPayload>();

  // On se sert du type pour extraire une liste d'erreur
  let errors = actionData.success ? [] : actionData.errors;

  // On affiche la liste des erreurs au dessus de notre formulaire
  return (
    <section>
      {errors.length > 0 ? (
        <ul>
          {errors.map((error) => (
            <li>{error}</li>
          ))}
        </ul>
      ) : null}
      <form method="post">{/* [...] */}</form>
    </section>
  );
}
```

Plutôt pas mal ! On pourrais aussi renvoyer un dictionnaire champ/erreurs sous la forme d'un objet pour avoir le détail des champs de leurs erreurs spécifique.

Mais déjà pour l'instant, cette solution est tout à fait envisageable et fonctionne dans de nombreux cas.

## Validation automatique

Allons encore plus loin avec un validateur automatique, l'idée est de créer une fonction de validation qui pourrais nous permettre de :

- Déterminer automatiquement le type de l'objet d'erreur
- Créer et gérer plus facilement les erreurs, voire des les grouper par champs
- Générer automatique un objet formaté avec les erreurs, dans la même forme que le formulaire

Évidement pour pouvoir faire autant sans y passer des heures, on va pouvoir se servir d'un outil très performant : [zod](https://github.com/colinhacks/zod)

Zod est un ~~validateur~~ parser, qui permet de créer des fonctions de validations, lui donner un objet et de récupérer un _autre objet_ qui lui est correct.  
C'est cette nuance qui fait de zod un parser plutôt qu'un validateur. (Pour plus d'information technique sur cette nuance, vous pouvez [lire l'inspiration de zod ici]([https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/]))

Le fonctionnement est très simple, on construit un validateur avec les fonctions de zod pour faire un schéma ressemblant à notre objet final, exemple complet avec notre formulaire :

```ts
let validator = z.object({
  name: z.string().required("Un nom est requis"),
  email: z.string().email("L'email est invalide"),
  phone: "06",
  password: "hunter2",
  passwordConf: "hunter3",
});
```

## Affichage des erreurs automatique

## Avec un composant d'erreur
