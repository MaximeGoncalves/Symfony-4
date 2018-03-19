Gestion Utilisateurs
======================

Sommaire :
====
1. [Création BDD](#Creation)
2. [Connexion](#Connexion)

# Creation de la base de données
---
Pour créer la base de données, modifier le fichier **.env**

    DATABASE_URL=mysql://root:@127.0.0.1:3306/softeasesupport

Puis en ligne de commande on créer la base de données : 

    $ php bin/console doctrine:database:create

On créer ensuite notre controller

    $ php bin/console make:controller 

On créer ensuite l'entitée Users 

    $ php bin/console make:entity

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\UserInterface;

/**
 * @ORM\Entity(repositoryClass="App\Repository\UsersRepository")
 */
class Users implements UserInterface,
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", unique=true)
     *
     * @var string
     */
    private $username;

    /**
     * @ORM\Column(type="string")
     *
     * @var string
     */
    private $password;

    /**
     * @ORM\Column(type="string",unique=true)
     *
     * @var string
     */
    private $email;

    /**
     * @ORM\Column(type="json")
     *
     * @var table,json
     */
    private $roles = [];


    /**
     * Get the value of username
     *
     * @return  string
     */ 
    public function getUsername()
    {
        return $this->username;
    }

    /**
     * Set the value of username
     *
     * @param  string  $username
     *
     * @return  self
     */ 
    public function setUsername(string $username)
    {
        $this->username = $username;

        return $this;
    }

    /**
     * Get the value of password
     *
     * @return  string
     */ 
    public function getPassword()
    {
        return $this->password;
    }

    /**
     * Set the value of password
     *
     * @param  string  $password
     *
     * @return  self
     */ 
    public function setPassword(string $password)
    {
        $this->password = $password;

        return $this;
    }

    /**
     * Get the value of email
     *
     * @return  string
     */ 
    public function getEmail()
    {
        return $this->email;
    }

    /**
     * Set the value of email
     *
     * @param  string  $email
     *
     * @return  self
     */ 
    public function setEmail(string $email)
    {
        $this->email = $email;

        return $this;
    }

    /**
     * Get the value of roles
     *
     * @return  table,json
     */ 
    public function getRoles()
    {
        return $this->roles;
    }

    /**
     * Set the value of roles
     *
     * @param  table,json  $roles
     *
     * @return  self
     */ 
    public function setRoles($roles)
    {
        $this->roles = $roles;

        return $this;
    }

    /**
     * Get the value of id
     */ 
    public function getId()
    {
        return $this->id;
    }

    /**
     * Set the value of id
     *
     * @return  self
     */ 
    public function setId($id)
    {
        $this->id = $id;

        return $this;
    }
}
```
On met à jour la base de données :

    php bin/console doctrine:migration:diff
    php bin/console doctrine:migration:migrate

Pour créer les utilisateurs via les fixtures il faut d'abord mettre à jour l'encoder de password dans le fichier "config\packages\security.yaml"

```YAML
    security:
        encoders:
            App\Entity\Users: bcrypt
```

Ensuite on va générer un fichier de type fixture pour cérer nos utilisateurs.

    $ php bin/consol make:fixture

Fichier Fixtures : 

```php
<?php

namespace App\DataFixtures;

use App\Entity\Users;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\Persistence\ObjectManager;
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;

class UsersFixtures extends Fixture
{
    private $passwordEncoder;

    public function __construct(UserPasswordEncoderInterface $passwordEncoder)
    {
        $this->passwordEncoder = $passwordEncoder;
    }

    public function load(ObjectManager $manager)
    {
        foreach ($this->getUserData() as [$username, $password, $email, $roles]) {
            $user = new Users();
            $user->setUsername($username);
            $user->setPassword($this->passwordEncoder->encodePassword($user, $password));
            $user->setEmail($email);
            $user->setRoles($roles);
 
            $manager->persist($user);
            $this->addReference($username, $user);
        }

        $manager->flush();
    }

    private function getUserData()
    {
        return [
            //$userData = [$username, $password, $email, $roles]
            ['admin', 'admin', 'admin@symfony.com', ['ROLES_ADMIN']],
            ['jane', 'kitten', 'jane@symfony.com', ['ROLES_USER']],
            ['tom', 'kitten', 'tom@symfony.com', ['ROLES_USER']],
        ];
    }
```

# Connexion
**Utiliser ces utilisateurs**
---