# Gestion de la Connexion des utilisateurs

On va modifier le fichier "Security.yaml" :

```yaml
security:
    encoders:
        App\Entity\User: bcrypt
    
        providers:
            database_users:
                entity: { class: App\Entity\Users, property: username }
    
        firewalls:
            dev:
                pattern: ^/(_(profiler|wdt)|css|images|js)/
                security: false
    
            main:
                # les urls auxquels s'appliquent ce firewall, dans ce cas, ce sont toutes les urls
                pattern: ^/
    
                # La connexion n'est pas requise sur toutes les pages
                # par exemple la page d'accueil
                anonymous: true
    
                form_login:
                    # Le nom de la route de la page de connexion
                    check_path: security_login
                    # Le nom de la route où se trouve le formulaire de connexion
                    # Si un utilisateur tente d'acceder à une page protégée sans en avoir les droits
                    # il sera redirigé sur cette page
                    login_path: security_login
                    # Securisation des formulaires
                    csrf_token_generator: security.csrf.token_manager
                    # La page par defaut apres une connexion reussie
                    default_target_path: admin
    
                logout:
                    # La route où se trouve le process de deconnexion
                    path: security_logout
                    # La route sur laquelle doit etre rediriger l'utilisateur apres une deconnexion
                    target: index
    
        access_control:
            # Les regles de securité
            # Là dans ce cas seul les utilisateurs ayant le rôle ROLE_ADMIN
            # peuvent acceder à toutes les pages commençant par /admin
            - { path: '^/admin', roles: ROLES_ADMIN }
```

On verifier que la gestion des sessions est bien activée (framework.yaml)
```yaml
framework:
    secret: '%env(APP_SECRET)%'
    #default_locale: en
    csrf_protection: { enabled: true }
    #http_method_override: true
 
    # uncomment this entire section to enable sessions
    session:
        # With this config, PHP's native session handling is used
        handler_id: ~
 
    #esi: ~
    #fragments: ~
    php_errors:
        log: true
```

Dans le controller 'SecurityController' créé au debut, on va ajouter les deux méthodes 'Login' et 'LogOut' :

```php
class SecurityController extends Controller
{
    /**
     * @Route("/login", name="security_login")
     */
    public function login(AuthenticationUtils $helper)
    {
        return $this->render('security/login.html.twig', [
            'lastUsername' => $helper->getLastUsername(),
            'Error' => $helper->getLastAuthenticationError(),
        ]);
    }

    /**
     * @Route("/logout", name="security_logout")
     */
    public function logout()
    {
        throw new \Exception('This should never be reached!');
    }
}
```

Ajouter différents dossier et fichier twig pour les rendu...
<br>Exemple : Admin/index...twig et app/index...twig

***IMPORTANT***

Pour gérer la redirection en fonction du groupe de l'utilisateur, il faut créer un Handler

```php
<?php 

namespace App\Controller;

use Symfony\Component\Security\Http\Authentication\AuthenticationSuccessHandlerInterface;
use Symfony\Component\Routing\RouterInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\HttpCache\ResponseCacheStrategy;
use Symfony\Component\HttpFoundation\RedirectResponse;


class AfterLoginRedirection implements AuthenticationSuccessHandlerInterface {

    private $router;

    public function __construct(RouterInterface $router)
    {
        $this->router = $router;
    }

    public function onAuthenticationSuccess(Request $Request, TokenInterface $token){
        $roles = $token->getRoles();
        $rolesTab = array_map(function($role){
            return $role->getRole();
        }, $roles);
        if(in_array('ROLE_ADMIN',$rolesTab)){
            return new RedirectResponse($this->router->generate('admin'));
        }else{
            return new RedirectResponse($this->router->generate('index'));
        }
    }
}
```

Ensuite il faut créer un service lié à ce handler dans le fichier 'services.yaml'

```yaml
redirect.after.login:
        class: App\Controller\AfterLoginRedirection
        tags: ['redirect.after.login']
```
Et pour finir il faut ajouter ce service au firewall du projet : 
```yaml
form_login:
                # Le nom de la route de la page de connexion
                check_path: security_login
                # Le nom de la route où se trouve le formulaire de connexion
                # Si un utilisateur tente d'acceder à une page protégée sans en avoir les droits
                # il sera redirigé sur cette page
                login_path: security_login
                # Securisation des formulaires
                csrf_token_generator: security.csrf.token_manager
                # La page par defaut apres une connexion reussie
                # default_target_path: admin
        ---->   success_handler: redirect.after.login
```