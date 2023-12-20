## Création d'un utilisateur

Pour créer un utilisateur il faut vous rendre une fois votre projet paramétrer dans votre terminal VS Code et faire la commande suivante

```
symfony console make:user
```

Le terminal va compiler et vous poser différentes questions.

1 : 

```
The name of the security user class (e.g. User) [User]:
```

`Le nom de la classe Utilisateur` par défaut il nous propose `User` on fait entrée pour ce choix.

2 : 

```
Do you want to store user data in the database (via Doctrine)? (yes/no) [yes]:
```

`Voulez-vous stocker user dans la base de données ?` par défaut il nous propose Oui donc en tape entrée.

3 : 

```
Enter a property name that will be the unique "display" name for the user (e.g. email, username, uuid) [email]:
```

`On nous demande si on veut un élément unique à vérifier afin d'éviter les doublons dans la bdd` par défaut c'est l'email qui est ciblé, on tape entrée car cela nous convient.

4 : 

```
Does this app need to hash/check user passwords? (yes/no) [yes]:
```

`Voulez-vous hacher et vérifier le mot de passe de l'utilisateur` par défaut il nous met Oui donc on tape entrée

4 Une fois fini, 4 fichiers sont crées

```
 created: src/Entity/User.php
 created: src/Repository/UserRepository.php
 updated: src/Entity/User.php
 updated: config/packages/security.yaml
```

Pour le mot notre utilisateur est seulement pourvu d'un email et d'un mot de passe, nous souhaitons avoir en plus un nom, prénom, adresse, code postal et une ville pour cela retour au terminal et nous entrons la commande suivante : 

```
symfony console make:entity User
```

L'entité User est différente des autres entités par le traitement à sa création mais pour appliquer d'autres propriétés on passe par la commande création des entités.

```
Your entity already exists! So let's add some new fields!
```

Le terminal nous dit que cette entité existe déjà mais que l'on peut y mettre d'autres propriétés et donc on entre la propriété `name`.

```
Field type (enter ? to see all types) [string]:
 > ?
```

On nous demande de quelle type sera notre propriété `name` nous voulons que se soit une chaine de caractères donc on fait entrée car c'est ce qui est proposé par défaut.

```
Field length [255]:
```

`Ensuite de combien de caractères maximum sera notre propriété` par défaut 255 on tape entrée.

```
Can this field be null in the database (nullable) (yes/no) [no]:
```

`On nous demande si la valeur peut être nulle dans la bdd`, on répond non

On fait le même traitement pour les autres propriétés en adaptant les types

firstName = string
address = text
zipCode = integer
town = string

une fois que vous n'avez plus à entrer de nouvelles propriété, faites entrer sans mettre de valeurs à cette question : 

```
Add another property? Enter the property name (or press <return> to stop adding fields):
```

Une fois nos propriétés établie, il faut créer le fichier de migration en tapant la commande suivante : 

```
symfony console make:migration
```

Cela aura pour effet d'ajouter un fichier au dossier `Migrations`

Il faut à présent valider la migration et envoyer les changements à la bdd, pour cela il faut taper une autre commande

```
symfony console doctrine:migration:migrate
```

ou

```
symfony console d:m:m
```

Cela créera la table user dans la bdd.

Maintenant que votre classe User est créée il faut ajouter les fonctionnalités d'authentification.

## Mise en place des systèmes d'authentification

Une fois de plus le terminal est votre allié, tapez la commande suivante :

```
symfony console make:auth
```

Plusieurs questions vont vous être posées

1 : 

```
What style of authentication do you want? [Empty authenticator]:
  [0] Empty authenticator
  [1] Login form authenticator
```

`Quel style d'authentification voulez-vous ?` Pour notre part nous souhaitons un système avec formulaire donc l'option `1`.

2 : 

```
The class name of the authenticator to create (e.g. AppCustomAuthenticator):
```

`Le nom de la classe authenticator` Nous prendrons celui donner en exemple en le recopiant `AppCustomAuthenticator`.

3 : 

```
Choose a name for the controller class (e.g. SecurityController) [SecurityController]:
```

`Le nom du contrôleur`. Nous prendrons le nom par défaut en tapent entrée

4 : 

```
Do you want to generate a '/logout' URL? (yes/no) [yes]:
```

`Voulons-nous générer une route de déconnexion?` Nous choisissons oui.

A la suite de ce questionnaire, vous recevrez un message de succès et 4 fichiers 3 fichiers ont été crée

```
 created: src/Security/AppCustomAuthenticator.php
 created: src/Controller/SecurityController.php
 created: templates/security/login.html.twig
```

Nous avons créer le système de connexion et les sécurités lié aux accès. A présent retour sur le terminal pour créer le système de connexion

```
symfony console make:registration-form
```

Plusieurs questions seront posées

1 : 

```
Do you want to add a #[UniqueEntity] validation attribute to your User class to make sure duplicate accounts aren't created? (yes/no) [yes]
```

`On nous demande si nous voulons un attribut de validation dans notre classe pour être sur d'éviter les doublons dans la bdd?` Nous répondrons oui.

2 : 

```
Do you want to send an email to verify the user's email address after registration? (yes/no) [yes]:
```

`On nous demande si nous souhaitons envoyer un email à l'utilisateur afin de vérifier son adresse mail` Nous répondrons non.

3 : 

```
Do you want to automatically authenticate the user after registration? (yes/no) [yes]:
```

`On nous demande enfin si nous voulons être automatiquement connecté après l'inscription` Nous répondrons oui.

Deux fichiers sont générés à l'issu de ce questionnaire

```
created: src/Form/RegistrationFormType.php
 created: templates/registration/register.html.twig
```

Nous souhaitons inscrire un utilisateur mais un premier problème nous tombe dessus

Dans le formulaire ne figure que le mail et le mot de passe, les autres propriétés que nous avons mis en place précédemment n'apparaisse pas, pour cela il faut aller dans le fichier `RegistrationFormType.php` qui est le dossier qui gère les champs de notre formulaire.

```php
<?php

  

namespace App\Form;

  

use App\Entity\User;

use Symfony\Component\Form\AbstractType;

use Symfony\Component\Form\Extension\Core\Type\CheckboxType;

use Symfony\Component\Form\Extension\Core\Type\PasswordType;

use Symfony\Component\Form\FormBuilderInterface;

use Symfony\Component\OptionsResolver\OptionsResolver;

use Symfony\Component\Validator\Constraints\IsTrue;

use Symfony\Component\Validator\Constraints\Length;

use Symfony\Component\Validator\Constraints\NotBlank;

  

class RegistrationFormType extends AbstractType

{

    public function buildForm(FormBuilderInterface $builder, array $options): void

    {

        $builder

            ->add('email')

            ->add('agreeTerms', CheckboxType::class, [

                                'mapped' => false,

                'constraints' => [

                    new IsTrue([

                        'message' => 'You should agree to our terms.',

                    ]),

                ],

            ])

            ->add('plainPassword', PasswordType::class, [

                                // instead of being set onto the object directly,

                // this is read and encoded in the controller

                'mapped' => false,

                'attr' => ['autocomplete' => 'new-password'],

                'constraints' => [

                    new NotBlank([

                        'message' => 'Please enter a password',

                    ]),

                    new Length([

                        'min' => 6,

                        'minMessage' => 'Your password should be at least {{ limit }} characters',

                        // max length allowed by Symfony for security reasons

                        'max' => 4096,

                    ]),

                ],

            ])

        ;

    }

  

    public function configureOptions(OptionsResolver $resolver): void

    {

        $resolver->setDefaults([

            'data_class' => User::class,

        ]);

    }

}
```

De base dans ce fichier ne figure que email et password, il faut ajouter les autres propriétés

```php
->add('name')

->add('firstName')

->add('address')

->add('zipCode')

->add('town')
```

Ce qui donne : 

```php
<?php

  

namespace App\Form;

  

use App\Entity\User;

use Symfony\Component\Form\AbstractType;

use Symfony\Component\Form\Extension\Core\Type\CheckboxType;

use Symfony\Component\Form\Extension\Core\Type\PasswordType;

use Symfony\Component\Form\FormBuilderInterface;

use Symfony\Component\OptionsResolver\OptionsResolver;

use Symfony\Component\Validator\Constraints\IsTrue;

use Symfony\Component\Validator\Constraints\Length;

use Symfony\Component\Validator\Constraints\NotBlank;

  

class RegistrationFormType extends AbstractType

{

    public function buildForm(FormBuilderInterface $builder, array $options): void

    {

        $builder

            ->add('name')

            ->add('firstName')

            ->add('address')

            ->add('zipCode')

            ->add('town')

            ->add('email')

            ->add('agreeTerms', CheckboxType::class, [

                                'mapped' => false,

                'constraints' => [

                    new IsTrue([

                        'message' => 'You should agree to our terms.',

                    ]),

                ],

            ])

            ->add('plainPassword', PasswordType::class, [

                                // instead of being set onto the object directly,

                // this is read and encoded in the controller

                'mapped' => false,

                'attr' => ['autocomplete' => 'new-password'],

                'constraints' => [

                    new NotBlank([

                        'message' => 'Please enter a password',

                    ]),

                    new Length([

                        'min' => 6,

                        'minMessage' => 'Your password should be at least {{ limit }} characters',

                        // max length allowed by Symfony for security reasons

                        'max' => 4096,

                    ]),

                ],

            ])

        ;

    }

  

    public function configureOptions(OptionsResolver $resolver): void

    {

        $resolver->setDefaults([

            'data_class' => User::class,

        ]);

    }

}
```

Ce n'est pas fini, en effet en l'état actuel le formulaire se trouvant sur la page `localhost:8000/register` contient bien tout les champs mais de façon un peu bancale.

Il faut aussi ajouter des choses au Template se trouvant dans le dossier `Templates` puis dans le dossier `Registration` et qui porte le nom de `registration.html.twig`.

```twig
{% extends 'base.html.twig' %}

  

{% block title %}Register{% endblock %}

  

{% block body %}

    <h1>Register</h1>

  

    {{ form_errors(registrationForm) }}

  

    {{ form_start(registrationForm) }}

        {{ form_row(registrationForm.email) }}

        {{ form_row(registrationForm.plainPassword, {

            label: 'Password'

        }) }}

        {{ form_row(registrationForm.agreeTerms) }}

  

        <button type="submit" class="btn">Register</button>

    {{ form_end(registrationForm) }}

{% endblock %}
```

De base la syntaxe twig `form_row` ne contient que l'email et le mot de passe, il faut ajouter les autres : 

```twig
{{ form_row(registrationForm.name) }}

{{ form_row(registrationForm.firstName) }}

{{ form_row(registrationForm.address) }}

{{ form_row(registrationForm.zipCode) }}

{{ form_row(registrationForm.town) }}
```

Ce qui donne à la fin : 

```twig
{% extends 'base.html.twig' %}

  

{% block title %}Register{% endblock %}

  

{% block body %}

    <h1>Register</h1>

  

    {{ form_errors(registrationForm) }}

  

    {{ form_start(registrationForm) }}

        {{ form_row(registrationForm.name) }}

        {{ form_row(registrationForm.firstName) }}

        {{ form_row(registrationForm.address) }}

        {{ form_row(registrationForm.zipCode) }}

        {{ form_row(registrationForm.town) }}

        {{ form_row(registrationForm.email) }}

        {{ form_row(registrationForm.plainPassword, {

            label: 'Password'

        }) }}

        {{ form_row(registrationForm.agreeTerms) }}

  

        <button type="submit" class="btn">Register</button>

    {{ form_end(registrationForm) }}

{% endblock %}
```

Vous pouvez à présent enregistrer un nouvel utilisateur, Bravo!

Une dernière chose et pas des moindre, il faut gérer l'affichage des liens d'authentification de façon dynamique

Faire apparaitre le bouton Connexion et Inscription lorsque personne n'est connecté et le faire disparaitre lorsque quelqu'un est connecté

De même pour le lien Déconnexion lorsque quelqu'un est connecté .

Pour cela rendez-vous dans le Template Home

Mettez en places liens dans des listes à puce

```html
<ul>

    <li><a href="">Deconnexion</a></li>

    <li><a href="">Inscription</a></li>

    <li><a href="">Connexion</a></li>

</ul>
```

Ajouter les liens vers les pages concernées. On utilise le nom du Template contenu dans le contrôleur

```twig
<ul>

    <li><a href="{{ path('app_logout') }}">Deconnexion</a></li>

    <li><a href="{{ path('app_register') }}">Inscription</a></li>

    <li><a href="{{ path('app_login') }}">Connexion</a></li>

</ul>
```

Mettez en place une condition qui dit que si un utilisateur est connecté il faut afficher un lien et caché les autres sinon on affiche d'autres liens

```twig
<ul>

    {% if app.user %}

    <li><a href="{{ path('app_logout') }}">Deconnexion</a></li>

    {% else %}

    <li><a href="{{ path('app_register') }}">Inscription</a></li>

    <li><a href="{{ path('app_login') }}">Connexion</a></li>

    {% endif %}

</ul>
```

Vous pouvez voir que vous êtes connecté en regardant dans la barre de contrôle mais vous pouvez aussi ajouter dans la condition en dessous du bouton Connexion un message de bienvenue à l'utilisateur : 

```twig
    <li>Bonjour {{ app.user.firstName }}</li>
```

Ce qui donne en tout ce type de fichier : 

```twig
<ul>

    {% if app.user %}

    <li><a href="{{ path('app_logout') }}">Deconnexion</a></li>

    <li>Bonjour {{ app.user.firstName }}</li>

    {% else %}

    <li><a href="{{ path('app_register') }}">Inscription</a></li>

    <li><a href="{{ path('app_login') }}">Connexion</a></li>

    {% endif %}

</ul>
```

Voila pour la mise en place de l'utilisateur et des outils d'authentification.